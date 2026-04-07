---
id: ASL-0005
title: File watcher service — `src/libs/file-watcher.ts`
project-code: ASL
status: shipped
priority: P1
created: 2026-04-07
shipped: 2026-04-07
shipped-commit: 55132c8
author: mccall
implementer: ryan
reviewer: mccall
depends-on: [ASL-0003, ASL-0004]
blocks: [ASL-0007]
files:
  - src/libs/file-watcher.ts
  - src/libs/__tests__/file-watcher.test.ts
  - package.json
estimated-complexity: medium (3-5 hours)
tags: [task, library, file-watcher, chokidar, coalesce, asl-phase-2]
---

> **Production behavior — Windows polling settling delay (post-implementation finding, 2026-04-07):**
>
> `start()` includes a **600ms settling delay AFTER chokidar emits the `ready` event** before resolving. This is empirically required on Windows when `usePolling: true` is set — chokidar's `ready` event fires before its initial directory tree scan is fully populated, so files written immediately after `await start()` returns can be missed otherwise. The 600ms is `interval (500) + 100ms jitter margin`, derived from one full poll cycle.
>
> **ASL-0007 implementers:** do NOT add additional sleep or settling logic after `await watcher.start()`. The watcher handles polling settling internally. Startup reconciliation can run immediately after `start()` resolves.
>
> **Spec example correction (post-implementation, 2026-04-07):**
>
> The original spec used a bare path (`.DS_Store`) in the `WATCHER_IGNORE_PATTERN` one-liner test example. The actual regex `[\/\\]\.[^\/\\]+$` requires a path separator before the dot. Real chokidar events always pass absolute paths (which always have separators), so production behavior is correct. The bare-path example was misleading. Use a separator-prefixed path like `/memory/.DS_Store` when verifying the regex by hand.

# ASL-0005 — File watcher service

## TL;DR

Build `src/libs/file-watcher.ts` — a pure programmatic library that watches `vault/studio/memory/{agent}/inbox/` and `vault/studio/memory/shared/inbox/` for `.md` file changes, coalesces rapid bursts into per-agent batches using a 1000ms trailing-edge debounce, and enqueues one `consolidate` task per affected agent via the ASL-0003 task queue. **No daemon code, no worker pool, no CLI tool, no wiring into `autonomous-distill.ts` or `reflect.ts`.** The library is the layer ASL-0007 (daemon entry point) wires up.

