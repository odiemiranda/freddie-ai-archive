---
id: brief-autonomous-self-learning
title: ASL Phase 2 — Design Brief
project-code: ASL
status: architecture-passed
created: 2026-04-07
updated: 2026-04-07
author: freddie + user (design conversation), mccall (architecture pass)
tags: [brief, daemon, pipeline, architecture]
---

# ASL Phase 2 — Design Brief

## Problem Statement

The self-learning pipeline shipped in AAR Phase 1 works as a proof of concept but hits structural ceilings that prevent it from being trusted in production or exposed to external MCP clients.

**Observed pain points (from the 2026-04-07 session):**

1. **Synchronous execution blocks user sessions.** The autonomous distill pipeline takes 7-8 minutes per full run and currently runs during `/bye` or on-demand. Users wait for learning to finish before they can do anything else. This is the classic failure mode of synchronous self-learning systems — Spotify, Netflix, and every production ML recommender went async specifically to avoid it.

2. **MiniMax judge produces false rejections via parser failures.** `judge-panel.ts:110-115` treats unparseable responses as REJECT ("conservative default"). MiniMax M2.5 responses are silently failing JSON parse — observed in the original 77 staged files and again in echo proposals on 2026-04-07. Every parse failure blocks unanimous approval even when Gemini + Grok both approved substantively. The signal-to-noise ratio of the entire judge panel is compromised.

3. **The AAR-0009 distill cache cannot hit in forward-progress runs.** Cache key includes the soul content hash. Every successful auto-apply mutates the soul, invalidating the cache. Cache only hits in narrow cases (mid-pipeline retry, all-staged runs, manual rerun without intervening changes). The 8-minute runtime is a property of the synchronous design, not something a cache can fix.

4. **Knowledge files can carry stale info that propagates into proposals.** During triage, we found a staged proposal claiming "MiniMax strict 256-token limit" — but that was bumped to 1024 in commit `3357141`. The knowledge file wasn't updated, so distillation tried to promote the stale rule. Constitution gate caught it, but only because it contradicted other rules. There's no mechanism to detect "knowledge file contains stale facts."

5. **No agent lifecycle visibility.** You cannot answer "when was tala last distilled?" or "which agents have pending inbox items that need consolidation?" without reading files and eyeballing timestamps. The morning briefing shows aggregate counts but not per-agent state.

6. **External MCP cannot ship safely.** The MCP server exposes pipeline tools. Every flaky thing in the pipeline becomes a flaky external API. Before external clients can safely consume the pipeline, the foundation needs to stop producing silent failures.

## Goals

1. **Decouple fast path from slow path.** Save / reflect / consolidate remain inline and fast. Distill + gates + judges + apply move to a background daemon triggered by memory folder changes.
2. **Make judge panel trustworthy.** Fix MiniMax parser OR replace/refactor the panel so infrastructure failures don't masquerade as substantive rejections.
3. **Introduce agent lifecycle state as a first-class concept** via brain.db, queryable from CLI and UI.
4. **Establish external MCP readiness criteria** — a concrete checklist that must be green before exposing pipeline tools to external clients.
5. **Do NOT build the web UI yet.** But design the daemon to be UI-ready (HTTP/SSE transport exists, state is queryable, task queue is observable).

## Non-Goals

- Rewriting Phase 1 pipeline logic (distill, gates, judges, apply paths all stay as-is).
- Building the web UI (separate project, deferred).
- Building the desktop app via Electrobun (separate project, deferred).
- Replacing Gemini/Grok as primary judges (only MiniMax is under review).
- Local LLM integration (deferred — see §6; gated on GPU hardware commitment the user has explicitly declined for this phase).
- Eliminating the cloud LLM dependency.

---

## Proposed Architecture

### 1. Daemon service

**Location:** `src/services/self-learning-daemon/` (new folder — distinct from `src/servers/freddie-ai/` which is MCP-only)

