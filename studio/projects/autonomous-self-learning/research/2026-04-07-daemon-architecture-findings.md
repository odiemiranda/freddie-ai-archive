---
id: research-asl-daemon-architecture-2026-04-07
title: ASL Phase 2 — Daemon Architecture Research Findings
project-code: ASL
status: complete
created: 2026-04-07
author: holmes
notebook-id: ace8b091
notebook-title: ASL self-learning daemon architecture
sources-loaded: ~79 across 10 queries (2 rounds)
tags: [research, daemon, chokidar, bun, sqlite, windows, file-watcher]
---

# ASL Phase 2 — Daemon Architecture Research Findings

Covers open questions 1, 2, 3, 4, 8 from `BRIEF.md`. Questions 6 and 7 were resolved by shipped commit `8e7a92d` (ASL-0001) and are excluded from this round. Question 5 is covered in the companion `2026-04-07-local-llm-findings.md`.

## Executive Summary

**Three of the BRIEF's core architectural assumptions are contradicted by the evidence:**

1. **Bun `worker_threads` is not safe for the daemon.** As of early 2026 the implementation is still flagged "experimental" by the Bun team, has an open stability tracking issue (#15964), and ships with Windows-specific segfaults during termination (#24405). The creator himself advises against calling `.terminate()` or using temporary workers. The BRIEF's stated rationale ("avoids 200-500ms startup cost, shares brain.db connection") is real but the reliability cost is unacceptable for a daemon that must run unattended. **Recommend: child processes with IPC instead.**

2. **Chokidar v4.0.0 is broken on Windows.** Issue #1361 documents that v4.0.0 intermittently fails to detect new files on Windows 11 and mishandles atomic writes. Multiple reports confirm that downgrading to **v3.6.0** is the only known-reliable fix. The BRIEF assumes "chokidar is the library" without a version pin. **Pin chokidar@3.6.0 explicitly.**

3. **The BRIEF's lease model ("lease_until = now + 5 minutes") is an anti-pattern for variable-duration tasks.** Production systems (SQS, BullMQ, Temporal, Celery) use short initial leases plus periodic heartbeat renewal. A fixed 5-minute lease forces 5 minutes of dead time whenever a worker crashes early, and still fails if a legitimate distill run takes 6 minutes. **Recommend: 60-second lease + 30-second heartbeat.**

The research also surfaces one non-obvious Windows gotcha that is invisible in the BRIEF: **Task Scheduler silently assigns Below-Normal process priority and low I/O priority to background tasks**, making them noticeably slower than interactive sessions unless the task XML is hand-edited.

---

## Q1 — Chokidar on Windows + Obsidian Sync

**Short answer:** Chokidar works on Windows but requires careful configuration, version pinning, and awareness of known regressions. Native `fs.watch` drops events under bulk writes; polling (`usePolling: true`) is the only fully reliable fallback but at significant CPU cost.

**Confidence: HIGH (4 cited GitHub issues, multiple Obsidian forum confirmations, version-specific reproductions)**

### Evidence

**Root cause of duplicate/missed events:**
- Native Windows `ReadDirectoryChangesW` API "is known to be unreliable and drops events when many changes happen simultaneously" (Obsidian forum thread `mnaoumov` investigation, chokidar README).
- Atomic saves (write to temp file → rename) produce rapid `unlink`/`add` sequences. Chokidar's default `atomic: true` suppresses this within a 100ms window.
- Sync clients (Obsidian Sync, OneDrive, Dropbox) writing files while Obsidian is also writing can produce `(conflicted copy)` artifacts — documented in Obsidian forum thread on OneDrive conflicts.
- Antivirus software generates extra events (confirmed across multiple stack overflow answers).

**Known-broken Chokidar versions:**
- **Chokidar Issue #1361** — "New files are not detected" on Chokidar **v4.0.0** / Windows 11. Atomic write detection in v4.0.0 is broken; atomic changes fire as `unlink` without delays.
- **Fix: downgrade to v3.6.0.** Multiple developers report this is the only reliable resolution.
- **Chokidar Issue #954** — "awaitWriteFinish does not work on Windows." Users report `awaitWriteFinish` settings being ignored entirely unless `usePolling: true` is also set.
- **Chokidar Issue #175** — underlying `fs.watch` atomic writes bug (long-standing).

**Recommended configuration for ASL daemon:**
```js
chokidar.watch('vault/studio/memory/*/inbox', {
  usePolling: true,              // only reliable fallback on Windows
  interval: 500,                 // balance between CPU and latency
  awaitWriteFinish: {
    stabilityThreshold: 2000,    // wait 2s after last size change
    pollInterval: 100
  },
  atomic: true,                  // filter atomic save artifacts
  ignoreInitial: true,           // daemon reconciles state on startup separately
  ignored: /(\.swp|\.tmp|^\.|~$)/
})
```

**Obsidian-specific forum evidence:**
- User `mnaoumov` (Obsidian forum) debugged bulk copy failures to `ReadDirectoryChangesW` dropping events and recommended Obsidian itself switch to chokidar for its polling fallback.
- User `af4jm` (Obsidian forum) reported bulk VS Code find-and-replace across 248 files; Obsidian Sync only caught 29. Workaround: relaunch Obsidian to force fresh scan.

### What we'd still need to verify empirically
- Whether chokidar v3.6.0 + polling produces acceptable event accuracy when Obsidian Sync is actively pulling changes from another machine. None of the cited sources test this exact combination.
- Actual CPU overhead of polling mode on the user's machine with the ASL memory file volume (low: typically <100 files across all agent inboxes).

---

## Q2 — Bun worker_threads vs child processes

**Short answer:** **Do not use Bun `worker_threads` for the ASL daemon.** Use child processes with IPC instead. This reverses the BRIEF's explicit preference.

**Confidence: HIGH (official Bun tracking issue, specific Windows crash reports, creator's direct guidance)**

### Evidence

**Bun worker_threads production status (late 2025 / early 2026):**
- **Bun Issue #15964** — Official "Worker & worker_threads stability tracking issue." Opened by Bun's creator, catalogs numerous segfaults.
- **Bun v1.2.13 release notes (May 2025)** — Official Bun statement: *"We're not ready to mark Worker in Bun as stable yet, but we're getting closer."*
- **Bun Issue #14332** — "Segfault when using Prisma in workers on Windows." Cited in the stability tracker.
- **Bun Issue #24405** — "Crash on Windows 11." Segfaults in Bun v1.3.1 caused by VM data race during worker termination (`notifyNeedTermination`) that crashes the host process entirely. Users on v1.3.5 and v1.3.8 still report persistent crashes during routine worker cleanup.
- **Creator guidance:** Users are advised *not* to call `.terminate()` and *not* to create temporary workers. Keep workers alive forever to avoid the segfault window. This is incompatible with a daemon that must restart workers on crash.

**Missing ecosystem features:**
- **Bun Issue #23875** — `worker_threads.Worker` missing `stdout` and `stderr` options throughout 2025. This breaks `workerpool` (10M+ weekly downloads) and BullMQ. A PR to implement these options was only opened in January 2026.

**SQLite concurrency constraints:**
- Official Bun docs: "never share a single Database instance across threads." Each worker must open its own connection.
- WAL mode permits concurrent readers + single writer — the connection-per-worker model is safe under WAL.
- Known gotcha: `SQLiteError: locking protocol` on WSL 1 and certain Windows network filesystem scenarios. Local NTFS + WAL should be fine with the connection-per-worker discipline.
- Bun-specific: "Bun statically links its own SQLite build" — the `-wal` and `-shm` sidecar files are managed automatically.

**Child process crash recovery advantage:**
- A child process can be killed cleanly (SIGKILL / terminate), OS cleans up all its resources, parent daemon simply spawns a replacement.
- A worker thread crash in Bun on Windows can bring down the host process (per #24405). Parent daemon dies with the child. This is a hard disqualifier for an unattended daemon.

### Revised recommendation for BRIEF §1

Replace:
> "Worker pool inside the process using `worker_threads` (NOT child shells — avoids ~200-500ms startup cost per task, shares brain.db connection)"

With:
> "Worker pool as child processes spawned via `Bun.spawn()` with IPC for work dispatch and result reporting. Each worker opens its own SQLite connection in WAL mode. The 200-500ms startup cost is absorbed per-task but is negligible compared to the 1-8 minute pipeline runtime. A crashed worker does not take down the parent daemon."

### What we'd still need to verify empirically
- Measure actual child-process startup cost on the user's machine with Bun 1.3.x loading the ASL codebase. If it's >2s, consider a long-lived worker pool of child processes that claim tasks via the queue (instead of one child per task).
- Confirm `Bun.spawn()` IPC stability on Windows 11 for long-running child processes (not investigated in this research round).

---

## Q3 — SQLite task queue lease semantics

**Short answer:** The BRIEF's fixed 5-minute lease is a documented anti-pattern. Use a **30-60 second lease with a 15-30 second heartbeat renewal** for variable-duration tasks that range 30s-5min.

**Confidence: HIGH (AWS SQS, BullMQ, Temporal, Celery all converge on the same pattern)**

### Evidence

**Production system patterns:**

| System | Lock/lease mechanism | Default duration | Heartbeat? |
|--------|---------------------|------------------|------------|
| AWS SQS | Visibility timeout | 30 seconds | Yes — `ChangeMessageVisibility` extends; 12-hour hard cap |
| BullMQ | `lockDuration` | 30 seconds | Yes — `stalledInterval` checks expired locks |
| Temporal | `HeartbeatTimeout` | Activity-specific | Explicit — activity must call heartbeat() or fail |
| Celery | `visibility_timeout` + `task_acks_late=True` | Broker-specific | Ack on completion, not on receipt |
| Graphile Worker (anti-example) | Hardcoded 4-hour lock | 4 hours | No — forced 4-hour wait after crashes |

**Failure modes confirmed in research:**

1. **Lease too short:** Healthy worker still running when lease expires → duplicate processing by two workers → wasted compute, inconsistent state if tasks aren't idempotent.
2. **Lease too long:** Crashed worker → job sits locked for the entire remaining lease window before recovery. Graphile Worker's 4-hour lock is cited as the cautionary example.

**Why heartbeat wins for variable-duration tasks:**
- Without heartbeat, you must set the lease to the *maximum* expected task duration. For ASL that's ~8 minutes (the current synchronous pipeline ceiling). A crash 5 seconds in then forces a 7:55 wait.
- With heartbeat: short lease (e.g., 60 seconds) + heartbeat every 30 seconds. Healthy workers auto-extend. Crashed workers stop heartbeating → lease expires in 60 seconds max → fast recovery.

**SQLite-specific implementation notes:**
- Use `BEGIN IMMEDIATE` (not `BEGIN DEFERRED`) when claiming tasks to acquire the write lock immediately and avoid `SQLITE_BUSY` during the UPDATE-RETURNING pattern.
- WAL mode is mandatory: readers (status checks, observability endpoints) don't block the writer (task claimer).
- `UPDATE ... WHERE id = (SELECT id FROM ... ORDER BY created_at LIMIT 1) RETURNING` is the correct atomic-claim pattern and matches what the BRIEF already proposes.

### Revised schema for BRIEF §2

Replace `lease_until` with a heartbeat model:

```sql
CREATE TABLE tasks (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  agent TEXT NOT NULL,
  task_type TEXT NOT NULL,
  status TEXT NOT NULL,          -- 'pending' | 'claimed' | 'completed' | 'failed'
  payload TEXT,
  claimed_by TEXT,               -- worker PID
  claimed_at TEXT,
  heartbeat_at TEXT,             -- last heartbeat from worker; claim expires if stale
  lease_duration_ms INTEGER NOT NULL DEFAULT 60000,  -- e.g., 60s default
  attempts INTEGER NOT NULL DEFAULT 0,
  max_attempts INTEGER NOT NULL DEFAULT 3,
  last_error TEXT,
  trigger_reason TEXT,
  created_at TEXT NOT NULL,
  completed_at TEXT,
  duration_ms INTEGER
);
```

**Lease-expiry reclaim logic:**
```sql
-- A claim is stale if the most recent heartbeat is older than lease_duration_ms ago.
WHERE status = 'claimed'
  AND (heartbeat_at IS NULL
       OR (julianday('now') - julianday(heartbeat_at)) * 86400000 > lease_duration_ms)
```

**Worker loop:**
1. Claim task (`BEGIN IMMEDIATE` + `UPDATE ... RETURNING`).
2. Start heartbeat interval (every `lease_duration_ms / 2`, e.g., 30s).
3. Run task.
4. Stop heartbeat interval.
5. Mark task completed/failed.
6. If worker crashes: heartbeat stops, task becomes reclaimable within `lease_duration_ms`.

### What we'd still need to verify empirically
- Exact `BEGIN IMMEDIATE` contention behavior under Bun's SQLite binding when the daemon starts and 5+ child processes all attempt to claim simultaneously.
- Whether the `update-check-heartbeat` query performs well with a small tasks table (<10k rows) and an index on `(status, heartbeat_at)`.

---

## Q4 — Windows daemon lifecycle without a service

**Short answer:** Use **PM2 + `pm2-windows-startup`** for the daemon. Don't use Task Scheduler unless you XML-patch the priority (`<Priority>7</Priority>` → `<Priority>4</Priority>`) — otherwise processes run at Below-Normal priority with low I/O priority and memory priority 4.

**Confidence: HIGH (multiple independent blog posts and Stack Overflow answers confirm the Task Scheduler priority gotcha)**

### Evidence

**Option comparison:**

| Option | Admin required? | Auto-restart on crash? | Auto-start on login? | Survives logoff? | Notes |
|--------|----------------|----------------------|---------------------|------------------|-------|
| NSSM / WinSW / Servy / node-windows | Yes (system service) | Yes | Yes | Yes | Overkill for single-user; requires admin |
| Task Scheduler "At log on" | No | No (by default) | Yes | Yes if "Run whether user is logged on" | Priority gotcha — see below |
| Startup folder | No | No | Yes | No | Leaves CMD window visible; primitive |
| PM2 + `pm2-windows-startup` | No | **Yes** | Yes (via registry Run key) | **No** (halts on logoff — desired for dev machine) | Cleanest fit for single-user |

**The Task Scheduler priority gotcha (critical finding):**
- Task Scheduler silently assigns:
  - Process priority: **Below-Normal**
  - I/O priority: Low
  - Memory priority: 4 (vs 5 for interactive)
- Confirmed by Server Fault thread "Process runs slower as a scheduled task than it does interactively" and multiple independent blog posts.
- **Fix:** Right-click task → Export to XML → locate `<Priority>7</Priority>` → change to `<Priority>4</Priority>` → delete old task → re-import edited XML. Cannot be done through the Task Scheduler GUI.

**PM2 lifecycle specifics:**
- `pm2-windows-startup` writes a registry entry under `HKCU\...\Run` that launches PM2 when your specific user logs in.
- Tied to the user session: when you log out, PM2 and its managed processes halt cleanly. This matches the "single-user dev machine" scope of ASL.
- `pm2 install pm2-logrotate` handles log rotation automatically — no manual shell redirects.
- PM2 handles crash restart natively (exponential backoff, retry limits).

**Logging recommendation:**
- Route daemon stdout/stderr through PM2 → `pm2-logrotate` → rotated logs in `~/.pm2/logs/`.
- Daemon itself writes structured JSON lines to a separate `daemon.log` for the observability HTTP API to tail.

### Revised recommendation for BRIEF §1 (Process model)

Add to the "Process model" bullet list:
- **Lifecycle:** PM2-managed, registered via `pm2-windows-startup`. Auto-starts on user login, halts on user logoff (desired — the user is the only consumer).
- **Auto-restart:** PM2 handles crash-restart with exponential backoff (max 5 attempts, then manual intervention).
- **Logging:** stdout/stderr captured by PM2; `pm2-logrotate` handles rotation. Daemon writes structured events to a separate log for the observability endpoint.
- **Alternative (if PM2 is rejected):** Task Scheduler "At log on" trigger **with XML-edited `<Priority>4</Priority>`** — otherwise daemon runs at Below-Normal priority and is 2-3x slower than interactive.

### What we'd still need to verify empirically
- Whether PM2 supports Bun processes cleanly on Windows (not explicitly covered in the sources — PM2 is historically Node.js-focused). Likely works via `pm2 start bun -- src/services/self-learning-daemon/index.ts` but needs testing.
- Whether `pm2-windows-startup` still works on Windows 11 as of 2026 (last source-confirmed on Windows 10).

---

## Q8 — File watcher debounce for rapid changes

**Short answer:** Use **coalescing** (accumulate all events in a window, deduplicate, emit batch), not simple debounce (emit last event only). Window size: **500-1000ms** for the ASL use case (AI agent writing memory files), not the BRIEF's proposed 2-5 seconds.

**Confidence: HIGH (notify_debouncer_full, Webpack aggregateTimeout, Rolldown useDebounce, fswatch --one-per-batch all converge on the coalescing pattern)**

### Evidence

**Debounce vs coalesce — critical distinction:**
- **Simple debounce** = "ignore all events during window, emit only the last one." **Loses information** about which files were touched in the burst. Bad fit for batching work across multiple distinct files.
- **Coalesce** = "accumulate all events in window, deduplicate by path, emit batch when quiet." Preserves the set of affected files. This is what production watchers do.

**How production tools implement coalescing:**

| Tool | Strategy | Window default |
|------|----------|---------------|
| `notify_debouncer_full` (Rust) | Tracks file system IDs; merges multiple `Rename` events, drops `Modify` after `Create`, deduplicates | Configurable |
| Webpack | `aggregateTimeout` — delay after first change to aggregate subsequent changes into one rebuild | 20ms (Webpack default — too short for ASL) |
| Rolldown | `useDebounce` with `debounceDelay` + `debounceTickRate` — coalesces at low level | Configurable |
| fswatch `--one-per-batch` | Gathers events over `latency` window, emits count + paths once | Configurable |

**Recommended window size for ASL:**

Research finding: *"~100-200ms for fast IDE reloads, 500-1000ms for build triggers to safely batch rapid saves, and up to 2-5 seconds for slower sync/backup operations."*

The ASL case ("5-10 memory files written in 30 seconds during a reflect pass") is **not** a sync/backup operation — the agent writes files sequentially as it reflects, with typical gaps of 1-3 seconds between writes. A 2-5 second window would collapse the entire reflect pass into one task, which is actually desired, BUT:
- It also delays the task by 2-5 seconds on every single file change, even for isolated saves.
- If Obsidian Sync is pulling files from another machine, events may arrive in clumps spanning 5+ seconds.

**Recommendation:** Start with **1000ms coalescing window** measured from last event in the burst (classic trailing-edge debounce on the coalesce emit). Track the set of touched agent inboxes during the window. When emitting, enqueue one task per affected agent, not one task per file change.

### Coalescing strategy for ASL daemon

```typescript
// Pseudocode — coalesce file events into per-agent tasks
const pending = new Map<string, Set<string>>()  // agent -> set of changed file paths
let flushTimer: Timer | null = null
const WINDOW_MS = 1000

watcher.on('all', (event, path) => {
  if (event !== 'add' && event !== 'change') return
  const agent = extractAgentFromPath(path)
  if (!agent) return

  if (!pending.has(agent)) pending.set(agent, new Set())
  pending.get(agent)!.add(path)

  // Reset flush timer — classic trailing-edge debounce, but preserving all paths
  if (flushTimer) clearTimeout(flushTimer)
  flushTimer = setTimeout(flush, WINDOW_MS)
})

function flush() {
  for (const [agent, paths] of pending) {
    // One consolidate task per agent, with the set of changed files as payload
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

This pattern:
- Collapses rapid bursts per agent into one consolidate task (BRIEF's stated goal).
- Preserves the list of changed files in the payload (useful for debug/observability).
- Handles the multi-agent case (reflect pass across tala + rune + mccall → 3 distinct tasks).
- Does not rely on chokidar's `awaitWriteFinish` alone (which has Windows reliability issues — see Q1).

### Failure modes

- **Window too short (<500ms):** Multiple redundant tasks for a single reflect pass. Wastes Gemini API calls, risks duplicate distillation.
- **Window too long (>3s):** Perceived latency. Also risks Obsidian Sync burst-pulls being split across window boundaries (if sync writes 10 files in 4s and window is 2s, you get 2 tasks instead of 1).
- **Missing events entirely (chokidar v4 regression):** Use v3.6.0 per Q1 finding.

### What we'd still need to verify empirically
- Actual inter-file gap during a typical reflect pass on the user's machine. If gaps are consistently <500ms, window can be shorter.
- Whether Obsidian Sync bursts arrive within the 1s window or spread out. User's specific setup (single-machine local dev or multi-machine sync) affects this.

---

## Surprise findings that reshape BRIEF assumptions

1. **Bun worker_threads on Windows is an outright disqualifier, not a preference.** The BRIEF treats worker_threads as the default and child processes as the fallback. Invert this. The "avoid startup cost" argument is real but secondary to not crashing the daemon.

2. **Chokidar v4.0.0 has active Windows regressions.** The BRIEF says "chokidar is the library" without a version pin. Add an explicit `chokidar@^3.6.0` pin with a comment linking to Issue #1361.

3. **Fixed-duration leases are an anti-pattern.** Heartbeat-based renewal is the production standard. The BRIEF's schema needs the `heartbeat_at` column and the worker loop needs a heartbeat interval.

4. **Task Scheduler on Windows silently slows processes.** This is not mentioned anywhere in the BRIEF but will bite the daemon if Task Scheduler is chosen over PM2. The fix requires XML editing.

5. **`usePolling: true` is likely required for Obsidian Sync reliability on Windows.** This has a CPU cost but is the only documented way to avoid missed events from `ReadDirectoryChangesW` under sync-client write patterns.

## Questions we could not answer from the notebook

- **PM2 + Bun compatibility on Windows 11.** Sources discuss PM2 + Node.js and PM2 on Windows separately, but not the specific combination. **Recommendation:** 1-hour empirical test — `pm2 start bun -- <script>` on a throwaway daemon, reboot, verify auto-restart, verify graceful shutdown on logoff.
- **Polling mode CPU cost at ASL's file volume.** Sources cite "significant CPU and memory cost" but no numbers for a small memory directory (<100 files). **Recommendation:** measure during integration testing.
- **Bun child_process IPC stability on Windows 11 for long-running children.** Not covered. **Recommendation:** stress test in ASL-0005 (worker pool implementation).

## Citations

All claims are sourced from NotebookLM notebook `ace8b091` — "ASL self-learning daemon architecture" — which ingested ~79 sources across 10 research queries spanning chokidar issues, Bun issue tracker, AWS SQS docs, BullMQ docs, Temporal docs, Windows Task Scheduler forums, and file watcher documentation. The full answer text with inline bracket citations is stashed at `vault/studio/research/2026-04-07-asl-daemon-architecture/` via `research save`.

Key issues/docs cited:
- Chokidar: Issue #175, #511, #954, #1361, #1416
- Bun: Issue #14332, #15964, #23875, #24405; v1.2.13 release notes
- AWS SQS: Visibility timeout docs; ChangeMessageVisibility
- BullMQ: Stalled jobs docs; lockDuration defaults
- Temporal: HeartbeatTimeout docs
- Celery: task_acks_late + visibility_timeout
- Obsidian forum: `mnaoumov` investigation; `af4jm` bulk replace
- Windows: Server Fault priority thread; node-windows docs; Servy/NSSM comparison
- File watchers: notify_debouncer_full Rust docs; Webpack aggregateTimeout; Rolldown useDebounce; fswatch --one-per-batch

## Estimated research cost

- 10 `add-research` queries + 1 summary re-ask = 11 NotebookLM compute cycles on notebook 1.
- 5 substantive `ask` calls + 1 summary = 6 query cycles.
- Token cost not reported by the research tool. Order-of-magnitude estimate: ~500k input tokens loaded into the notebook, ~15k output tokens across all answers.