The watcher uses `chokidar@^3.6.0` pinned to v3 (NOT v4 — Issue #1361 documents broken atomic-write detection on Windows 11). Configuration is locked by BRIEF §1 and research `2026-04-07-daemon-architecture-findings.md` Q1 & Q8 — do not deviate. The coalesce algorithm is copied verbatim from research Q8; it preserves the set of touched file paths across rapid bursts rather than dropping intermediate events (debounce would lose that information).

**Critical decoupling rule (locked):** the watcher does NOT import from `src/libs/agent-lifecycle.ts`. `refreshInboxCount(agent)` is called by the worker (ASL-0006) when it claims the consolidate task, not by the watcher after enqueue. See "Design decision: inbox count ownership" below.

## Context

Both upstream dependencies are shipped:

- **ASL-0003** (commit `d61f6fc`) — `src/libs/task-queue.ts` exposes `enqueueTask({ agent, taskType, triggerReason, payload? })`. The watcher calls this when a coalesce window flushes. Signature:
  ```ts
  export function enqueueTask(input: EnqueueTaskInput): EnqueueTaskResult;
  export interface EnqueueTaskInput {
    agent: string;
    taskType: TaskType;              // "consolidate" | "distill" | "full-pipeline"
    payload?: unknown;                // JSON.stringify'd internally
    triggerReason?: TriggerReason;    // "file-change" | "manual" | "schedule"
    leaseDurationMs?: number;
    maxAttempts?: number;
  }
  ```
- **ASL-0004** (commit `bab968c`) — `src/libs/agent-lifecycle.ts` exposes `refreshInboxCount(agent)`. **The watcher does NOT call this.** Ownership is pinned to the worker (ASL-0006).

Read these before touching anything:

- `vault/studio/projects/autonomous-self-learning/BRIEF.md` §1 (post-architecture-pass "File watcher" bullet, lines ~77-125) and §4 (hook changes)
- `vault/studio/projects/autonomous-self-learning/research/2026-04-07-daemon-architecture-findings.md` Q1 (chokidar config — authoritative) and Q8 (coalesce vs debounce — authoritative, includes pseudocode you will copy)
- `src/libs/task-queue.ts` — the `enqueueTask` API you are calling. Do NOT modify.
- `src/libs/agent-lifecycle.ts` — the lifecycle API. Do NOT import from it in this task.
- `package.json` — check if `chokidar` is already installed and at what version. **Confirmed state as of doc write: chokidar is NOT in `dependencies` or `devDependencies`. You MUST install it.** If between doc write and implementation someone added `chokidar@^4.x`, uninstall and re-pin to v3.6.0.
- `vault/studio/memory/` directory layout:
  ```
  vault/studio/memory/
    echo/     { archive/, inbox/, knowledge/, staged/, echo-memory.md, memory-index.json }
    freddie/  { archive/, inbox/, knowledge/, memory-index.json }
    holmes/   { ... }
    mccall/   { ... }
    penny/    { ... }
    raw/      ← NEVER WATCHED (append-only, immutable per memory-boundaries rule)
    rune/     { ... }
    ryan/     { ... }
    shared/   { archive/, inbox/, knowledge/, memory-index.json, shared-memory.md }
    sol/      { ... }
    tala/     { archive/, inbox/, knowledge/, staged/, tala-memory.md, memory-index.json }
  ```
  Note `tala/` and `echo/` have an extra `staged/` subdir — the watcher ignores it (not `inbox/`).

**Pinning rule (hard):** `chokidar@^3.6.0` is mandatory. v4.0.0 is broken on Windows 11 per Issue #1361 (intermittently fails to detect new files, mishandles atomic writes, `awaitWriteFinish` ignored). Downgrade to v3.6.0 is the only known-reliable fix. If `package.json` already has v4 when Ryan picks up this task, the doc instructs Ryan to stop and reinstall v3.6.x.

## Architecture Diagram

```mermaid
flowchart LR
    subgraph "Filesystem (watched)"
      TI[memory/tala/inbox/*.md]
      RI[memory/rune/inbox/*.md]
      HI[memory/holmes/inbox/*.md]
      SI[memory/shared/inbox/*.md]
      ETC[memory/{other}/inbox/*.md]
    end

    subgraph "Filesystem (NOT watched)"
      RAW[memory/raw/**]
      KN[memory/{agent}/knowledge/**]
      AR[memory/{agent}/archive/**]
      ST[memory/tala/staged/**]
    end

    subgraph "file-watcher.ts"
      CH[chokidar v3.6.0<br/>usePolling:true<br/>interval:500<br/>awaitWriteFinish:2000<br/>atomic:true<br/>ignoreInitial:true]
      EX[extractAgentFromPath]
      PM[pending: Map&lt;agent, Set&lt;path&gt;&gt;]
      FT[flushTimer<br/>1000ms trailing]
      FL[flush]
    end

    subgraph "Downstream (NOT this task)"
      TQ[(task-queue.ts<br/>enqueueTask)]
      W[Worker pool<br/>ASL-0006]
      AL[agent-lifecycle.ts<br/>refreshInboxCount]
    end

    TI --> CH
    RI --> CH
    HI --> CH
    SI --> CH
    ETC --> CH

    RAW -. ignored .- CH
    KN -. ignored .- CH
    AR -. ignored .- CH
    ST -. ignored .- CH

    CH -->|add/change event| EX
    EX -->|agent name| PM
    PM -->|new event| FT
    FT -->|window quiet| FL
    FL -->|one call per agent| TQ

    TQ -.->|claimed by| W
    W -.->|after claim| AL
```

## Goal

After this task ships:

1. `src/libs/file-watcher.ts` exists and exports a clean `createFileWatcher(options)` factory plus the documented types, constants, and the `extractAgentFromPath` helper.
2. `chokidar@^3.6.0` is in `package.json` dependencies, version-pinned to the v3 major line.
3. The watcher uses the exact chokidar configuration specified in BRIEF §1 / research Q1 — `usePolling: true`, `interval: 500`, `awaitWriteFinish: { stabilityThreshold: 2000, pollInterval: 100 }`, `atomic: true`, `ignoreInitial: true`, and the `ignored` regex filtering `.swp`/`.tmp`/dotfiles/`~`-suffix/`raw`/`knowledge`/`archive`.
4. The coalesce buffer matches research Q8 verbatim: per-agent `Set<string>` of changed paths, single trailing-edge timer across all agents, 1000ms window, one `enqueueTask` call per affected agent on flush.
5. A new event arriving during an in-flight flush starts a fresh pending map and timer without losing events — the in-flight flush captures its own snapshot before the first `await`.
6. The watcher does NOT import from `src/libs/agent-lifecycle.ts`. The worker owns inbox count refresh (ASL-0006).
7. The watcher does NOT watch `raw/`, `knowledge/`, `archive/`, `staged/`, or any non-`.md` file.
8. `start()` is idempotent — calling twice is a no-op. `stop()` clears the timer and pending buffer and closes the chokidar instance.
9. `getStatus()` reports `{ running, pendingAgents, lastFlushAt, eventsReceived, eventsCoalesced, tasksEnqueued }` for observability.
10. `createFileWatcher` accepts an injected `enqueueTaskFn` for testing — defaults to the real `enqueueTask` from `task-queue.ts`. Tests pass a mock so they never touch the real DB.
11. The test suite covers: `extractAgentFromPath` for all path shapes, the coalesce buffer logic (exhaustively, using simulated events), in-flight flush handling, idempotent start/stop, status reporting, and ONE integration smoke test against a real temp directory + real chokidar + mock enqueue.
12. All acceptance criteria below pass.

## Detailed Specification

### Part 0 — Dependency install (do this first)

Before writing any code, verify and install chokidar:

```bash
# 1. Check current state
grep chokidar package.json
# Expected output (current state): nothing / no match.

# 2. If missing: install v3 explicitly
bun add chokidar@^3.6.0

# 3. If ALREADY present but at v4 (broken on Windows): uninstall and reinstall v3
#    Only execute if step 1 showed a v4 match:
# bun remove chokidar
# bun add chokidar@^3.6.0

# 4. Verify the lockfile pins v3.6.x
grep -A1 '"chokidar"' package.json
# Expected: "chokidar": "^3.6.0"
```

**Rule:** after install, open `package.json` and confirm the version in `dependencies` starts with `^3.`. If it starts with `^4.` or `~4.`, STOP — Windows regression, re-pin to v3.6.0. If the version string is anything other than `^3.6.0` or a compatible `^3.x` ≥ 3.6.0, re-run `bun add chokidar@^3.6.0`.

**Do NOT add any other dependency.** `crypto.randomUUID` and `os.tmpdir()` are stdlib (already available in Bun). `path` and `fs` are already imported elsewhere in the codebase.

### Part 1 — Module header and imports

**File:** `src/libs/file-watcher.ts` (NEW)

```ts
/**
 * file-watcher.ts — File system watcher for per-agent inbox directories.
 *
 * Watches vault/studio/memory/{agent}/inbox/*.md and
 * vault/studio/memory/shared/inbox/*.md via chokidar. Coalesces rapid-fire
 * changes (e.g., a reflect pass writing 5-10 files in sequence) into a single
 * `consolidate` task per affected agent, enqueued via the ASL-0003 task queue.
 *
 * Locked design decisions:
 *   - chokidar v3.6.0 pin (NOT v4 — Issue #1361 Windows regression)
 *   - usePolling: true (non-negotiable; native ReadDirectoryChangesW drops
 *     events under Obsidian Sync burst writes on Windows)
 *   - 1000ms trailing-edge coalesce window (research Q8)
 *   - One single flush timer across ALL agents, not one per agent — keeps
 *     timer management simple and matches the BRIEF pseudocode verbatim
 *   - Decoupled from agent-lifecycle.ts — does NOT call refreshInboxCount.
 *     The worker (ASL-0006) owns that update after claiming the task.
 *
 * This library does NOT:
 *   - Start a daemon (ASL-0007 owns the entry point)
 *   - Spawn workers or claim tasks (ASL-0006)
 *   - Reconcile existing files at startup — ignoreInitial: true means the
 *     watcher fires only for NEW events. Startup reconciliation is a daemon
 *     concern, not a watcher concern.
 *   - Import from agent-lifecycle.ts
 *   - Modify autonomous-distill.ts, reflect.ts, or any hook script
 *
 * Windows polling CPU cost (documented tradeoff):
 *   usePolling: true walks every watched file every 500ms. For the ASL
 *   memory directory (<100 .md files total across all agent inboxes), this
 *   is negligible (<1% CPU on a modern desktop per chokidar benchmarks).
 *   The alternative — native ReadDirectoryChangesW — drops events under
 *   bulk writes from Obsidian Sync and is documented-unreliable for this
 *   use case (research Q1, multiple Obsidian forum confirmations).
 */

import chokidar, { type FSWatcher } from "chokidar";
import { basename, dirname, join, resolve, sep } from "path";
import { enqueueTask } from "./task-queue.js";
import type { EnqueueTaskResult, TriggerReason } from "./task-queue.js";
import { fromRoot } from "./paths.js";
```

**Important:** static top-of-file import for chokidar. Do NOT use `await import("chokidar")` — Bun handles the static import cleanly and dynamic import obscures the version pin at the type layer. The `FSWatcher` type comes from chokidar v3 — it is the type of `chokidar.watch(...)` return value.

### Part 2 — Constants

```ts
/**
 * Default coalesce window in milliseconds. Research Q8 settled on 1000ms:
 *   - Short enough to keep perceived latency acceptable for isolated saves
 *   - Long enough to absorb the typical 1-3 second gaps between files in
 *     a reflect pass (the whole pass collapses to ONE task)
 *   - Matches the BRIEF §1 "File watcher" pseudocode exactly
 *
 * Tunable via FileWatcherOptions.coalesceWindowMs if empirical measurement
 * during daemon runtime shows bursts need a longer window.
 */
export const DEFAULT_COALESCE_WINDOW_MS = 1000;

/**
 * Regex for chokidar `ignored` option.
 *
 * Filters:
 *   - .swp, .tmp (editor temp files)
 *   - Dotfiles (^.) and ~-suffix backup files
 *   - The raw/ subtree (append-only per memory-boundaries rule)
 *   - knowledge/ subtree (changes detected by workers via isDistillStale,
 *     not by the watcher)
 *   - archive/ subtree (historical, not actionable)
 *
 * Note: this is a single regex passed to chokidar — it tests against the
 * full path, not just the basename. Forward-slash and back-slash are both
 * matched via the character class so Windows paths work.
 *
 * staged/ is NOT in this regex because the glob pattern already restricts
 * to inbox/ subtrees — nothing from staged/ would match a glob like
 * */inbox/*.md anyway. Belt-and-braces would add it to the regex, but the
 * glob filter is the primary mechanism.
 */
export const WATCHER_IGNORE_PATTERN =
  /(\.swp$|\.tmp$|[\/\\]\.[^\/\\]+$|~$|[\/\\]raw[\/\\]|[\/\\]knowledge[\/\\]|[\/\\]archive[\/\\])/;
```

**Regex note for Ryan:** the character class `[\/\\]` matches both forward slash and back slash. On Windows chokidar may produce either separator depending on whether paths came from globs or raw reads. Test the regex in a one-liner before claiming done:

```bash
bun -e "const r = /(\.swp$|\.tmp$|[\/\\\\]\.[^\/\\\\]+$|~$|[\/\\\\]raw[\/\\\\]|[\/\\\\]knowledge[\/\\\\]|[\/\\\\]archive[\/\\\\])/; for (const p of ['a.md', '.DS_Store', 'foo.swp', 'vault/studio/memory/raw/2026.md', 'vault\\\\studio\\\\memory\\\\tala\\\\knowledge\\\\a.md', 'vault/studio/memory/tala/inbox/a.md', 'foo~']) console.log(p, '=>', r.test(p) ? 'IGNORED' : 'WATCHED')"
```

Expected output:
```
a.md => WATCHED
.DS_Store => IGNORED
foo.swp => IGNORED
vault/studio/memory/raw/2026.md => IGNORED
vault\studio\memory\tala\knowledge\a.md => IGNORED
vault/studio/memory/tala/inbox/a.md => WATCHED
foo~ => IGNORED
```

If any of these fail, fix the regex before continuing. The spec is "inbox paths watched, everything else ignored."

### Part 3 — Public types

```ts
/**
 * Options passed to createFileWatcher.
 *
 * vaultMemoryDir is REQUIRED — the factory needs to know which memory tree
 * to watch. Production wiring passes fromRoot("vault", "studio", "memory");
 * tests pass a temp directory. No default because leaking a default of
 * "vault/studio/memory" into tests would pollute the real vault.
 */
export interface FileWatcherOptions {
  /** Absolute path to the memory root. e.g., fromRoot("vault","studio","memory"). */
  vaultMemoryDir: string;

  /** Coalesce window in ms. Defaults to DEFAULT_COALESCE_WINDOW_MS (1000). */
  coalesceWindowMs?: number;

  /**
   * Injected enqueue function for testing. Defaults to the real
   * task-queue.ts export. Tests pass a mock (a jest.fn-style capture)
   * so tests never hit the real brain.db.
   */
  enqueueTaskFn?: typeof enqueueTask;
}

/**
 * Opaque handle returned by createFileWatcher. Callers hold this for the
 * lifetime of the watcher and call start/stop on it.
 */
export interface FileWatcher {
  /**
   * Begin watching. Idempotent — a second call while already running is
   * a no-op. Returns a promise that resolves once chokidar's initial
   * "ready" event has fired.
   */
  start(): Promise<void>;

  /**
   * Stop watching. Clears any pending flush timer, empties the pending
   * buffer WITHOUT flushing (pending events are dropped on stop), and
   * closes the chokidar instance. Idempotent — stopping an already-stopped
   * watcher is a no-op.
   *
   * Rationale for dropping pending on stop: the daemon shutdown path is
   * "stop watcher → drain workers → exit." Any pending file events will
   * be re-detected on daemon restart (chokidar's ignoreInitial is true,
   * but the daemon will do its own startup reconciliation in ASL-0007 to
   * catch files that changed between sessions). Flushing on stop would
   * enqueue tasks that might never be processed if the DB is being
   * checkpointed during shutdown.
   */
  stop(): Promise<void>;

  /**
   * Observability snapshot. Read-only — does not change watcher state.
   * Used by the daemon's HTTP observability endpoint (ASL-0007).
   */
  getStatus(): WatcherStatus;
}

/**
 * Observability snapshot.
 *
 * - running: true after start() has resolved and stop() has not been called
 * - pendingAgents: the keys of the current pending map (agents with unflushed events)
 * - lastFlushAt: ISO timestamp of the most recent flush, or null if never
 * - eventsReceived: total count of 'add'/'change' events received (post-ignored)
 * - eventsCoalesced: count of events that were merged into an already-pending agent
 *                    (i.e., the event was not the first one in its window for its agent)
 * - tasksEnqueued: total count of enqueueTask calls made
 */
export interface WatcherStatus {
  running: boolean;
  pendingAgents: string[];
  lastFlushAt: string | null;
  eventsReceived: number;
  eventsCoalesced: number;
  tasksEnqueued: number;
}
```

### Part 4 — extractAgentFromPath helper

This is the only exported helper besides the factory. It is a pure function and is the first thing to write and test — the rest of the watcher logic depends on it producing correct agent names.

```ts
/**
 * Extract the agent name from an inbox file path.
 *
 * Expected path shapes (relative to vaultMemoryDir):
 *   {agent}/inbox/{file}.md        → returns "{agent}"
 *   shared/inbox/{file}.md         → returns "shared"
 *
 * Returns null if:
 *   - path is not under vaultMemoryDir
 *   - path is not inside an inbox/ directory
 *   - path is inside raw/, knowledge/, archive/, or any other non-inbox subtree
 *   - path is at an unexpected depth (e.g., vaultMemoryDir/inbox/file.md — no agent dir)
 *
 * Path separators: handles both forward slash (Linux, macOS, chokidar on
 * macOS/Linux) and back slash (chokidar on Windows without normalization).
 * Normalizes via `resolve` so equivalent paths produce the same result.
 *
 * Example:
 *   extractAgentFromPath(
 *     "D:/PROJECTS/freddie-ai/vault/studio/memory/tala/inbox/2026-04-07-foo.md",
 *     "D:/PROJECTS/freddie-ai/vault/studio/memory"
 *   ) === "tala"
 *
 *   extractAgentFromPath(
 *     "D:/PROJECTS/freddie-ai/vault/studio/memory/shared/inbox/foo.md",
 *     "D:/PROJECTS/freddie-ai/vault/studio/memory"
 *   ) === "shared"
 *
 *   extractAgentFromPath(
 *     "D:/PROJECTS/freddie-ai/vault/studio/memory/tala/knowledge/foo.md",
 *     "D:/PROJECTS/freddie-ai/vault/studio/memory"
 *   ) === null
 */
export function extractAgentFromPath(
  absPath: string,
  vaultMemoryDir: string,
): string | null {
  // Normalize both sides so Windows back-slash vs forward-slash doesn't
  // trip up the string compare. `resolve` also collapses "../" and "./"
  // segments, which matters if a pathological chokidar event slipped one
  // through (unlikely but cheap to defend against).
  const normalizedPath = resolve(absPath);
  const normalizedRoot = resolve(vaultMemoryDir);

  // Must be inside the memory root. Use sep-terminated prefix to avoid
  // false matches like ".../memory-backup/...".
  const prefix = normalizedRoot + sep;
  if (!normalizedPath.startsWith(prefix)) return null;

  // The path after the prefix should be "{agent}/inbox/{file.md}" or
  // "{agent}/inbox/{subdir}/{file.md}". We split on the OS separator.
  const relative = normalizedPath.slice(prefix.length);
  const parts = relative.split(sep);

  // Minimum shape: [agent, "inbox", filename] → 3 parts
  if (parts.length < 3) return null;

  const agent = parts[0];
  const second = parts[1];

  // Must be inside /inbox/
  if (second !== "inbox") return null;

  // Reject suspicious agent names — an empty string or "." would bypass
  // the earlier prefix check if someone crafted a weird path.
  if (!agent || agent === "." || agent === "..") return null;

  // Reject the raw pseudo-agent explicitly, even though chokidar's ignored
  // regex should have filtered it. Belt-and-braces.
  if (agent === "raw") return null;

  return agent;
}
```

**Why this is a separate exported function:** Ryan tests it exhaustively before any chokidar integration. Once it's correct, the rest of the watcher has a stable foundation. The helper has no runtime dependencies (pure string manipulation) so tests run instantly.

### Part 5 — The factory

```ts
/**
 * Internal state for a single watcher instance. Encapsulated in the closure
 * returned by createFileWatcher so multiple watchers can run side-by-side
 * in tests without sharing state.
 */
interface WatcherState {
  chokidarInstance: FSWatcher | null;
  running: boolean;
  pending: Map<string, Set<string>>;
  flushTimer: ReturnType<typeof setTimeout> | null;
  lastFlushAt: string | null;
  eventsReceived: number;
  eventsCoalesced: number;
  tasksEnqueued: number;
  startPromise: Promise<void> | null;   // for start() idempotency
}

/**
 * Create a file watcher bound to a specific memory directory.
 *
 * Production wiring (in ASL-0007 daemon):
 *   const watcher = createFileWatcher({
 *     vaultMemoryDir: fromRoot("vault", "studio", "memory"),
 *   });
 *   await watcher.start();
 *   // … daemon runs …
 *   await watcher.stop();
 *
 * Test wiring:
 *   const calls: Array<{agent: string, taskType: string, payload: unknown}> = [];
 *   const mockEnqueue = (input) => {
 *     calls.push(input);
 *     return { id: 1, task: {...} } as EnqueueTaskResult;
 *   };
 *   const watcher = createFileWatcher({
 *     vaultMemoryDir: tempDir,
 *     coalesceWindowMs: 100,
 *     enqueueTaskFn: mockEnqueue,
 *   });
 */
export function createFileWatcher(options: FileWatcherOptions): FileWatcher {
  const vaultMemoryDir = resolve(options.vaultMemoryDir);
  const coalesceWindowMs = options.coalesceWindowMs ?? DEFAULT_COALESCE_WINDOW_MS;
  const enqueueFn = options.enqueueTaskFn ?? enqueueTask;

  const state: WatcherState = {
    chokidarInstance: null,
    running: false,
    pending: new Map(),
    flushTimer: null,
    lastFlushAt: null,
    eventsReceived: 0,
    eventsCoalesced: 0,
    tasksEnqueued: 0,
    startPromise: null,
  };

  // ─── flush ─────────────────────────────────────────────────────
  /**
   * Drain the pending map and enqueue one task per agent.
   *
   * In-flight flush handling (critical):
   *   - Capture the current pending map into a local variable
   *   - Reset state.pending to a fresh empty Map BEFORE the enqueue loop
   *   - Clear state.flushTimer so a new event starting during the loop
   *     begins a new window (not the continuation of this one)
   *
   * Why: enqueueTask is synchronous today (bun:sqlite prepare+run), but
   * if it ever becomes async (e.g., we wrap it in a retry helper), a new
   * event could arrive between the await and the loop iteration. Capturing
   * the snapshot up front means the new event lands in the fresh map and
   * the current flush finishes its captured set. No event is ever lost.
   */
  async function flush(): Promise<void> {
    const snapshot = state.pending;
    state.pending = new Map();
    state.flushTimer = null;

    if (snapshot.size === 0) return;

    for (const [agent, paths] of snapshot) {
      try {
        enqueueFn({
          agent,
          taskType: "consolidate",
          triggerReason: "file-change" as TriggerReason,
          payload: {
            changedFiles: [...paths],
            window: coalesceWindowMs,
          },
        });
        state.tasksEnqueued += 1;
      } catch (err) {
        // Failure isolation: an enqueue error for one agent must not
        // block the enqueue for other agents in the same snapshot.
        // Log to stderr (the daemon's stderr is captured by PM2 logs)
        // and continue. The daemon's observability layer will surface
        // the failure via tasksEnqueued vs eventsReceived drift.
        console.error(
          `[file-watcher] enqueueTask failed for agent=${agent}: ${(err as Error).message}`,
        );
      }
    }

    state.lastFlushAt = new Date().toISOString();
  }

  // ─── scheduleFlush ─────────────────────────────────────────────
  /**
   * Reset the flush timer to fire at now + coalesceWindowMs. Called on
   * every matching event — this is the "trailing edge" behavior: the
   * flush only fires once the stream of events has been quiet for the
   * full window.
   */
  function scheduleFlush(): void {
    if (state.flushTimer) {
      clearTimeout(state.flushTimer);
    }
    state.flushTimer = setTimeout(() => {
      // flush is async but we don't await it here — the timer callback is
      // fire-and-forget. Any error inside flush is caught by the try/catch
      // around enqueueFn; flush itself never rejects.
      void flush();
    }, coalesceWindowMs);
  }

  // ─── handleEvent ───────────────────────────────────────────────
  /**
   * Called from the chokidar 'all' listener. Filters events, extracts
   * the agent, and adds the path to the pending map.
   */
  function handleEvent(event: string, absPath: string): void {
    // Only care about add + change. unlink means a file was removed —
    // consolidation should not run on an empty or vanishing inbox. If
    // the file was moved (unlink + add), the add event will fire after
    // the atomic window and will enqueue normally.
    if (event !== "add" && event !== "change") return;

    // Only .md files. chokidar's glob already restricts this but the
    // ignored regex and additional filters can let edge cases through.
    if (!absPath.endsWith(".md")) return;

    const agent = extractAgentFromPath(absPath, vaultMemoryDir);
    if (!agent) return;

    state.eventsReceived += 1;

    let set = state.pending.get(agent);
    if (!set) {
      set = new Set();
      state.pending.set(agent, set);
    } else {
      // The agent already has pending events — this one is being coalesced
      // into the existing batch. Count it for observability.
      state.eventsCoalesced += 1;
    }

    // Set.add is idempotent on the file path; re-changes to the same file
    // within a window don't double-count. That's desired — the payload
    // carries the SET of changed files, not a list with duplicates.
    set.add(absPath);

    scheduleFlush();
  }

  // ─── start ─────────────────────────────────────────────────────
  async function start(): Promise<void> {
    // Idempotent: second call while running returns the same promise
    // (resolves immediately if start already finished).
    if (state.startPromise) return state.startPromise;

    state.startPromise = (async () => {
      // Glob patterns — watch every per-agent inbox AND shared inbox.
      // Using a glob means chokidar discovers agent dirs at runtime, so
      // new agent directories created after start() still get watched
      // (chokidar v3 handles mid-run dir creation when usePolling is on).
      const globPatterns = [
        join(vaultMemoryDir, "*", "inbox", "*.md"),
        join(vaultMemoryDir, "shared", "inbox", "*.md"),
        // Note: "*/inbox/*.md" already matches "shared/inbox/*.md" — the
        // second pattern is defensive belt-and-braces in case glob
        // resolution treats "shared" specially. Chokidar deduplicates
        // overlapping patterns internally.
      ];

      state.chokidarInstance = chokidar.watch(globPatterns, {
        usePolling: true,           // LOCKED — BRIEF §1 + research Q1
        interval: 500,              // LOCKED — 500ms poll
        awaitWriteFinish: {
          stabilityThreshold: 2000, // LOCKED — 2s size-stable wait
          pollInterval: 100,
        },
        atomic: true,               // LOCKED — filter unlink+add bursts
        ignoreInitial: true,        // LOCKED — daemon reconciles separately
        ignored: WATCHER_IGNORE_PATTERN,
      });

      state.chokidarInstance.on("all", handleEvent);

      // Wait for chokidar's initial ready event before resolving start().
      // This guarantees the caller that the watcher is actually armed.
      await new Promise<void>((resolvePromise, rejectPromise) => {
        state.chokidarInstance!.once("ready", () => resolvePromise());
        state.chokidarInstance!.once("error", (err) => rejectPromise(err));
      });

      state.running = true;
    })();

    return state.startPromise;
  }

  // ─── stop ──────────────────────────────────────────────────────
  async function stop(): Promise<void> {
    if (!state.running && !state.startPromise) return;

    // Cancel any pending flush and drop the buffer — see stop() comment
    // in the FileWatcher interface for the rationale.
    if (state.flushTimer) {
      clearTimeout(state.flushTimer);
      state.flushTimer = null;
    }
    state.pending = new Map();

    if (state.chokidarInstance) {
      await state.chokidarInstance.close();
      state.chokidarInstance = null;
    }

    state.running = false;
    state.startPromise = null;
  }

  // ─── getStatus ─────────────────────────────────────────────────
  function getStatus(): WatcherStatus {
    return {
      running: state.running,
      pendingAgents: [...state.pending.keys()],
      lastFlushAt: state.lastFlushAt,
      eventsReceived: state.eventsReceived,
      eventsCoalesced: state.eventsCoalesced,
      tasksEnqueued: state.tasksEnqueued,
    };
  }

  return { start, stop, getStatus };
}
```

**Critical implementation notes for Ryan:**

1. **The closure-captured `state` object is the source of truth.** Do not move fields to module-level variables — that would mean two watchers in the same process share state. Tests create and tear down multiple watchers; they must be independent.

2. **Single timer across all agents.** `scheduleFlush` resets ONE timer; it does not maintain a per-agent timer. If events for tala arrive at t=0 and events for rune arrive at t=500ms, the flush fires at t=1500ms with BOTH agents in the snapshot. This is intentional — matches research Q8 pseudocode.

3. **The snapshot-capture pattern in `flush` is load-bearing.** You MUST capture `state.pending` into `snapshot` and reset `state.pending = new Map()` BEFORE the enqueue loop. Doing it the other way around (enqueue, then reset) loses events that arrive during the loop.

4. **`enqueueFn` is called synchronously inside the loop** — there is no `await` before it. This matches the current synchronous signature of `task-queue.ts`. If the signature ever becomes async, the capture pattern already handles it correctly.

5. **`chokidar.watch` returns an `FSWatcher`** — the `close()` method is async in v3 and must be awaited. Do not call `.close()` without await; it leaks file handles.

### Part 6 — Test file

**File:** `src/libs/__tests__/file-watcher.test.ts` (NEW)

The test strategy has three layers:

1. **Pure unit tests for `extractAgentFromPath`** — no fs, no chokidar, no timers. Runs instantly.
2. **Coalesce logic unit tests** — simulate events by calling a test helper that mimics what the chokidar listener would do, bypass chokidar entirely. This is the BULK of the test coverage.
3. **ONE integration smoke test** — real chokidar, real temp directory, real file writes, mock enqueue. Verifies the end-to-end wiring actually fires.

To make layer 2 possible, the library must expose the event handler OR accept an internal hook for tests. **Preferred approach:** tests use the public `createFileWatcher` factory and inject a mock `enqueueTaskFn`, then exercise the watcher by directly emitting events to the chokidar instance via `state.chokidarInstance?.emit("add", path)`. But this requires access to the internal state.

**Cleaner alternative (decision: use this):** extract the coalesce buffer + flush logic into a tiny internal class that the factory wraps, AND export a `_createCoalesceBufferForTest` escape hatch for direct access.

```ts
/**
 * Internal escape hatch for tests. Not part of the public API.
 * Exposes the pure coalesce buffer so tests can exercise the state machine
 * without standing up chokidar.
 *
 * @internal
 */
export function _createCoalesceBufferForTest(options: {
  coalesceWindowMs: number;
  enqueueTaskFn: typeof enqueueTask;
}): {
  handleEvent: (event: string, absPath: string, agent: string) => void;
  flushNow: () => Promise<void>;
  getPending: () => Map<string, Set<string>>;
  getStatus: () => { eventsReceived: number; eventsCoalesced: number; tasksEnqueued: number; lastFlushAt: string | null };
  stop: () => void;
};
```

Implement it by extracting the `state` + `handleEvent` + `flush` + `scheduleFlush` logic into a small factory that the main `createFileWatcher` wraps. Both the public factory and the test escape hatch produce the same state machine — one wrapped in chokidar, one standalone.

**Do NOT let the test escape hatch leak into production code paths.** Its only consumer is the test file. The `@internal` JSDoc tag signals this.

**Test imports and setup:**

```ts
import { test, expect, beforeAll, beforeEach, afterEach, afterAll, describe } from "bun:test";
import { mkdirSync, writeFileSync, rmSync, existsSync } from "fs";
import { join, resolve, sep } from "path";
import { tmpdir } from "os";
import { randomUUID } from "crypto";
import {
  createFileWatcher,
  extractAgentFromPath,
  DEFAULT_COALESCE_WINDOW_MS,
  WATCHER_IGNORE_PATTERN,
  _createCoalesceBufferForTest,
  type FileWatcher,
} from "../file-watcher.js";
import type { EnqueueTaskInput, EnqueueTaskResult, Task } from "../task-queue.js";

// Test-only fake enqueue that captures calls into an array.
type EnqueueCapture = Array<EnqueueTaskInput>;

function makeMockEnqueue(calls: EnqueueCapture): (input: EnqueueTaskInput) => EnqueueTaskResult {
  let idCounter = 1;
  return (input: EnqueueTaskInput) => {
    calls.push(input);
    const fakeTask: Task = {
      id: idCounter++,
      agent: input.agent,
      taskType: input.taskType,
      status: "pending",
      payload: typeof input.payload === "string" ? input.payload : JSON.stringify(input.payload ?? null),
      claimedBy: null,
      claimedAt: null,
      heartbeatAt: null,
      leaseDurationMs: input.leaseDurationMs ?? 60_000,
      attempts: 0,
      maxAttempts: input.maxAttempts ?? 3,
      lastError: null,
      triggerReason: input.triggerReason ?? null,
      createdAt: new Date().toISOString(),
      completedAt: null,
      durationMs: null,
    };
    return { id: fakeTask.id, task: fakeTask };
  };
}

// Temp directories are tracked in a module-level set so afterAll can nuke them
// even if an individual test crashes mid-run.
const TEMP_DIRS: string[] = [];

function makeTempMemoryDir(): string {
  const dir = join(tmpdir(), `asl-0005-${Date.now()}-${randomUUID()}`);
  mkdirSync(dir, { recursive: true });
  TEMP_DIRS.push(dir);
  return dir;
}

afterAll(() => {
  for (const dir of TEMP_DIRS) {
    try {
      rmSync(dir, { recursive: true, force: true });
    } catch {
      // Ignore — best effort cleanup. Test runner exit will clean OS temp on reboot.
    }
  }
});
```

**Test list:**

| # | Group | Test name | What it asserts |
|---|-------|-----------|-----------------|
| 1 | extract | `extractAgentFromPath returns agent for per-agent inbox path` | `extractAgentFromPath("/root/tala/inbox/foo.md", "/root")` === `"tala"` |
| 2 | extract | `extractAgentFromPath returns "shared" for shared inbox` | `extractAgentFromPath("/root/shared/inbox/foo.md", "/root")` === `"shared"` |
| 3 | extract | `extractAgentFromPath returns null for knowledge subdir` | `extractAgentFromPath("/root/tala/knowledge/foo.md", "/root")` === `null` |
| 4 | extract | `extractAgentFromPath returns null for archive subdir` | `extractAgentFromPath("/root/tala/archive/foo.md", "/root")` === `null` |
| 5 | extract | `extractAgentFromPath returns null for raw subdir` | `extractAgentFromPath("/root/raw/foo.md", "/root")` === `null` |
| 6 | extract | `extractAgentFromPath returns null for staged subdir` | `extractAgentFromPath("/root/tala/staged/foo.md", "/root")` === `null` (not inbox) |
| 7 | extract | `extractAgentFromPath returns null for path outside vaultMemoryDir` | `extractAgentFromPath("/other/tala/inbox/foo.md", "/root")` === `null` |
| 8 | extract | `extractAgentFromPath handles Windows back-slash paths` | Use `sep`-joined paths — verify tala extracts on both platforms |
| 9 | extract | `extractAgentFromPath rejects paths with only vaultMemoryDir + inbox + file` | `extractAgentFromPath("/root/inbox/foo.md", "/root")` === `null` (no agent component) |
| 10 | extract | `extractAgentFromPath handles nested inbox files` | `extractAgentFromPath("/root/tala/inbox/sub/foo.md", "/root")` === `"tala"` (parts[1] is still "inbox") |
| 11 | coalesce | `single event for one agent enqueues once after window` | Call `handleEvent("add", "/root/tala/inbox/a.md", "tala")`, wait > coalesceWindowMs, expect exactly 1 enqueue call with agent="tala" and changedFiles containing "/root/tala/inbox/a.md" |
| 12 | coalesce | `five events for one agent within window coalesce to one enqueue` | Fire 5 `handleEvent` calls for tala in rapid succession (paths a.md..e.md), wait > window, expect 1 enqueue call with changedFiles length 5 |
| 13 | coalesce | `events for two agents within window fire two enqueues` | Fire events for tala (a.md) and rune (b.md) within window, wait, expect 2 enqueue calls — one per agent — each with changedFiles length 1 |
| 14 | coalesce | `events outside window fire multiple flushes` | Fire event for tala, wait > window, fire another for tala, wait > window. Expect 2 separate enqueue calls, each with changedFiles length 1 |
| 15 | coalesce | `timer resets on new events` | Fire event at t=0, fire another at t=(window/2), wait until t=window. Expect NO enqueue yet. Wait until t=window + (window/2). Expect 1 enqueue with both paths. (Verifies trailing-edge reset behavior.) |
| 16 | coalesce | `same path repeated in window appears once in payload` | Fire 3 events for `/root/tala/inbox/a.md`, wait > window. Expect 1 enqueue with changedFiles === ["/root/tala/inbox/a.md"] (length 1, set dedup). |
| 17 | coalesce | `eventsCoalesced counts merges, not first events` | Fire 3 events for tala. After flush, status.eventsReceived === 3, status.eventsCoalesced === 2 (events 2 and 3 merged into the existing tala set). |
| 18 | coalesce | `unlink events are ignored` | Fire `handleEvent("unlink", ...)`, wait > window. Expect 0 enqueue calls. |
| 19 | coalesce | `non-md events are ignored at the buffer level` | Fire `handleEvent("add", "/root/tala/inbox/foo.txt", "tala")`, wait > window. Expect 0 enqueue calls. (Buffer double-checks `.md` suffix even though chokidar globs restrict it.) |
| 20 | coalesce | `in-flight flush starts new pending map cleanly` | Use an enqueue mock that calls `handleEvent` for a NEW path during its own invocation. Verify the new event lands in a fresh pending map, not the in-flight snapshot. Verify BOTH end up flushed (first with the original path, second on the next window). |
| 21 | coalesce | `tasksEnqueued increments per successful enqueue` | Fire events for 3 agents, wait > window. status.tasksEnqueued === 3. |
| 22 | coalesce | `enqueue failure for one agent does not block others` | Use a mock enqueue that throws when agent === "badagent". Fire events for tala, badagent, rune. Wait > window. Expect 2 successful enqueues (tala, rune) captured in the non-throwing side, badagent's throw logged to console but not propagated. |
| 23 | factory | `createFileWatcher start is idempotent` | Call start() twice, only ONE chokidar watcher exists (verify via smoke test or state check — can inspect `state.startPromise` reuse). |
| 24 | factory | `createFileWatcher stop clears timer and pending buffer` | Fire events, call stop() BEFORE window expires. Expect 0 enqueue calls. Fire no more events. Verify getStatus().pendingAgents is empty and getStatus().running is false. |
| 25 | factory | `createFileWatcher stop is idempotent` | Start, stop, stop again. No throw. |
| 26 | factory | `getStatus reports correct counts and running state` | Verify every field transitions correctly: running false → true after start, back to false after stop; lastFlushAt null before first flush then ISO string after; eventsReceived increments on each event; tasksEnqueued increments per enqueue. |
| 27 | factory | `WATCHER_IGNORE_PATTERN matches expected paths` | Unit test the regex constant directly — verify .swp, .tmp, dotfiles, ~-suffix, raw/, knowledge/, archive/ all match; inbox/*.md does NOT match. Cover both forward and back slash separators. |
| 28 | integration | `SMOKE: real chokidar detects a new file and enqueues` | Create temp dir with `tala/inbox/` subtree, start watcher with coalesceWindowMs=500, write a new `.md` file into `tala/inbox/`, wait 500ms + 1500ms margin (polling interval + stabilityThreshold + coalesce), verify mock enqueue was called once with agent="tala" and changedFiles containing the file path. ONE integration test only. |

**Test rules (read every line):**

- **NO real filesystem watching in unit tests except #28.** Tests 1-27 all use either pure path manipulation (`extractAgentFromPath`) or the `_createCoalesceBufferForTest` escape hatch. Chokidar is only started in test #28.
- **NO `vault/studio/memory/` writes from tests.** All file fixtures go into temp dirs under `os.tmpdir()`.
- **NO writes to the real brain.db.** All tests use `makeMockEnqueue` — the mock never touches `task-queue.ts`'s real code path. Verify by NOT importing `initBrain` in the test file. If the test imports `initBrain`, you have a coupling bug — remove it.
- **Temp dirs are cleaned up in afterAll.** `rmSync(dir, { recursive: true, force: true })`. Wrap in try/catch to survive Windows file-lock races on cleanup.
- **Use a SHORT coalesceWindowMs in tests.** 50-100ms. Real window (1000ms) would make the suite take tens of seconds. Tests verify the algorithm, not the specific window value.
- **For the in-flight flush test (#20):** make the mock enqueue synchronously call `bufferHandle.handleEvent` back into the buffer with a NEW path and DIFFERENT agent. This is the tightest test of the snapshot-capture pattern. If the implementation does NOT capture `pending` into a local variable before the loop, this test will show the new event being lost OR being erroneously flushed in the same batch.
- **Test fixture paths use `sep`-joined strings** for cross-platform correctness. Example: `join(root, "tala", "inbox", "a.md")`.
- **Integration test (#28) timing:** after writing the file, await `new Promise(r => setTimeout(r, coalesceWindowMs + 2500))`. The 2500ms covers: 500ms polling interval + 2000ms awaitWriteFinish stabilityThreshold + the coalesce window. Any less and the test flakes.
- **Integration test cleanup:** always `await watcher.stop()` in `afterEach` even if the test throws, and always `rmSync` the temp dir.
- **Bun test segfault fallback (Windows known issue):** if `bun test src/libs/__tests__/file-watcher.test.ts` segfaults, fall back to `bun run src/libs/__tests__/file-watcher.test.ts` (Bun's test APIs work at runtime). Document whichever path works in Reporting Back.

### Part 7 — Manual smoke checks

After implementation, Ryan runs:

```bash
# 1. Verify the chokidar pin actually landed
grep -A1 '"chokidar"' package.json
# Expected: "chokidar": "^3.6.0"

# 2. Run the new test file
bun test src/libs/__tests__/file-watcher.test.ts

# 3. Confirm brain.db is untouched (no cross-coupling leaked)
bun run tool brain --check

# 4. Grep-check the decoupling rules
grep -n "agent-lifecycle" src/libs/file-watcher.ts
# Expected: NO OUTPUT (empty grep)

grep -n "agent-lifecycle" src/libs/__tests__/file-watcher.test.ts
# Expected: NO OUTPUT (empty grep)

grep -n "initBrain" src/libs/__tests__/file-watcher.test.ts
# Expected: NO OUTPUT (empty grep) — test must not touch real brain.db

# 5. Quick real-watcher smoke with a temp dir (one-liner — NOT committed)
bun -e "
  import('./src/libs/file-watcher.ts').then(async m => {
    const { mkdirSync, writeFileSync, rmSync } = await import('fs');
    const { join } = await import('path');
    const { tmpdir } = await import('os');
    const dir = join(tmpdir(), 'asl-0005-smoke-' + Date.now());
    mkdirSync(join(dir, 'tala', 'inbox'), { recursive: true });
    const calls = [];
    const watcher = m.createFileWatcher({
      vaultMemoryDir: dir,
      coalesceWindowMs: 500,
      enqueueTaskFn: (input) => { calls.push(input); return { id: 1, task: null }; },
    });
    await watcher.start();
    writeFileSync(join(dir, 'tala', 'inbox', 'a.md'), '# test');
    await new Promise(r => setTimeout(r, 3500));
    await watcher.stop();
    console.log('calls:', JSON.stringify(calls, null, 2));
    rmSync(dir, { recursive: true, force: true });
  });
"
```

Expected output from the smoke script: one call with `agent: "tala"`, `taskType: "consolidate"`, `triggerReason: "file-change"`, and a payload object containing `changedFiles: ["<absolute path to a.md>"]`.

## Files in Scope

| Path | Action | Purpose |
|------|--------|---------|
| `src/libs/file-watcher.ts` | CREATE | The library — types, constants, `extractAgentFromPath`, `createFileWatcher`, internal coalesce state machine, test escape hatch. ~400-500 LOC. |
| `src/libs/__tests__/file-watcher.test.ts` | CREATE | Test suite — 28 tests (27 unit + 1 integration smoke) covering path extraction, coalesce state machine, factory lifecycle, and one real-chokidar smoke. ~500-600 LOC. |
| `package.json` | MODIFY | Add `chokidar@^3.6.0` to `dependencies`. Only add if not already present at a compatible v3 version; uninstall+reinstall if v4 is already there. |

**Out of scope (do NOT touch):**

- `src/libs/task-queue.ts` — READ ONLY. Use its public `enqueueTask` API. Do not modify, extend, or wrap.
- `src/libs/agent-lifecycle.ts` — DO NOT IMPORT. Inbox count refresh is the worker's job in ASL-0006. Grep check: `grep -n "agent-lifecycle" src/libs/file-watcher.ts` must return empty.
- `src/libs/autonomous-distill.ts` — ASL-0008 wires the pipeline.
- `src/libs/reflect.ts` — ASL-0008 wires reflect.
- `src/libs/consolidate-memory.ts` — ASL-0006 worker invokes consolidate.
- `src/libs/brain/*` — schema is locked.
- `src/libs/sqlite.ts` — do not touch the DB layer. The watcher never opens a DB connection.
- `src/libs/paths.ts` — READ ONLY. Use `fromRoot` if the production wiring needs it, but the watcher library itself should accept `vaultMemoryDir` as a parameter (no hard-coded root).
- `src/services/self-learning-daemon/` — does not exist; ASL-0007 creates it. Do NOT create any daemon entry point.
- `src/tools/agent-state.ts` — does not exist; ASL-0009.
- Any hook script in `.claude/hooks/` — ASL-0008.
- Any reference, soul, knowledge, or inbox file in `vault/`.
- Any new dependency besides `chokidar@^3.6.0`. No `os`, `crypto.randomUUID` (stdlib), no `lodash.debounce`, no `p-queue`, no extras.

## Acceptance Criteria

| # | Criterion | Verification |
|---|-----------|--------------|
| 1 | `src/libs/file-watcher.ts` exists and exports `createFileWatcher`, `extractAgentFromPath`, `DEFAULT_COALESCE_WINDOW_MS`, `WATCHER_IGNORE_PATTERN`, `_createCoalesceBufferForTest`, and the four types (`FileWatcherOptions`, `FileWatcher`, `WatcherStatus`, plus any internal helper interface if used) | Code review + `grep export src/libs/file-watcher.ts` |
| 2 | `package.json` dependencies contains `"chokidar": "^3.6.0"` (or compatible v3.6.x+, NOT v4.x) | `grep -A1 '"chokidar"' package.json` |
| 3 | `chokidar.watch` is called with ALL of: `usePolling: true`, `interval: 500`, `awaitWriteFinish: { stabilityThreshold: 2000, pollInterval: 100 }`, `atomic: true`, `ignoreInitial: true`, `ignored: WATCHER_IGNORE_PATTERN` | Code review — all six keys must be present with the exact values |
| 4 | Default coalesce window is 1000ms (`DEFAULT_COALESCE_WINDOW_MS = 1000`) | Code review |
| 5 | `extractAgentFromPath` returns the correct agent for per-agent inbox paths | Tests 1, 10 |
| 6 | `extractAgentFromPath` returns `"shared"` for shared inbox paths | Test 2 |
| 7 | `extractAgentFromPath` returns `null` for raw/knowledge/archive/staged subdirs | Tests 3, 4, 5, 6 |
| 8 | `extractAgentFromPath` returns `null` for paths outside the memory root | Test 7 |
| 9 | `extractAgentFromPath` handles both forward and back slash separators | Test 8 |
| 10 | Coalesce: single event → one enqueue after window | Test 11 |
| 11 | Coalesce: 5 events for one agent within window → ONE enqueue with 5 paths | Test 12 |
| 12 | Coalesce: events for two agents within window → TWO enqueues (one per agent) | Test 13 |
| 13 | Coalesce: events outside window → multiple flushes | Test 14 |
| 14 | Coalesce: timer resets on new events (event at t=window/2 prevents flush at t=window) | Test 15 |
| 15 | Coalesce: same path repeated in window dedupes to one entry in payload | Test 16 |
| 16 | Coalesce: `unlink` events are ignored (only `add` and `change` count) | Test 18 |
| 17 | In-flight flush handling: new event during flush lands in fresh pending map | Test 20 |
| 18 | `start()` is idempotent — second call is a no-op | Test 23 |
| 19 | `stop()` clears the timer, empties pending, closes chokidar | Test 24 |
| 20 | `stop()` is idempotent — second call is a no-op | Test 25 |
| 21 | `getStatus()` reports all six fields correctly through the lifecycle | Test 26 |
| 22 | Enqueue failure for one agent does not block enqueues for other agents in the same snapshot | Test 22 |
| 23 | `WATCHER_IGNORE_PATTERN` matches `.swp`, `.tmp`, dotfiles, `~`-suffix, `raw/`, `knowledge/`, `archive/` but NOT `inbox/*.md` | Test 27 |
| 24 | Integration smoke test: real chokidar detects a new file and enqueues one task | Test 28 |
| 25 | The library does NOT import from `src/libs/agent-lifecycle.ts` | `grep -n "agent-lifecycle" src/libs/file-watcher.ts` returns empty |
| 26 | The test file does NOT import `initBrain` or anything from `src/libs/brain/` — no real DB access | `grep -n "initBrain\|libs/brain" src/libs/__tests__/file-watcher.test.ts` returns empty |
| 27 | `bun run tool brain --check` succeeds after the changes (no schema regression) | Manual smoke run |
| 28 | All unit + integration tests pass (or the documented Bun-test fallback works) | Test output |
| 29 | `package.json` has exactly one dependency added (chokidar), no unexpected additions | `git diff package.json` shows only the chokidar line |
| 30 | No real filesystem watching happens in unit tests 1-27 — only test 28 stands up chokidar | Code review of test file |
| 31 | The watcher code uses a closure-captured `state` object — no module-level mutable state | Code review: no `let` outside functions except exported constants |
| 32 | The flush function captures `state.pending` into a local snapshot BEFORE the enqueue loop AND resets `state.pending = new Map()` before the first enqueue call | Code review — line order matters |
| 33 | TypeScript compiles cleanly — no `any` types in the public API surface | `bun run tool brain --check` triggers module load |

## Defensive Reflexes — Ryan, verify before claiming done

1. **Chokidar version gate.** Before writing any code, run `grep chokidar package.json`. If it shows v4, STOP. Uninstall with `bun remove chokidar` then reinstall with `bun add chokidar@^3.6.0` and verify the `^3.` prefix. If it shows v3.6.x or higher within v3, no action needed. If it's missing entirely (confirmed state as of doc write), run `bun add chokidar@^3.6.0`. After install, re-run grep and confirm the version string starts with `^3.`. If anything about the version looks wrong, stop and re-verify before writing code.

2. **`WATCHER_IGNORE_PATTERN` regex verification.** Run the one-liner shown in Part 2 with the 7 test paths. All 7 must produce the documented output. If even one is wrong, the regex is wrong — fix the regex, do not work around it in the handler.

3. **`extractAgentFromPath` on Windows.** NTFS paths come in with back-slashes from some chokidar code paths and forward-slashes from others. The `resolve()` normalization must pass tests 1-10 on BOTH platforms. If a test passes on Linux and fails on Windows, the normalization is wrong.

4. **Snapshot-capture in `flush`.** Test #20 is the safety net. If the implementation does `for (const [agent, paths] of state.pending)` WITHOUT first capturing to a local variable, test #20 will fail in a subtle way — the new event may or may not land in the loop depending on timing. Verify the code literally matches the pattern:
   ```ts
   const snapshot = state.pending;
   state.pending = new Map();
   state.flushTimer = null;
   // ... then loop over snapshot, not state.pending
   ```
   If any of those three lines is missing or reordered, test #20 catches it only probabilistically — code review is the real gate.

5. **Single timer, not per-agent.** `scheduleFlush` resets ONE timer on the `state.flushTimer` field. Do not introduce `Map<agent, timer>` — that would produce different flush behavior and break test #13.

6. **chokidar `close()` must be awaited.** `await state.chokidarInstance.close()` inside `stop()`. Without the await, the test process may hang on exit due to leaked file handles.

7. **`ignoreInitial: true` is locked.** Do NOT set it to `false` "just in case we miss files." That's a daemon-level concern (ASL-0007 does startup reconciliation). If `ignoreInitial` is false, the watcher fires an `add` event for every existing file at startup, which produces a massive task storm on daemon boot. Never change this.

8. **Do not import from `agent-lifecycle.ts`.** Grep must return empty. If you find yourself "just wanting to refresh the inbox count after enqueue," STOP — that's the worker's job in ASL-0006. The whole point of the decoupling is failure isolation. If `refreshInboxCount` throws, the watcher should not lose events.

9. **Do not open a DB connection in this library.** `enqueueTask` does it internally; the watcher never touches `getDb`/`getRawDb`/`initBrain`. Grep: `grep -n "getDb\|getRawDb\|initBrain" src/libs/file-watcher.ts` must return empty.

10. **Test 20 — the in-flight flush test — is the hardest.** If the mock enqueue synchronously calls `handleEvent` back into the buffer, and the test expects both the original snapshot AND the new event to be flushed separately, the implementation MUST have reset `state.pending = new Map()` before entering the enqueue loop. Walk through this test manually against your implementation before running it. If you can't explain why both events end up enqueued, the implementation is wrong.

11. **Integration test #28 timing.** 500ms polling interval + 2000ms stabilityThreshold + 500ms coalesce window = 3000ms minimum. Wait 3500ms for safety. If the test flakes at 3500ms, bump to 4500ms rather than shortening stabilityThreshold (the stabilityThreshold value is locked at 2000ms per the config spec). Do not disable `usePolling` or `awaitWriteFinish` to speed up the test — that would test a configuration the daemon does not use.

12. **Do not use `atomic: false` or `awaitWriteFinish: false` anywhere.** These are the two knobs most likely to "fix flakiness" in a naive sense while breaking production behavior. If a test is flaky, the fix is longer waits, not weaker config.

13. **Do NOT create `src/services/self-learning-daemon/`.** That directory is ASL-0007's. If you catch yourself reaching for a daemon wrapper, stop — ASL-0005 is a library only.

14. **`package.json` diff sanity.** After `bun add chokidar@^3.6.0`, run `git diff package.json`. The only line that should change is adding chokidar to dependencies. If the diff shows a whole lockfile rewrite OR changes to other dependencies' versions, something is wrong with bun's install — investigate before continuing.

15. **`bun.lockb` vs `bun.lock`.** Depending on which lockfile format this project uses, the install will touch one or the other. Do not `git add` the lockfile yet — let Freddie decide at commit time. Just make sure the install succeeded.

## What NOT to Do

- **Do NOT import from `src/libs/agent-lifecycle.ts`.** Design decision locked above — the worker owns inbox count refresh, not the watcher.
- **Do NOT modify `src/libs/task-queue.ts`.** Use its public API only.
- **Do NOT modify `autonomous-distill.ts`, `reflect.ts`, `consolidate-memory.ts`, or any existing brain/ file.** Wiring is ASL-0008.
- **Do NOT create a daemon entry point.** No files under `src/services/self-learning-daemon/`. That's ASL-0007.
- **Do NOT create a worker pool.** That's ASL-0006.
- **Do NOT use `chokidar@^4.x`.** It is broken on Windows 11 per Issue #1361. Pin to `^3.6.0`.
- **Do NOT set `usePolling: false`.** Native `ReadDirectoryChangesW` drops events under Obsidian Sync burst writes. Polling is non-negotiable.
- **Do NOT set `ignoreInitial: false`.** Startup reconciliation is the daemon's job, not the watcher's. Firing add events for every existing file at startup produces a task storm.
- **Do NOT watch `raw/` under any condition.** The raw layer is append-only, immutable, and explicitly excluded by the memory-boundaries rule. The `ignored` regex enforces this.
- **Do NOT watch `knowledge/`, `archive/`, or `staged/` subdirs.** Knowledge changes are detected by workers via `isDistillStale` from ASL-0004. Archive and staged are historical/working state, not actionable signals.
- **Do NOT watch non-`.md` files.** The glob pattern restricts to `*.md`; the `handleEvent` function also double-checks the extension as belt-and-braces.
- **Do NOT use `debounce` (drop-intermediate) — use coalesce (preserve-set).** Research Q8 is explicit: coalesce preserves the set of changed files, debounce loses that information.
- **Do NOT introduce a per-agent timer.** ONE timer on `state.flushTimer`, reset on every matching event. Per-agent timers make the BRIEF pseudocode's multi-agent flush behavior impossible.
- **Do NOT add a new dependency besides chokidar.** No `p-queue`, `lodash`, `debounce-micro`, `throttle`, `eventemitter3`, etc. Everything else is stdlib + `task-queue.ts`.
- **Do NOT use `await import("chokidar")`.** Use a top-of-file static import. Dynamic import hides the version pin at the type layer.
- **Do NOT write to the real `vault/studio/memory/` tree in tests.** All tests use `os.tmpdir()` + a unique timestamp + `randomUUID()` suffix.
- **Do NOT open a brain.db connection in the watcher code.** `enqueueTask` opens its own. The watcher never calls `initBrain`, `getDb`, or `getRawDb`.
- **Do NOT open a brain.db connection in the test file.** Mock enqueue captures calls; no real enqueue ever runs. If the test imports `initBrain`, that's a coupling bug — remove it.
- **Do NOT use `setTimeout` for test timing when you can avoid it.** The coalesce tests DO need to wait for the window, but use a SHORT window (50-100ms) so tests finish in seconds, not minutes.
- **Do NOT reduce `stabilityThreshold` or `interval` to make the integration test faster.** The config values are locked by BRIEF §1. If the test is too slow, increase the wait time, not by weakening the config.
- **Do NOT flush on `stop()`.** Dropping pending events on stop is intentional — see the `stop()` docstring rationale. The daemon's startup reconciliation in ASL-0007 will catch any missed files.
- **Do NOT log at INFO level on every event.** The daemon's observability layer adds logging. The watcher logs errors only (enqueue failures to stderr) and nothing else. No `console.log("event received: ...")`.
- **Do NOT memoize `extractAgentFromPath` results.** It's already O(1) string manipulation. Memoization adds a map that would leak memory over long daemon runs.
- **Do NOT catch and swallow errors silently.** The only try/catch in production code is around `enqueueFn` — and that one logs to stderr before continuing. Do not add other try/catches that hide failures.
- **Do NOT add a CHECK constraint, schema change, or migration.** This task touches zero SQL.
- **Do NOT touch `drizzle.config.ts` or `src/libs/brain/schema.ts`.** Not this task.

## Reporting Back

When complete, Ryan should report:

1. **Chokidar install verification** — paste the output of `grep -A1 '"chokidar"' package.json`. Must show `^3.6.x` within the v3 major line. If you had to uninstall v4 first, say so.
2. **Summary of files created/modified** — line counts for `file-watcher.ts`, `file-watcher.test.ts`, and the `package.json` diff.
3. **Public exports list** — paste the names and signatures of every exported function/type/constant so McCall can verify the API surface matches the spec exactly.
4. **Regex verification output** — paste the output of the one-liner regex test from Part 2. All 7 paths must produce the documented IGNORED/WATCHED classification.
5. **Test results** — pass/fail count for the test file. If `bun test` segfaults on Windows, document the fallback (`bun run` direct execution) and paste its output instead. Call out each of the 28 tests explicitly (pass/fail).
6. **Integration test (#28) timing** — report the actual wait time that made it pass consistently. If it flakes at 3500ms but passes reliably at 4500ms, say so — that data is useful for ASL-0007.
7. **Smoke test output** — run the smoke script from Part 7 and paste the captured calls array. The output should show one enqueue call with agent="tala" and the expected payload shape.
8. **Decoupling grep output** — paste results of:
   - `grep -n "agent-lifecycle" src/libs/file-watcher.ts` (must be empty)
   - `grep -n "agent-lifecycle" src/libs/__tests__/file-watcher.test.ts` (must be empty)
   - `grep -n "initBrain\|libs/brain" src/libs/__tests__/file-watcher.test.ts` (must be empty)
   - `grep -n "getDb\|getRawDb\|initBrain" src/libs/file-watcher.ts` (must be empty)
9. **`bun run tool brain --check` output** — confirm the schema is intact and the module loads cleanly.
10. **`git status` and `git diff --stat`** — prove only the three files in scope (`file-watcher.ts`, `file-watcher.test.ts`, `package.json`, plus possibly the lockfile) were modified.
11. **Temp dir cleanup confirmation** — run `ls <os-tmpdir>` and confirm no leftover `asl-0005-*` directories after the test suite exits. If any linger, cleanup is broken.
12. **Defensive-reflex output** — for each of the 15 reflexes above, a one-line confirmation (`verified` / `found N issue, fixed by Y`).
13. **Any deviations from this spec** — if Ryan finds something wrong (chokidar v3 API drift, a glob pattern that doesn't work on Windows, a type mismatch with `task-queue.ts`), fix it AND report it so the doc can be corrected for future reference.
14. **Windows polling CPU cost observation** — during the integration test, eyeball Task Manager (or `Get-Process` CPU %) for the test process. Report whether the polling overhead is visible (>1% of a core) or not. This data will inform ASL-0007's decision on whether to tune `interval` upward.

McCall will then code-review against architectural intent (decoupling integrity, snapshot-capture pattern correctness, chokidar config lock, test coverage of the coalesce state machine including the in-flight flush case) and either approve or send back for fixes before Freddie commits.