**Responsibilities:**
- Watch `vault/studio/memory/{agent}/inbox/` and `vault/studio/memory/shared/inbox/` for file changes
- Coalesce rapid changes into per-agent task batches (see §1 File watcher below)
- Hash-check changed files against `sync_meta` — skip if content unchanged
- Enqueue tasks in the brain.db `tasks` table
- Run a worker pool that claims tasks with heartbeat-based lease semantics
- Execute tasks (consolidate, distill, gate, judge, apply) using existing Phase 1 libs
- Update `agent_lifecycle` table on task completion
- Expose an HTTP/SSE API for observability (future UI integration)
- Survive restarts without losing in-flight work

**Process model:**
- Single daemon process (long-running supervisor).
- **Worker pool as child processes spawned via `Bun.spawn()` with IPC for work dispatch and result reporting.** Each worker opens its own SQLite connection in WAL mode (readers + single writer is safe). The 200-500ms per-task startup cost is absorbed per-task but is negligible compared to the 1-8 minute pipeline runtime. **A crashed worker does NOT take down the parent daemon** — this is the decisive advantage over `worker_threads`.
  - **Why not `worker_threads`:** Bun `worker_threads` on Windows is an outright disqualifier per the research round. Issues #14332 (Prisma workers segfault on Windows), #15964 (official stability tracker), and #24405 (VM data race during `notifyNeedTermination` crashes host process) are all open as of early 2026. Creator guidance advises against calling `.terminate()` and against creating temporary workers. A segfault during worker termination crashes the host daemon — unacceptable for unattended operation.
  - **Empirical unknown for ASL-0006:** Measure actual `Bun.spawn()` startup cost loading the ASL codebase on the user's machine. If >2s per task, pivot to long-lived worker processes that poll the task queue (one process per pool slot, reused across tasks) instead of one child per task. This is a minor refactor gated on measurement.
  - **Per-agent mutex:** enforced via the task queue itself (claim query can include `AND NOT EXISTS (SELECT 1 FROM tasks WHERE agent = ? AND status = 'claimed')` to prevent two simultaneous claims for the same agent — detailed in ASL-0003).
  - **Global semaphore around LLM calls** for rate limiting (respects Gemini 10 RPM free tier) — lives in the parent daemon, enforced via IPC token grants to children.
- **PID file + graceful shutdown on SIGTERM:** parent drains in-flight workers, releases claims, then exits.

**File watcher:**
- **Library: `chokidar@^3.6.0` — pinned explicitly.** Do NOT use v4.0.0 — Issue #1361 documents broken atomic-write detection on Windows 11 (new files fire as `unlink` events without delays, causing missed events). Only v3.6.0 is known-reliable on Windows 11 as of early 2026.
- **Configuration (mandatory):**
  ```js
  chokidar.watch('vault/studio/memory/*/inbox', {
    usePolling: true,              // native ReadDirectoryChangesW drops events under Obsidian Sync write patterns
    interval: 500,                 // balance CPU vs latency
    awaitWriteFinish: {
      stabilityThreshold: 2000,    // wait 2s after last size change
      pollInterval: 100
    },
    atomic: true,                  // filter atomic save artifacts
    ignoreInitial: true,           // daemon reconciles state on startup separately
    ignored: /(\.swp|\.tmp|^\.|~$)/
  })
  ```
- **`usePolling: true` is mandatory**, not optional. Native `ReadDirectoryChangesW` is known-unreliable and drops events under bulk writes — Obsidian Sync pulling files from another machine produces exactly the burst pattern that native watchers miss. Polling has a CPU cost but the ASL memory directory is small (<100 files total across all agents) so the overhead is negligible.
- **Coalescing (NOT debounce):** Accumulate all events in a 1000ms window into a per-agent map. When the window closes, emit one `consolidate` task per affected agent, with the set of changed file paths as payload. Debounce would lose information about which files were touched; coalesce preserves the set while collapsing bursts.
  ```ts
  // Coalesce file events into per-agent tasks — classic trailing-edge debounce on the emit, but preserving all paths
  const pending = new Map<string, Set<string>>()  // agent -> set of changed file paths
  let flushTimer: Timer | null = null
  const WINDOW_MS = 1000

  watcher.on('all', (event, path) => {
    if (event !== 'add' && event !== 'change') return
    const agent = extractAgentFromPath(path)
    if (!agent) return
    if (!pending.has(agent)) pending.set(agent, new Set())
    pending.get(agent)!.add(path)
    if (flushTimer) clearTimeout(flushTimer)
    flushTimer = setTimeout(flush, WINDOW_MS)
  })

  function flush() {
    for (const [agent, paths] of pending) {
      enqueueTask({
        agent,
        task_type: 'consolidate',
        payload: JSON.stringify({ changed_files: [...paths], trigger: 'file-change' }),
        trigger_reason: 'file-change',
      })
    }
    pending.clear()
    flushTimer = null
  }
  ```
- **Why 1000ms, not 2-5s:** A typical reflect pass writes 5-10 memory files sequentially with 1-3 second gaps between writes. A 2-5 second window would collapse the pass into one task (desired), but also adds 2-5 seconds of latency to every isolated save. 1000ms captures the typical burst pattern without the tail-latency cost. If empirical measurement during ASL-0005 shows bursts exceeding 1s gaps, tune upward.
- **Interaction with Obsidian Sync:** vault/ is synced via Obsidian Sync (not iCloud, not git). The watcher sees sync-driven writes the same as local edits — that's desired (memory files created on another machine trigger distillation on this one). Polling mode is the critical enabler here: native watchers miss sync-client burst writes entirely.
- **Interaction with iCloud backup:** `backup.ts` is currently manual-trigger (user runs it explicitly). It copies brain.db and vault/ to iCloud. The daemon does NOT trigger backups. No conflict.

### 1a. Daemon lifecycle management

**Process supervisor: PM2 + `pm2-windows-startup`.**

| Option | Verdict | Why |
|--------|---------|-----|
| NSSM / WinSW / Servy / node-windows | Rejected | Requires admin, overkill for single-user dev scope |
| Windows Task Scheduler "At log on" | **Rejected** | Silently assigns Below-Normal process priority + Low I/O priority + memory priority 4 — daemon runs **2-3x slower than interactive**. Fix requires XML-editing `<Priority>7</Priority>` → `<Priority>4</Priority>` which cannot be done through the Task Scheduler GUI. Too error-prone for a permanent daemon. |
| Startup folder | Rejected | No auto-restart on crash, leaves CMD window visible |
| **PM2 + `pm2-windows-startup`** | **Chosen** | Registry-based autostart via `HKCU\...\Run`, tied to user login, halts cleanly on logoff (matches single-user dev scope). Native crash-restart with exponential backoff. `pm2 install pm2-logrotate` handles rotation. |

**PM2 responsibilities:**
- Auto-start on user login (writes registry Run key via `pm2-windows-startup`)
- Crash-restart with exponential backoff (default max 5 attempts, then manual intervention)
- Halt on user logoff — desired, the user is the only consumer
- Log capture to `~/.pm2/logs/` with `pm2-logrotate` managing rotation

**Daemon logging:**
- stdout/stderr captured by PM2 → rotated by `pm2-logrotate`
- Daemon itself additionally writes **structured JSON lines to `daemon.log`** for the observability HTTP API to tail. Two log sinks is deliberate: PM2 logs are human-readable failure diagnostics; `daemon.log` is machine-parseable operational state.

**Empirical unknown — ASL-0007 MUST verify in a 1-hour smoke test:**
- PM2 + Bun compatibility on Windows 11 is **not source-confirmed** by the research round. PM2 is historically Node.js-focused. Likely works via `pm2 start bun -- <script>` but needs testing.
- Smoke test checklist:
  1. `pm2 start bun -- src/services/self-learning-daemon/index.ts`
  2. Verify daemon is running via `pm2 list`
  3. Reboot the machine
  4. Verify daemon auto-restarted after login
  5. Verify graceful shutdown on `pm2 stop` and on user logoff
  6. Trigger a deliberate crash (kill worker, throw uncaught in daemon) — verify auto-restart kicks in
- If PM2 + Bun fails on Windows 11: fall back to **Task Scheduler with XML-patched `<Priority>4</Priority>`** as the only working alternative. Do not attempt unpatched Task Scheduler. Document whichever path is chosen in ASL-0007's completion notes.

### 2. Task queue in brain.db

New table with **heartbeat-based lease semantics** (not fixed-duration lease — see rationale below):

```sql
CREATE TABLE tasks (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  agent TEXT NOT NULL,
  task_type TEXT NOT NULL,                  -- 'consolidate' | 'distill' | 'full-pipeline'
  status TEXT NOT NULL,                     -- 'pending' | 'claimed' | 'completed' | 'failed'
  payload TEXT,                             -- JSON, task-specific args
  claimed_by TEXT,                          -- worker PID / identifier
  claimed_at TEXT,                          -- ISO timestamp of initial claim
  heartbeat_at TEXT,                        -- last heartbeat from worker; stale → reclaimable
  lease_duration_ms INTEGER NOT NULL DEFAULT 60000,  -- 60s default; configurable per task type
  attempts INTEGER NOT NULL DEFAULT 0,
  max_attempts INTEGER NOT NULL DEFAULT 3,
  last_error TEXT,
  trigger_reason TEXT,                      -- 'file-change' | 'manual' | 'schedule'
  created_at TEXT NOT NULL,
  completed_at TEXT,
  duration_ms INTEGER
);

CREATE INDEX idx_tasks_status_created   ON tasks(status, created_at);
CREATE INDEX idx_tasks_agent_status     ON tasks(agent, status);
CREATE INDEX idx_tasks_status_heartbeat ON tasks(status, heartbeat_at);
```

**Claim semantics (use `BEGIN IMMEDIATE` to avoid `SQLITE_BUSY` on the UPDATE-RETURNING):**

```sql
BEGIN IMMEDIATE;

UPDATE tasks
SET status = 'claimed',
    claimed_by = ?,
    claimed_at = datetime('now'),
    heartbeat_at = datetime('now'),
    attempts = attempts + 1
WHERE id = (
  SELECT id FROM tasks
  WHERE status = 'pending'
     OR (
       -- reclaimable: claimed but heartbeat is stale (worker crashed or hung)
       status = 'claimed'
       AND (
         heartbeat_at IS NULL
         OR (julianday('now') - julianday(heartbeat_at)) * 86400000 > lease_duration_ms
       )
     )
  ORDER BY created_at
  LIMIT 1
)
RETURNING id, agent, task_type, payload, lease_duration_ms;

COMMIT;
```

**Why heartbeat-based leasing (reversed from the original fixed-5-minute draft):**

| Lease model | Crash recovery | Legitimate long tasks | Verdict |
|---|---|---|---|
| Fixed 5-min lease | 5 min of dead time on early crashes | Fails if distill takes >5 min | Anti-pattern — cited by BullMQ, Temporal, Celery, SQS as the thing their designs avoid |
| **Heartbeat (60s lease + 30s heartbeat)** | Max 60s dead time after crash | Healthy worker auto-extends indefinitely | AWS SQS, BullMQ, Temporal, Celery all converge on this |
| No lease (ack on completion) | Impossible to detect dead workers | Works fine | Can't recover from stuck/hung workers |

**Worker loop:**
1. **Claim task** via `BEGIN IMMEDIATE` + `UPDATE ... RETURNING`. Reads `lease_duration_ms` from the returned row.
2. **Start heartbeat interval** — every `lease_duration_ms / 2` (e.g., 30s for default 60s lease). Heartbeat is a simple `UPDATE tasks SET heartbeat_at = datetime('now') WHERE id = ? AND claimed_by = ?`.
3. **Run task** (consolidate / distill / etc.) using existing Phase 1 libs.
4. **Stop heartbeat interval.**
5. **Mark completed or failed:** `UPDATE tasks SET status = ?, completed_at = datetime('now'), duration_ms = ?, last_error = ? WHERE id = ?`.
6. **If worker crashes** at any point: heartbeat stops firing → another worker's claim query will find this task with a stale heartbeat and reclaim it within `lease_duration_ms` max.

**Per-task-type lease tuning (future):**
- `consolidate` — default 60s (typically completes in <10s)
- `distill` — 300000 (5 min) default — distillation historically runs 1-8 min, heartbeat keeps it alive
- `full-pipeline` — 600000 (10 min) — covers the longest observed pipeline runtime with margin
- All three override `lease_duration_ms` on insert. The default of 60s is the conservative fallback.

**Empirical unknowns for ASL-0003:**
- `BEGIN IMMEDIATE` contention behavior under Bun's bun:sqlite binding when 5+ child workers all attempt to claim simultaneously. Should be safe (WAL + IMMEDIATE is the standard), but verify.
- `(status, heartbeat_at)` index performance with small table (<10k rows). Should be a non-issue but measure.

### 3. Agent lifecycle table

New table — brain.db is the source of truth for per-agent state; a human-readable `.state.md` view can optionally be regenerated on demand.

```sql
CREATE TABLE agent_lifecycle (
  agent TEXT PRIMARY KEY,
  last_saved_at TEXT,                  -- last memory file written
  last_reflected_at TEXT,              -- last /reflect run
  last_consolidated_at TEXT,           -- last consolidate-memory run
  last_distilled_at TEXT,              -- last distill-soul run
  last_distill_input_hash TEXT,        -- hash of knowledge dir at last distill (for staleness detection)
  soul_version_hash TEXT,              -- current soul content hash
  inbox_count INTEGER NOT NULL DEFAULT 0,
  knowledge_count INTEGER NOT NULL DEFAULT 0,
  pending_task_count INTEGER NOT NULL DEFAULT 0,
  distill_trigger_reason TEXT,
  updated_at TEXT NOT NULL
);
```

**Distill decision logic (daemon):**
- On file watcher event → check `agent_lifecycle.last_distill_input_hash` vs current hash of agent's knowledge dir.
- If different → enqueue distill task.
- If same → no-op (nothing to learn).
- On manual `asl distill-all` → enqueue one task per agent regardless of hash.

**CLI tool:** `bun run tool agent-state [--agent NAME]` prints the table as a human-readable view.

### 4. Hook changes

**Current:**
```
Save memory → reflect → (session end) → consolidate → distill → gates → judges → apply
```

**Proposed:**
```
Inline (fast, <2s):
  Save memory → reflect → (nothing more)

File watcher (triggered async):
  File change → coalesce 1000ms → enqueue consolidate task per agent

Daemon worker (background):
  Claim consolidate task → run → enqueue distill task if knowledge changed

Daemon worker (background):
  Claim distill task → distill → gates → judges → apply → update lifecycle
```

Hook scripts become tiny. Session end does NOT trigger distill. The daemon handles it.

### 5. Judge panel fixes

**Shipped in ASL-0001 (commit `8e7a92d`).** MiniMax parser defensive fix, judge health check pre-flight, quorum-among-alive math, errorType classification, and degradedMode handling are all in place. Questions Q6 (parser root cause) and Q7 (quorum under degraded mode) are resolved — see ASL-0001 task doc for the full design.

**Outstanding for ASL Phase 2:** none from judge panel. The post-ship signal is clean enough that infrastructure failures no longer masquerade as substantive rejections. If the raw body logging fires during a real run, we'll have evidence to diagnose the MiniMax parse failure root cause in a follow-up task.

### 6. LLM strategy refinement

**Gemini — primary, no changes this phase:**
- Distill, constitution gate, gate-regression contradiction checks — all stay on Gemini.
- Distill prompt leanness is a future optimization (not this phase).
- Free tier 10 RPM is the hard rate limit — daemon global semaphore respects it.

**MiniMax — shipped defensive fix (ASL-0001):**
- Parser defensive reads + raw body logging + retry-once are in place.
- Judge health check pre-flight + quorum-among-alive means parse failures no longer block pipeline progress.
- If the root cause ever gets diagnosed, we can remove the defensive path. Until then, the defensive path IS the fix.

**Local LLMs — DEFERRED to a future phase.** The research round confirmed llama.cpp + GBNF grammar could deliver 0.75-0.85 accuracy on binary contradiction checks (possibly hitting the 90% gate with Chain-of-Thought + curated few-shot + temperature 0), but this is **gated on GPU hardware**. CPU-only inference at 3-6 tok/s means 6+ minutes of local LLM runtime per pipeline run, which defeats the entire decoupling effort. User has explicitly declined the GPU hardware commitment for this phase. Gemini remains the sole LLM for gate-regression contradiction checks. If a GPU becomes available or a future 4B model changes the CPU latency math, revisit as a new ASL task. Research findings preserved at `research/2026-04-07-local-llm-findings.md`.

### 7. External MCP readiness checklist

MCP server stays in `src/servers/freddie-ai/` (MCP-only, per user convention). External exposure is gated on:

- [x] MiniMax parser bug fixed OR judge panel refactored — **shipped in ASL-0001**
- [x] Judge health check pre-flight in place — **shipped in ASL-0001**
- [ ] Daemon running and processing tasks reliably
- [ ] `agent_lifecycle` table populated and queryable
- [ ] Review-staged CLI tool exists (ASL-0012)
- [ ] At least 3 full pipeline runs without silent failures in daemon logs
- [ ] Task queue idempotency verified via crash-recovery test
- [ ] Rate limiting verified under concurrent task load
- [ ] `external_mcp_ready` flag in settings flipped to true

Until all boxes are checked, external MCP is a no-go. Hard block.

### 8. UI — deferred to separate project

**Two parallel UI ideas captured:**

1. **Web UI** — straightforward, HTTP-based, polls/SSE against the daemon's HTTP API
2. **Desktop app via Electrobun** — native feel, same backend API

Both share the daemon as the backend. The daemon's HTTP transport already exists in `src/servers/freddie-ai/` (http mode) and can be extended with new routes:
- `GET /agents` — list with lifecycle state
- `GET /agents/:name` — detail view
- `GET /tasks?status=pending` — task queue snapshot
- `GET /events` — SSE stream
- `POST /staged/:id/approve|reject` — triage actions

**This brief does NOT scope the UI.** It only ensures the daemon's API is UI-ready. Separate project, separate brief, after ASL Phase 2 ships.

---

## Open Design Questions

All 8 questions from the original draft are resolved. Research and resolution trail below:

| # | Question | Resolution | Evidence |
|---|----------|------------|----------|
| Q1 | Windows file watching reliability (chokidar config, Obsidian Sync interaction) | `chokidar@^3.6.0` pinned + `usePolling: true` mandatory + `awaitWriteFinish` + `atomic: true` | `research/2026-04-07-daemon-architecture-findings.md` §Q1 |
| Q2 | Worker thread vs child process | **Child processes via `Bun.spawn()` with IPC.** Reverses original BRIEF preference — Bun `worker_threads` has open Windows segfault issues (#14332, #15964, #24405) that crash the host daemon. | `research/2026-04-07-daemon-architecture-findings.md` §Q2 |
| Q3 | Task queue crash recovery semantics | **Heartbeat-based lease** (60s default + 30s heartbeat) instead of fixed 5-min lease. Matches AWS SQS / BullMQ / Temporal / Celery pattern. | `research/2026-04-07-daemon-architecture-findings.md` §Q3 |
| Q4 | Daemon lifecycle management on Windows | **PM2 + `pm2-windows-startup`**. Do NOT use Task Scheduler (silent Below-Normal priority gotcha). PM2 + Bun compatibility needs 1-hour smoke test in ASL-0007. | `research/2026-04-07-daemon-architecture-findings.md` §Q4 |
| Q5 | Local LLM quality at 7B scale | **DEFERRED** — gated on GPU hardware commitment that the user has explicitly declined for this phase. Research preserved for future revival. | `research/2026-04-07-local-llm-findings.md` |
| Q6 | MiniMax parser root cause | Resolved by defensive fix — commit `8e7a92d` (ASL-0001). Root cause diagnosis deferred until next fire of the raw-body log. | `tasks/2026-04-07-181755-ASL-0001-minimax-parser-fix.md` |
| Q7 | Judge quorum under degraded mode | Resolved by ASL-0001 quorum-among-alive with 1-alive autonomy floor (never auto-applies, always stages). | `tasks/2026-04-07-181755-ASL-0001-minimax-parser-fix.md` §ADR |
| Q8 | Task trigger granularity (file change → task) | **1000ms coalescing window** (NOT 2-5s debounce). Preserves the set of touched files per agent, collapses reflect-pass bursts into one task per affected agent. | `research/2026-04-07-daemon-architecture-findings.md` §Q8 |

## Implementation Slices

Remaining slices numbered ASL-0002 through ASL-0012. ASL-0010 skipped; no renumbering — **numbering is permanent** to preserve stable references in decisions/commits/tasks registry.

- **ASL-0001 — MiniMax parser fix + judge health check** — **SHIPPED** (commit `8e7a92d`)
- **ASL-0002 — brain.db schema: `tasks` + `agent_lifecycle` tables** (migration, no behavior change)
- **ASL-0003 — Task queue library** (`src/libs/task-queue.ts` with heartbeat-based claim/lease semantics)
- **ASL-0004 — Agent lifecycle library** (`src/libs/agent-lifecycle.ts` with upsert/query)
- **ASL-0005 — File watcher service** (chokidar@3.6.0 + polling + coalesce + enqueue)
- **ASL-0006 — Worker pool** (`src/services/self-learning-daemon/worker.ts` — child processes via `Bun.spawn()`)
- **ASL-0007 — Daemon entry point** (`src/services/self-learning-daemon/index.ts` + PM2 lifecycle scripts + 1-hour smoke test)
- **ASL-0008 — Hook decoupling** (remove distill from session-end hooks)
- **ASL-0009 — `agent-state` CLI tool**
- **ASL-0010 — Local LLM integration for gate-regression** — **DEFERRED (no GPU commitment)**
- **ASL-0011 — External MCP readiness gate** (settings flag + startup check)
- **ASL-0012 — Review-staged CLI tool** (deferred from AAR-0008 triage; on the readiness checklist)

Slice order is roughly dependency-ordered. ASL-0001 shipped independently. ASL-0002 is the foundation for everything after — nothing else can be built without the schema in place.

## Source Material

This brief synthesizes:
- The 2026-04-07 session conversation (AAR-0008 bug investigation → pipeline optimization → strategic pivot)
- User proposal for daemon + task queue + meta-tracking architecture
- Freddie's analysis of Phase 1 ceiling and bottleneck patterns
- **Holmes research round 2026-04-07** — NotebookLM-backed findings on Q1-Q5, Q8 (see `research/2026-04-07-daemon-architecture-findings.md` and `research/2026-04-07-local-llm-findings.md`)
- **McCall architecture pass 2026-04-07** — reversed 3 BRIEF assumptions (worker model, lease semantics, file-watcher pattern); added PM2 lifecycle section; deferred local LLM per user decision
- Explicit user constraints: chokidar on Windows, `src/services/{name}/` folder convention, no UI this phase, no GPU commitment this phase, iCloud backup is piggybacking via manual backup.ts and does not conflict with brain.db

## Next Steps

1. ~~Fix MiniMax parser this session~~ — **shipped** (ASL-0001, commit `8e7a92d`)
2. ~~NotebookLM research round on open design questions~~ — **complete**
3. ~~McCall architecture pass incorporating research findings~~ — **this document**
4. **Slice task docs** (ASL-0002 → ASL-0012) sequentially — ASL-0002 task doc is being written in the same architecture pass
5. **Sequential Ryan implementation** one task at a time, starting with ASL-0002
