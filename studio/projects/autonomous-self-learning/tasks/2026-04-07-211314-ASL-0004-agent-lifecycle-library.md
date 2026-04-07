---
id: ASL-0004
title: Agent lifecycle library — `src/libs/agent-lifecycle.ts`
project-code: ASL
status: shipped
priority: P1
created: 2026-04-07
shipped: 2026-04-07
shipped-commit: bab968c
author: mccall
implementer: ryan
reviewer: mccall
depends-on: [ASL-0002]
blocks: [ASL-0006, ASL-0007, ASL-0008, ASL-0009]
files:
  - src/libs/agent-lifecycle.ts
  - src/libs/__tests__/agent-lifecycle.test.ts
estimated-complexity: medium (3-5 hours)
tags: [task, library, lifecycle, brain-db, asl-phase-2]
---

> **Post-implementation correction (2026-04-07):** This spec assumed
> `vault/studio/memory/shared/knowledge/` did not exist. It actually
> contains 5 files in the real vault. Tests #11 and #17 originally
> hardcoded `EMPTY_COMPOSITE = sha256(EMPTY:EMPTY)` — this would never
> match the live composite hash. Ryan caught this during implementation
> and patched both tests to compute the expected composite at test time
> using `_hashKnowledgeDirForTest` against the real shared dir. The
> algorithm structure (`sha256(agent:shared)`) is still pinned; only
> the snapshot floats. McCall code review APPROVED with 4 nits, all
> non-blocking follow-ups for ASL-0009. Shipped 2026-04-07.

# ASL-0004 — Agent lifecycle library

## TL;DR

Build `src/libs/agent-lifecycle.ts` — a pure programmatic library that wraps the `agent_lifecycle` table shipped in ASL-0002. It exposes typed upsert/read/list helpers, computes a deterministic SHA-256 hash of an agent's knowledge directory (the staleness signal that drives the daemon's distill trigger), and refreshes count fields by walking the filesystem and querying the `tasks` table directly. **No daemon code, no CLI tool, no file watcher, no integration into `autonomous-distill.ts` or `reflect.ts` — those are downstream tasks (ASL-0006, ASL-0007, ASL-0008, ASL-0009).** The library is the layer that ASL-0006 (worker pool) and ASL-0009 (`agent-state` CLI) build on top of.

## Context

ASL-0002 shipped (commit `1f5fa1a`) the `agent_lifecycle` table:

```
agent_lifecycle (
  agent                    TEXT PRIMARY KEY,
  last_saved_at            TEXT,
  last_reflected_at        TEXT,
  last_consolidated_at     TEXT,
  last_distilled_at        TEXT,
  last_distill_input_hash  TEXT,
  soul_version_hash        TEXT,
  inbox_count              INTEGER NOT NULL DEFAULT 0,
  knowledge_count          INTEGER NOT NULL DEFAULT 0,
  pending_task_count       INTEGER NOT NULL DEFAULT 0,
  distill_trigger_reason   TEXT,
  updated_at               TEXT NOT NULL
)
```

The Drizzle definition `agentLifecycle` is at `src/libs/brain/schema.ts:234-247`. The table has zero rows on a fresh DB — every consumer of this library may be looking up an agent that has never been written. The library API must handle that cleanly (return `null`, not throw).

Read these before touching anything:
- `vault/studio/projects/autonomous-self-learning/BRIEF.md` §3 — authoritative spec for the lifecycle table and the distill staleness decision logic
- `vault/studio/projects/autonomous-self-learning/tasks/2026-04-07-203621-ASL-0002-brain-db-schema.md` — the schema task doc; column names and types are locked
- `vault/studio/projects/autonomous-self-learning/research/2026-04-07-daemon-architecture-findings.md` — context on why the lifecycle layer is decoupled from the task queue layer
- `src/libs/brain/index.ts` — `initBrain()` and `getRawDb()` patterns
- `src/libs/brain/schema.ts:234-247` — `agentLifecycle` Drizzle definition (already shipped)
- `src/libs/sqlite.ts` — `getDb()` / `getRawDb()` singletons
- `src/libs/staged.ts` — operational-state lib that wraps DB queries with a typed API; same shape this library should follow
- `src/libs/distill-cache.ts` — already implements `hashDirectoryContents(dir)` over `.md` files with sorted filenames + sha256 + "empty" sentinel. **Match this pattern.** Do not invent a new one.
- `src/libs/autonomous-distill.ts` — current pipeline; this library will eventually be wired into it but **NOT in this task**.
- `src/libs/paths.ts` — `fromRoot()` for project-root-relative paths

**Decoupling rule (hard):** ASL-0003 (task queue library, parallel work) and ASL-0004 must not import each other. If `src/libs/task-queue.ts` exists when Ryan writes this code, do not import from it. The only cross-library touch is `refreshPendingTaskCount(agent)` reading the `tasks` table directly via raw SQL — that's a deliberate cross-table read, not a cross-library API call.

## Architecture Diagram

```mermaid
flowchart TD
    subgraph "Filesystem"
      KD[memory/{agent}/knowledge/*.md]
      SK[memory/shared/knowledge/*.md]
      IB[memory/{agent}/inbox/*.md]
      SOUL[.claude/souls/{agent}.md]
    end

    subgraph "brain.db"
      AL[(agent_lifecycle)]
      TS[(tasks)]
    end

    subgraph "agent-lifecycle.ts API"
      U[upsertAgentLifecycle]
      G[getAgentLifecycle]
      L[listAgentLifecycle]
      H[computeDistillInputHash]
      SH[computeCurrentSoulHash]
      ST[isDistillStale]

      RS[recordSaved]
      RR[recordReflected]
      RC[recordConsolidated]
      RD[recordDistilled]

      RIC[refreshInboxCount]
      RKC[refreshKnowledgeCount]
      RPTC[refreshPendingTaskCount]
    end

    KD --> H
    SK --> H
    SOUL --> SH
    IB --> RIC
    KD --> RKC
    TS -. raw SQL read .-> RPTC

    H --> ST
    AL -. last_distill_input_hash .-> ST

    RS --> U
    RR --> U
    RC --> U
    RD --> U
    RIC --> U
    RKC --> U
    RPTC --> U

    U <--> AL
    G <--> AL
    L <--> AL

    subgraph "Future consumers (NOT in scope)"
      D[Daemon ASL-0006/0007]
      C[agent-state CLI ASL-0009]
      AD[autonomous-distill.ts ASL-0008]
    end

    D -. uses .-> ST
    C -. uses .-> L
    AD -. uses .-> RD
```

## Goal

After this task ships:

1. `src/libs/agent-lifecycle.ts` exists and exports a typed, complete CRUD + hash + count API for the `agent_lifecycle` table.
2. The library handles all edge cases: missing agent dirs, missing soul files, empty knowledge dirs, unknown agents (no row yet) — none of these throw.
3. `computeDistillInputHash(agent)` produces a deterministic hex digest that depends ONLY on the sorted contents of the agent's knowledge dir + shared knowledge dir. Same input → same hash, every run.
4. `isDistillStale(agent)` returns a structured result the daemon can act on: stale yes/no, current hash, stored hash.
5. `recordDistilled(agent, ts, trigger)` is the complete write path for a successful distill run — it sets `last_distilled_at`, `last_distill_input_hash` (recomputed), `distill_trigger_reason`, AND `soul_version_hash` (recomputed) in one atomic upsert.
6. The library does not modify `autonomous-distill.ts`, `reflect.ts`, or any hook script. It exposes the API; downstream tasks wire it up.
7. The test suite covers upsert semantics, hash determinism, hash sentinel for missing dirs, count refresh, and `listAgentLifecycle`. Tests use a temp directory for fs-dependent fixtures and a sentinel agent prefix for DB cleanup.
8. All 20 acceptance criteria below pass.

## Detailed Specification

### Part 1 — Module structure and imports

**File:** `src/libs/agent-lifecycle.ts` (NEW)

```ts
/**
 * agent-lifecycle.ts — Per-agent lifecycle state for the ASL daemon.
 *
 * Wraps the agent_lifecycle table (shipped in ASL-0002). Provides:
 *   - Typed upsert/read/list over agent_lifecycle rows
 *   - Deterministic hash of an agent's knowledge dir (staleness signal)
 *   - Convenience updaters for the four lifecycle events (save, reflect,
 *     consolidate, distill)
 *   - Count refreshers that walk the filesystem or query the tasks table
 *
 * This library does NOT:
 *   - Trigger any work (no daemon, no scheduler, no file watcher)
 *   - Import from src/libs/task-queue.ts (the parallel ASL-0003 library)
 *   - Modify autonomous-distill.ts or reflect.ts (wiring is ASL-0008)
 *   - Run as a CLI (ASL-0009 ships the CLI on top of this)
 *
 * The decoupling rule is structural: ASL-0003 and ASL-0004 stay independent
 * libraries. The only cross-table touch here is refreshPendingTaskCount(),
 * which reads the tasks table directly via raw SQL — NOT through task-queue.ts.
 */

import { existsSync, readdirSync, readFileSync, statSync } from "fs";
import { join } from "path";
import { createHash } from "crypto";
import { eq } from "drizzle-orm";
import { getDb, getRawDb } from "./sqlite.js";
import { agentLifecycle } from "./brain/schema.js";
import { fromRoot } from "./paths.js";
```

**Constants:**

```ts
const MEMORY_ROOT = fromRoot("vault", "studio", "memory");
const SOULS_DIR = fromRoot(".claude", "souls");

/**
 * Sentinel hash returned when a knowledge dir is missing or contains zero
 * .md files. Computed once at module load. Stable across runs.
 *
 * Why a sentinel instead of null: isDistillStale() needs a hash to compare
 * against last_distill_input_hash. Returning null forces every caller to
 * special-case "no knowledge yet"; returning a sentinel lets the comparison
 * work uniformly. Empty-knowledge agents have a known-stable hash and only
 * appear stale once they accumulate at least one knowledge file.
 */
const EMPTY_KNOWLEDGE_HASH = createHash("sha256").update("empty").digest("hex");

/**
 * Sentinel hash returned when a soul file is missing. Same rationale as
 * EMPTY_KNOWLEDGE_HASH — stable, comparable, never null.
 */
const MISSING_SOUL_HASH = createHash("sha256").update("missing").digest("hex");
```

### Part 2 — Public types

```ts
/**
 * Full row from agent_lifecycle. Mirrors the Drizzle definition exactly —
 * camelCase field names, snake_case SQL columns. All "last_*_at" fields are
 * ISO-8601 strings or null. Counts are non-null integers (default 0).
 */
export interface AgentLifecycle {
  agent: string;
  lastSavedAt: string | null;
  lastReflectedAt: string | null;
  lastConsolidatedAt: string | null;
  lastDistilledAt: string | null;
  lastDistillInputHash: string | null;
  soulVersionHash: string | null;
  inboxCount: number;
  knowledgeCount: number;
  pendingTaskCount: number;
  distillTriggerReason: string | null;
  updatedAt: string;
}

/**
 * Partial set of fields callers can pass to upsertAgentLifecycle.
 * Excludes `agent` (passed separately) and `updatedAt` (always set by the
 * library to the current ISO timestamp).
 */
export type AgentLifecycleFields = Partial<Omit<AgentLifecycle, "agent" | "updatedAt">>;

/**
 * Result of isDistillStale(agent). Always carries both hashes so callers
 * (daemon, CLI, observability) can log the comparison without re-computing.
 */
export interface DistillStalenessResult {
  agent: string;
  stale: boolean;          // true iff currentHash !== storedHash
  currentHash: string;     // always populated — sentinel if no knowledge
  storedHash: string | null;  // null iff no row yet OR row has null last_distill_input_hash
}

/**
 * Optional reason string passed to recordDistilled. Free-form for now;
 * the daemon will use values like "file-change", "manual", "schedule",
 * "stale-hash". Not enum-validated at the library boundary — the column is
 * TEXT and the daemon owns the vocabulary.
 */
export type DistillTriggerReason = string;
```

### Part 3 — Hash computation helpers

**Match `distill-cache.ts:hashDirectoryContents` exactly.** That function is the canonical pattern in this codebase: sort filenames, hash filename + content for each, return hex digest, return `"empty"` sentinel for missing/empty dirs. ASL-0004 reuses the same algorithm but exports its own helper because:
1. The sentinel SHOULD be a real sha256 (`sha256("empty")`) here, not the literal string `"empty"`. The lifecycle column stores hex digests; mixing literal `"empty"` strings would break daemon comparisons.
2. The lifecycle hash composes TWO directories (agent + shared) into one final digest, while distill-cache hashes them separately and embeds them into a larger composite.

**Do NOT import `hashDirectoryContents` from `distill-cache.ts`** — it's not exported (it's a private helper) and exporting it would couple the two libraries. Reimplement here with the sentinel adjustment.

```ts
/**
 * Hash all .md files in a single directory deterministically.
 *
 * Returns the hex SHA-256 digest. Returns EMPTY_KNOWLEDGE_HASH if the dir
 * is missing or contains zero .md files.
 *
 * Algorithm (matches src/libs/distill-cache.ts:hashDirectoryContents with
 * one change — the empty sentinel is a real sha256 hex, not the literal
 * string "empty", so daemon comparisons against last_distill_input_hash
 * never see a non-hex value):
 *   1. List dir entries; filter to *.md files only
 *   2. Sort filenames lexicographically (stable across Windows/Linux)
 *   3. For each file: hash.update(filename) then hash.update(file bytes)
 *   4. Return hex digest
 *
 * Filename is included in the hash so renaming a file (without changing
 * content) produces a different digest — that IS a knowledge change.
 *
 * Subdirectories are NOT recursed. Only top-level .md files are hashed.
 * This matches the distill-cache convention and the way knowledge files
 * are organized in vault/studio/memory/{agent}/knowledge/.
 *
 * Symlinks are followed (readFileSync default behavior). Non-.md files
 * (.json memory-index, .DS_Store, etc.) are ignored.
 */
function hashKnowledgeDir(dir: string): string {
  if (!existsSync(dir)) return EMPTY_KNOWLEDGE_HASH;

  let entries: string[];
  try {
    entries = readdirSync(dir);
  } catch {
    return EMPTY_KNOWLEDGE_HASH;
  }

  const files = entries.filter(f => f.endsWith(".md")).sort();
  if (files.length === 0) return EMPTY_KNOWLEDGE_HASH;

  const h = createHash("sha256");
  for (const f of files) {
    h.update(f);                                    // filename
    try {
      h.update(readFileSync(join(dir, f)));         // content (bytes)
    } catch {
      // File disappeared between readdirSync and readFileSync (race) —
      // skip it. Determinism is best-effort under concurrent file writes;
      // the next call will pick up the consistent state.
      continue;
    }
  }
  return h.digest("hex");
}

/**
 * Compute the staleness hash for an agent.
 *
 * Composition: SHA-256 of `${agentKnowledgeHash}:${sharedKnowledgeHash}`.
 * Both inputs are individually deterministic, so the composite is too.
 *
 * Why include shared knowledge: shared/knowledge/ is loaded by every distill
 * run and influences the output. Changes there should invalidate every
 * agent's distill cache. The daemon needs the same property — a shared
 * knowledge change means every agent is potentially stale.
 *
 * Returns hex digest. Always 64 hex chars.
 */
export function computeDistillInputHash(agent: string): string {
  const agentDir = join(MEMORY_ROOT, agent, "knowledge");
  const sharedDir = join(MEMORY_ROOT, "shared", "knowledge");

  const agentHash = hashKnowledgeDir(agentDir);
  const sharedHash = hashKnowledgeDir(sharedDir);

  return createHash("sha256")
    .update(`${agentHash}:${sharedHash}`)
    .digest("hex");
}

/**
 * Compute the current soul-file content hash for an agent.
 *
 * Returns MISSING_SOUL_HASH if the soul file does not exist (e.g., a new
 * agent that hasn't been distilled yet). Always returns a stable hex digest.
 *
 * This is a snapshot hash — recordDistilled() stores it in soul_version_hash.
 * Later code (daemon, CLI, soul-drift detector) can compare the live hash
 * against the stored value to detect external soul edits between distill runs.
 */
export function computeCurrentSoulHash(agent: string): string {
  const soulPath = join(SOULS_DIR, `${agent}.md`);
  if (!existsSync(soulPath)) return MISSING_SOUL_HASH;

  try {
    const content = readFileSync(soulPath);
    return createHash("sha256").update(content).digest("hex");
  } catch {
    return MISSING_SOUL_HASH;
  }
}
```

### Part 4 — Core CRUD: upsert, get, list

**Drizzle is preferred for these — `onConflictDoUpdate` expresses the upsert semantics cleanly. Raw SQL is reserved for the cross-table read in Part 6.**

```ts
/**
 * Upsert a partial set of fields onto an agent's lifecycle row.
 *
 * Semantics:
 *   - If no row exists for `agent`, INSERT a new one with the provided
 *     fields plus library-injected `updated_at`. Counts and other NOT NULL
 *     columns rely on the schema defaults (0 for counts, null for timestamps).
 *   - If a row exists, UPDATE only the columns provided in `fields`. Untouched
 *     columns retain their current values. `updated_at` is always rewritten
 *     to the current ISO timestamp.
 *
 * The library injects `updated_at` on every call — callers MUST NOT pass it
 * in `fields`. The Partial type already excludes it.
 *
 * Returns void. Read with getAgentLifecycle if you need the post-upsert row.
 */
export function upsertAgentLifecycle(
  agent: string,
  fields: AgentLifecycleFields,
): void {
  const db = getDb();
  const now = new Date().toISOString();

  // Build the values payload — agent + updated_at + all provided fields.
  // The Drizzle schema defaults handle inbox_count/knowledge_count/
  // pending_task_count/lease_duration_ms on the INSERT path.
  const insertValues = {
    agent,
    ...fields,
    updatedAt: now,
  };

  // For the UPDATE path, only the fields the caller provided are written.
  // updated_at is always rewritten.
  const updateValues = {
    ...fields,
    updatedAt: now,
  };

  db.insert(agentLifecycle)
    .values(insertValues)
    .onConflictDoUpdate({
      target: agentLifecycle.agent,
      set: updateValues,
    })
    .run();
}

/**
 * Read an agent's lifecycle row. Returns null if no row exists.
 *
 * Does NOT auto-create a row. Callers that want "get-or-create" should call
 * upsertAgentLifecycle first with an empty fields object.
 */
export function getAgentLifecycle(agent: string): AgentLifecycle | null {
  const db = getDb();
  const rows = db.select()
    .from(agentLifecycle)
    .where(eq(agentLifecycle.agent, agent))
    .all();

  if (rows.length === 0) return null;
  return rows[0] as AgentLifecycle;
}

/**
 * Read all lifecycle rows. Used by the agent-state CLI (ASL-0009) and the
 * daemon's morning-briefing equivalent.
 *
 * Order: by agent name ascending. The CLI does its own sorting if it wants
 * different ordering.
 *
 * Returns empty array if no rows. Never throws.
 */
export function listAgentLifecycle(): AgentLifecycle[] {
  const db = getDb();
  return db.select()
    .from(agentLifecycle)
    .all() as AgentLifecycle[];
}
```

### Part 5 — Convenience updaters

These are thin wrappers around `upsertAgentLifecycle` for the four lifecycle events. They exist so callers (daemon, future hooks) can write `recordSaved("tala")` instead of remembering the field name.

```ts
/**
 * Mark that the agent just had a memory file written.
 * Sets last_saved_at. Does NOT refresh inbox_count — that's a separate
 * call (refreshInboxCount) so callers can choose whether to walk the dir.
 */
export function recordSaved(agent: string, ts?: string): void {
  upsertAgentLifecycle(agent, { lastSavedAt: ts ?? new Date().toISOString() });
}

/**
 * Mark that the agent just completed a /reflect run.
 * Sets last_reflected_at. The reflect tool will eventually call this
 * (ASL-0008), but NOT in this task — wiring is out of scope.
 */
export function recordReflected(agent: string, ts?: string): void {
  upsertAgentLifecycle(agent, { lastReflectedAt: ts ?? new Date().toISOString() });
}

/**
 * Mark that the agent just completed a consolidate-memory run.
 * Sets last_consolidated_at AND refreshes knowledge_count by walking
 * the agent's knowledge dir. Consolidation is the operation that grows
 * the knowledge dir, so this is the natural moment to update the count.
 */
export function recordConsolidated(agent: string, ts?: string): void {
  const now = ts ?? new Date().toISOString();
  const knowledgeCount = countMarkdownFiles(join(MEMORY_ROOT, agent, "knowledge"));
  upsertAgentLifecycle(agent, {
    lastConsolidatedAt: now,
    knowledgeCount,
  });
}

/**
 * Mark that the agent just completed a distill-soul run.
 *
 * Atomically sets:
 *   - last_distilled_at = ts (or now)
 *   - last_distill_input_hash = freshly computed hash of knowledge dir
 *   - distill_trigger_reason = trigger (or null)
 *   - soul_version_hash = freshly computed hash of soul file
 *
 * The two hash recomputations happen here so callers don't have to
 * orchestrate them separately. After this call returns, isDistillStale()
 * will return false (until the next knowledge change).
 *
 * NOT WIRED INTO autonomous-distill.ts in this task — that's ASL-0008.
 * The library exposes the function; integration is a separate slice.
 */
export function recordDistilled(
  agent: string,
  ts?: string,
  trigger?: DistillTriggerReason,
): void {
  const now = ts ?? new Date().toISOString();
  const inputHash = computeDistillInputHash(agent);
  const soulHash = computeCurrentSoulHash(agent);

  upsertAgentLifecycle(agent, {
    lastDistilledAt: now,
    lastDistillInputHash: inputHash,
    distillTriggerReason: trigger ?? null,
    soulVersionHash: soulHash,
  });
}
```

### Part 6 — Count refreshers

These walk the filesystem (or query the tasks table) to refresh the three count columns. They exist as standalone functions so callers can refresh on-demand without doing a full lifecycle recompute.

```ts
/**
 * Count *.md files at the top level of a directory. Returns 0 if the
 * directory is missing. Does NOT recurse into subdirectories.
 *
 * Used by both refreshInboxCount and refreshKnowledgeCount.
 */
function countMarkdownFiles(dir: string): number {
  if (!existsSync(dir)) return 0;
  try {
    return readdirSync(dir).filter(f => f.endsWith(".md")).length;
  } catch {
    return 0;
  }
}

/**
 * Walk vault/studio/memory/{agent}/inbox/ and update inbox_count.
 * Creates a lifecycle row if one doesn't exist.
 */
export function refreshInboxCount(agent: string): number {
  const count = countMarkdownFiles(join(MEMORY_ROOT, agent, "inbox"));
  upsertAgentLifecycle(agent, { inboxCount: count });
  return count;
}

/**
 * Walk vault/studio/memory/{agent}/knowledge/ and update knowledge_count.
 * Creates a lifecycle row if one doesn't exist.
 */
export function refreshKnowledgeCount(agent: string): number {
  const count = countMarkdownFiles(join(MEMORY_ROOT, agent, "knowledge"));
  upsertAgentLifecycle(agent, { knowledgeCount: count });
  return count;
}

/**
 * Query the tasks table directly to count pending tasks for an agent.
 *
 * IMPORTANT — decoupling from ASL-0003 (task queue library):
 * This function uses raw SQL against the tasks table. It does NOT import
 * from src/libs/task-queue.ts. The two libraries are deliberately
 * decoupled — composition happens at the daemon layer (ASL-0006/0007),
 * not via direct cross-library calls.
 *
 * Tradeoff: this function knows the column names of the tasks table
 * (agent, status). If ASL-0003 ever renames those columns, this query
 * breaks silently. Tests pin the column names so the breakage shows up
 * at test time, not at runtime in production.
 *
 * Counts only `status = 'pending'` rows. Claimed/completed/failed are
 * excluded — pending_task_count is "work waiting to be done", not "work
 * in progress."
 */
export function refreshPendingTaskCount(agent: string): number {
  const raw = getRawDb();
  const row = raw
    .prepare("SELECT COUNT(*) as count FROM tasks WHERE agent = ? AND status = 'pending'")
    .get(agent) as { count: number } | undefined;

  const count = row?.count ?? 0;
  upsertAgentLifecycle(agent, { pendingTaskCount: count });
  return count;
}
```

### Part 7 — Staleness check

This is the function the daemon's distill-trigger logic calls to decide whether enqueueing a distill task is worthwhile.

```ts
/**
 * Compare the agent's current knowledge-dir hash against the value stored
 * at the last distill run. If they differ (or no value is stored yet),
 * the agent is "stale" — the daemon should enqueue a distill task.
 *
 * Always returns a structured result so callers can log both hashes
 * without re-computing.
 *
 * Decision matrix:
 *   no row in agent_lifecycle      → stale=true,  storedHash=null
 *   row exists, last_distill_input_hash=null → stale=true, storedHash=null
 *   row exists, hash matches current         → stale=false
 *   row exists, hash differs from current    → stale=true
 *
 * Empty-knowledge agents (no .md files yet) return:
 *   currentHash = sha256(`${EMPTY_KNOWLEDGE_HASH}:${sharedHash}`)
 *   stale=true on first call (no stored hash), stale=false thereafter
 *
 * This is intentional — recordDistilled() will store the empty-state hash,
 * and subsequent isDistillStale calls will see a match. If shared knowledge
 * is also empty, the agent stays in a stable "nothing to distill" state.
 */
export function isDistillStale(agent: string): DistillStalenessResult {
  const currentHash = computeDistillInputHash(agent);
  const row = getAgentLifecycle(agent);
  const storedHash = row?.lastDistillInputHash ?? null;

  return {
    agent,
    stale: storedHash !== currentHash,
    currentHash,
    storedHash,
  };
}
```

### Part 8 — Test file

**File:** `src/libs/__tests__/agent-lifecycle.test.ts` (NEW)

The test file lives in `src/libs/__tests__/` (the established convention — see `staged-roundtrip.test.ts`, `judge-panel.test.ts`, `minimax-chat.test.ts` for examples).

**Test fixture strategy — read carefully:**

The library reads from `vault/studio/memory/{agent}/...` via `MEMORY_ROOT` constant. To test fs-dependent behavior without touching real vault data, the tests build a temp directory and **monkey-patch** the path via a test helper. Two viable approaches:

**Option A (preferred):** Use a sentinel agent name (e.g., `__asl_0004_test_agent__`) and write fixture files into the REAL `vault/studio/memory/__asl_0004_test_agent__/...` tree, then clean up at start AND end of every test. This is invasive but matches what ASL-0002 tests do.

**Option B (cleaner):** Refactor the library to accept an optional `memoryRoot` parameter on the hash/count functions, defaulting to `MEMORY_ROOT`. Tests pass a temp directory. **Do NOT do this for the convenience updaters or upserts** — they should always use the real MEMORY_ROOT in production. Only the pure-fs helpers (`hashKnowledgeDir`, `countMarkdownFiles`) need the override.

**Decision: use Option A for the public API tests** (convenience updaters, upsert, list) and **Option B's temp-dir override for the pure hash determinism tests**. Rationale: the public API tests need the real path resolution to verify integration; the hash tests need controlled inputs to assert determinism without depending on whatever happens to be in the real `__asl_0004_test_agent__/knowledge/` dir.

To support Option B, expose an internal helper signature:

```ts
// Exported ONLY for tests. Not part of the public API. Marked with
// @internal so consumers know not to use it.
/** @internal */
export function _hashKnowledgeDirForTest(dir: string): string {
  return hashKnowledgeDir(dir);
}
```

Same pattern as how `staged.ts` exports `parseStagedFrontmatter` for cross-module use — internal helpers can be exported with a leading underscore + `@internal` JSDoc tag.

**Test list:**

```ts
import { test, expect, beforeAll, beforeEach, afterEach, afterAll } from "bun:test";
import { mkdirSync, writeFileSync, rmSync, existsSync } from "fs";
import { join } from "path";
import { tmpdir } from "os";
import {
  upsertAgentLifecycle,
  getAgentLifecycle,
  listAgentLifecycle,
  computeDistillInputHash,
  computeCurrentSoulHash,
  isDistillStale,
  recordSaved,
  recordReflected,
  recordConsolidated,
  recordDistilled,
  refreshInboxCount,
  refreshKnowledgeCount,
  refreshPendingTaskCount,
  _hashKnowledgeDirForTest,
} from "../agent-lifecycle.js";
import { initBrain } from "../brain/index.js";
import { getRawDb } from "../sqlite.js";
import { fromRoot } from "../paths.js";

const TEST_AGENT = "__asl_0004_test_agent__";
const TEST_AGENT_2 = "__asl_0004_test_agent_2__";
const TEST_MEMORY_DIR = fromRoot("vault", "studio", "memory", TEST_AGENT);
const TEST_INBOX_DIR = join(TEST_MEMORY_DIR, "inbox");
const TEST_KNOWLEDGE_DIR = join(TEST_MEMORY_DIR, "knowledge");

let TEMP_FIXTURE_DIR: string;

beforeAll(() => {
  initBrain();
  TEMP_FIXTURE_DIR = join(tmpdir(), `asl-0004-${Date.now()}`);
  mkdirSync(TEMP_FIXTURE_DIR, { recursive: true });
});

afterAll(() => {
  // Clean up temp fixture
  if (TEMP_FIXTURE_DIR && existsSync(TEMP_FIXTURE_DIR)) {
    rmSync(TEMP_FIXTURE_DIR, { recursive: true, force: true });
  }
  // Clean up real-vault test fixtures (covered by beforeEach/afterEach
  // for individual tests too — this is a belt-and-braces sweep)
  if (existsSync(TEST_MEMORY_DIR)) {
    rmSync(TEST_MEMORY_DIR, { recursive: true, force: true });
  }
  // Clean up DB rows
  const raw = getRawDb();
  raw.exec(`DELETE FROM agent_lifecycle WHERE agent LIKE '__asl_0004_%'`);
  raw.exec(`DELETE FROM tasks WHERE agent LIKE '__asl_0004_%'`);
});

beforeEach(() => {
  // Pristine state for each test
  if (existsSync(TEST_MEMORY_DIR)) {
    rmSync(TEST_MEMORY_DIR, { recursive: true, force: true });
  }
  const raw = getRawDb();
  raw.exec(`DELETE FROM agent_lifecycle WHERE agent LIKE '__asl_0004_%'`);
  raw.exec(`DELETE FROM tasks WHERE agent LIKE '__asl_0004_%'`);
});

afterEach(() => {
  if (existsSync(TEST_MEMORY_DIR)) {
    rmSync(TEST_MEMORY_DIR, { recursive: true, force: true });
  }
  const raw = getRawDb();
  raw.exec(`DELETE FROM agent_lifecycle WHERE agent LIKE '__asl_0004_%'`);
  raw.exec(`DELETE FROM tasks WHERE agent LIKE '__asl_0004_%'`);
});
```

**Tests to write:**

| # | Test name | What it asserts |
|---|-----------|-----------------|
| 1 | `upsertAgentLifecycle creates a new row when none exists` | After upsert with `{lastSavedAt: t}`, `getAgentLifecycle` returns a row with that timestamp and counts defaulting to 0. |
| 2 | `upsertAgentLifecycle partial update only changes specified fields` | Insert with `{lastSavedAt}`, then upsert with `{lastDistilledAt}`. Verify `lastSavedAt` is preserved AND `lastDistilledAt` is set. |
| 3 | `upsertAgentLifecycle always rewrites updated_at` | Insert, capture `updatedAt`, sleep is forbidden — instead, manually set `updatedAt` to a known value via raw SQL, then re-upsert and verify the field changed. |
| 4 | `getAgentLifecycle returns null for unknown agent` | Read a never-written agent → null, no throw. |
| 5 | `listAgentLifecycle returns all rows` | Insert two distinct test agents, verify list contains both, verify ordering doesn't matter (filter by agent name in assertions). |
| 6 | `recordSaved sets only lastSavedAt` | Call recordSaved, verify lastSavedAt is set, lastReflectedAt/lastDistilledAt are null. |
| 7 | `recordConsolidated refreshes knowledgeCount` | Create knowledge dir with 3 .md files, call recordConsolidated, verify knowledgeCount=3 and lastConsolidatedAt is set. |
| 8 | `recordDistilled sets all four fields atomically` | Set up knowledge dir with one .md file. Call recordDistilled. Verify lastDistilledAt, lastDistillInputHash (matches computeDistillInputHash), distillTriggerReason, AND soulVersionHash (matches computeCurrentSoulHash) are all set in the same row. |
| 9 | `recordDistilled records null trigger when omitted` | Call without trigger arg, verify distillTriggerReason is null. |
| 10 | `computeDistillInputHash is deterministic` | Create temp dir with two .md files. Hash twice via `_hashKnowledgeDirForTest`. Equal. Then compose into the agent+shared format and verify the public function is also stable across two calls. |
| 11 | `computeDistillInputHash returns sentinel for missing dir` | Call on a nonexistent agent — must return a 64-char hex string equal to `sha256(${sha256("empty")}:${sha256("empty")})`. Hardcode the expected value in the assertion so accidental algorithm drift breaks the test. |
| 12 | `computeDistillInputHash differs when content changes` | Hash empty dir, then add a file, hash again — different. |
| 13 | `computeDistillInputHash ignores non-.md files` | Add `.md` and `.json` files. Hash. Add another `.json`. Hash again. Equal — `.json` doesn't affect the digest. |
| 14 | `computeDistillInputHash is stable across filesystem readdir order` | Create files in the order `c.md`, `a.md`, `b.md`. Hash. Manually verify the function sorts before hashing by also creating the same files in a second temp dir in a different creation order — both temp dirs hash to the same value. |
| 15 | `computeDistillInputHash differs when filename changes but content stays the same` | Renaming `a.md` to `b.md` (same content) produces a different hash (filename is part of the digest). |
| 16 | `computeCurrentSoulHash returns sentinel when soul missing` | Call for a never-existing agent, get the `sha256("missing")` hex digest. Hardcode the expected value. |
| 17 | `isDistillStale returns true when no row exists` | Fresh agent with empty knowledge → stale=true, storedHash=null, currentHash is the empty sentinel composite. |
| 18 | `isDistillStale returns false after recordDistilled with no changes` | Set up knowledge dir. Call recordDistilled. Call isDistillStale → stale=false, storedHash equals currentHash. |
| 19 | `isDistillStale returns true after knowledge changes` | Set up knowledge dir, recordDistilled, add a new .md file, call isDistillStale → stale=true. |
| 20 | `refreshInboxCount counts files and writes to DB` | Create inbox dir with 2 files, call refreshInboxCount → returns 2, getAgentLifecycle row has inboxCount=2. |
| 21 | `refreshInboxCount returns 0 for missing dir` | No fixture. Call → returns 0, row has inboxCount=0. |
| 22 | `refreshKnowledgeCount counts files and writes to DB` | Same shape as #20 but for knowledge dir. |
| 23 | `refreshPendingTaskCount queries tasks table directly` | Insert 3 task rows directly via raw SQL with `status='pending'` and one with `status='completed'`. Call refreshPendingTaskCount → returns 3, row has pendingTaskCount=3. Cleanup test rows. |
| 24 | `refreshPendingTaskCount returns 0 when no tasks` | No tasks for the test agent → returns 0. |

**Test rules (read every line):**

- **NO `sleep()` calls.** Test #3 uses raw SQL to seed a known `updatedAt` value instead of sleeping to observe a timestamp change. Same pattern for any test that wants to verify timestamps differ — set the prior value explicitly via `raw.exec("UPDATE agent_lifecycle SET updated_at = ? WHERE agent = ?", ...)`.
- **Test agent names start with `__asl_0004_`** so the `LIKE '__asl_0004_%'` cleanup in `beforeEach`/`afterEach`/`afterAll` catches every variant.
- **`beforeEach` AND `afterEach` clean up.** Belt-and-braces — if a test fails mid-run, the next test starts pristine, and `afterAll` is the final sweep.
- **Real-vault fixture writes use `vault/studio/memory/__asl_0004_test_agent__/`** — never any other path.
- **Temp dir fixtures use `os.tmpdir()` + a unique timestamp** to avoid collisions across parallel test runs.
- **The `_hashKnowledgeDirForTest` helper is the ONLY way to test the pure hash function with a custom path.** All other tests use the real `MEMORY_ROOT` via the public API.
- **Bun test segfault fallback (Windows known issue):** If `bun test src/libs/__tests__/agent-lifecycle.test.ts` segfaults, fall back to `bun run src/libs/__tests__/agent-lifecycle.test.ts` (Bun's test APIs work at runtime even when `bun test` doesn't). If that also fails, manually run a one-off script that imports the library, calls each function, and asserts against the result. Document whichever path works in the Reporting Back section. This is the same fallback ASL-0002 documented.
- **Do NOT mock `getDb()` or `getRawDb()`.** The real connection to `vault/studio/brain.db` is the test target. Cleanup queries handle isolation.

### Part 9 — Manual smoke checks

After implementation, Ryan should run these and report the output:

```bash
# 1. Run the new test file
bun test src/libs/__tests__/agent-lifecycle.test.ts

# 2. Verify brain.db still syncs cleanly (no schema regression)
bun run tool brain --check

# 3. Run a one-liner via the brain tool to verify the lifecycle row is queryable
#    after a manual upsert. Use the inline form below; do NOT add a CLI script.
bun -e "import('./src/libs/agent-lifecycle.js').then(m => { m.upsertAgentLifecycle('__asl_0004_smoke__', { lastSavedAt: new Date().toISOString() }); console.log(m.getAgentLifecycle('__asl_0004_smoke__')); m.upsertAgentLifecycle('__asl_0004_smoke__', {}); /* cleanup via raw SQL */ })"
```

(The `bun -e` line above is a smoke check, not production code — it stays in this doc and is NOT committed.)

After the smoke check, manually clean up the smoke-test row:
```bash
bun -e "import('./src/libs/sqlite.js').then(m => { m.getRawDb().exec(\"DELETE FROM agent_lifecycle WHERE agent LIKE '__asl_0004_%'\"); })"
```

## Files in Scope

| Path | Action | Purpose |
|------|--------|---------|
| `src/libs/agent-lifecycle.ts` | CREATE | The library — types, hash helpers, CRUD, convenience updaters, count refreshers. ~300-400 LOC. |
| `src/libs/__tests__/agent-lifecycle.test.ts` | CREATE | Test suite — 24 tests covering upsert semantics, hash determinism, count refreshes, and the staleness check. ~400-500 LOC. |

**Out of scope (do NOT touch):**

- `src/libs/autonomous-distill.ts` — wiring `recordDistilled()` into the pipeline is ASL-0008.
- `src/libs/reflect.ts` — wiring `recordReflected()` is ASL-0008.
- `src/libs/consolidate-memory.ts` (if it exists) — wiring `recordConsolidated()` is ASL-0008.
- `src/libs/remember.ts` or any save tool — wiring `recordSaved()` is ASL-0008.
- `src/libs/task-queue.ts` — DO NOT IMPORT FROM IT. It may not even exist yet (parallel ASL-0003). If it exists, ignore it.
- `src/libs/brain/index.ts` — schema is already shipped (ASL-0002). No changes here.
- `src/libs/brain/schema.ts` — `agentLifecycle` is already exported. No changes here.
- `src/libs/distill-cache.ts` — do NOT import `hashDirectoryContents` (it's private and the sentinel is wrong for this use case). Reimplement in agent-lifecycle.ts.
- `src/services/self-learning-daemon/` — does not exist; do not create.
- `src/tools/agent-state.ts` — does not exist; ASL-0009 owns it.
- Any hook script in `.claude/hooks/` — wiring is ASL-0008.
- Any reference, soul, or knowledge file in `vault/`.
- `package.json` — no new dependencies. Use `crypto`, `fs`, `path`, `os` from Node stdlib (already imported across the codebase) and `bun:sqlite`/`drizzle-orm` (already in use).

## Acceptance Criteria

| # | Criterion | Verification |
|---|-----------|--------------|
| 1 | `src/libs/agent-lifecycle.ts` exists and exports `AgentLifecycle`, `AgentLifecycleFields`, `DistillStalenessResult`, `DistillTriggerReason`, plus all 13 functions listed in the spec | Code review + import smoke from a one-liner |
| 2 | `upsertAgentLifecycle(agent, fields)` uses Drizzle's `onConflictDoUpdate` (NOT raw SQL) and always injects `updatedAt` automatically | Code review |
| 3 | `upsertAgentLifecycle` does a partial update — fields not in the input remain untouched | Test #2 |
| 4 | `getAgentLifecycle(agent)` returns null for unknown agents (no throw) | Test #4 |
| 5 | `listAgentLifecycle()` returns an array (possibly empty), never null, never throws | Test #5 |
| 6 | `computeDistillInputHash(agent)` returns a 64-char hex digest, deterministic across calls | Test #10 |
| 7 | `computeDistillInputHash` returns the documented sentinel composite when both knowledge dirs are missing/empty | Test #11 with hardcoded expected hex |
| 8 | `computeDistillInputHash` ignores non-.md files | Test #13 |
| 9 | `computeDistillInputHash` is stable across filesystem readdir order (sorts internally) | Test #14 |
| 10 | `computeDistillInputHash` changes when filenames change even with identical content (filename in digest) | Test #15 |
| 11 | `computeCurrentSoulHash(agent)` returns the documented `sha256("missing")` sentinel when the soul file is absent | Test #16 with hardcoded expected hex |
| 12 | `recordDistilled(agent, ts, trigger)` atomically sets all four fields (`lastDistilledAt`, `lastDistillInputHash`, `distillTriggerReason`, `soulVersionHash`) in one upsert | Test #8 |
| 13 | `isDistillStale` returns `{stale: true, storedHash: null}` for unknown agents | Test #17 |
| 14 | `isDistillStale` returns `stale: false` immediately after `recordDistilled` (no knowledge change in between) | Test #18 |
| 15 | `isDistillStale` returns `stale: true` after a `.md` file is added to the knowledge dir post-distill | Test #19 |
| 16 | `refreshPendingTaskCount` uses raw SQL against the `tasks` table (not via task-queue.ts) and counts only `status='pending'` rows | Code review + Test #23 |
| 17 | The library does NOT import from `src/libs/task-queue.ts` | `grep -n "task-queue" src/libs/agent-lifecycle.ts` returns nothing |
| 18 | The library does NOT modify `autonomous-distill.ts` or `reflect.ts` | `git diff` shows zero changes to those files |
| 19 | All 24 tests pass (or the documented Bun-test fallback works) | Test run output |
| 20 | `bun run tool brain --check` succeeds after the changes | Manual smoke run |
| 21 | No new dependencies in `package.json` | `git diff package.json` is empty |
| 22 | All test fixture writes go to the `__asl_0004_test_agent__` namespace; afterAll sweep deletes any matching DB rows | Code review of `afterAll` |
| 23 | Tests do NOT call `setTimeout`/`sleep`/`await` for timing — `updated_at` change tests use raw SQL to seed prior values | Code review |
| 24 | TypeScript compiles cleanly — no `any` types except in the documented `getRawDb().prepare(...).get() as { count: number }` cast | Compile check / `bun run tool brain --check` triggers module load |

## Defensive Reflexes — Ryan, verify before claiming done

1. **Hash sentinel hex value.** Before writing tests #11 and #16, compute the expected sentinels in a one-liner: `bun -e "import('crypto').then(c => { const a = c.createHash('sha256').update('empty').digest('hex'); console.log('a:', a, c.createHash('sha256').update(a + ':' + a).digest('hex')); console.log('m:', c.createHash('sha256').update('missing').digest('hex')); })"`. Paste the hex into the test as a literal. If the computed value differs from the literal at test time, you've drifted from the spec — fix the spec OR fix the implementation, do not just update the literal.

2. **Drizzle `onConflictDoUpdate` syntax.** Drizzle's API surface for upserts has changed across versions. Verify against the version in `package.json`. The documented signature is `db.insert(table).values(v).onConflictDoUpdate({ target: column, set: updateValues })`. If the linter complains about `target`, check whether the project uses an older Drizzle that wants `target: [column]` (array). Adjust to whatever the installed version accepts. Test #2 will catch the wrong shape at runtime.

3. **`agent_lifecycle` row count must be zero across all tests.** After the test suite finishes, `bun -e "import('./src/libs/sqlite.js').then(m => { console.log(m.getRawDb().prepare(\"SELECT COUNT(*) FROM agent_lifecycle WHERE agent LIKE '__asl_0004_%'\").get()); })"` must return `{ "COUNT(*)": 0 }`. If it doesn't, cleanup is broken — fix it before claiming done.

4. **Same for `tasks` table.** The `refreshPendingTaskCount` test (#23) inserts task rows directly. Cleanup must remove them. After the suite, `SELECT COUNT(*) FROM tasks WHERE agent LIKE '__asl_0004_%'` must return 0.

5. **Hash determinism on Windows.** `readdirSync` returns entries in NTFS order, which is NOT alphabetical. Test #14 specifically verifies the sort. If the test passes on Linux but fails on Windows, the sort isn't being applied.

6. **Filename hash inclusion.** Test #15 catches the case where someone refactors `hashKnowledgeDir` to hash only file contents (forgetting filenames). Don't drop the `h.update(f)` line — it's load-bearing.

7. **`existsSync` race conditions.** Between `existsSync(dir)` and `readdirSync(dir)`, the directory could vanish. Wrap `readdirSync` in try/catch and return the sentinel on error, matching the spec. Same for `readFileSync` inside the file loop — file may disappear between `readdirSync` and `readFileSync`.

8. **Drizzle `eq` import.** `import { eq } from "drizzle-orm"` — easy to forget. The `getAgentLifecycle` query uses it.

9. **No `as any` shortcuts in production code.** The only acceptable cast is the `getRawDb().prepare(...).get(agent) as { count: number } | undefined` in `refreshPendingTaskCount`, because `bun:sqlite`'s `.get()` returns `unknown`. Document why with a one-line comment. Everywhere else, use the typed `agentLifecycle` columns from the Drizzle schema.

10. **Don't auto-create rows on `getAgentLifecycle`.** Some libraries do "get-or-create" implicitly. This one does NOT — `getAgentLifecycle` is a pure read. The CLI and the daemon need to distinguish "agent has never been touched" from "agent has been touched but counts are zero." Test #4 enforces this.

11. **Manually verify the sentinel hex values.** Compute `sha256("empty")` and `sha256("missing")` manually before locking the test literals. If anyone ever changes the sentinel constants, the test must fail loudly — that's the whole point.

## What NOT to Do

- **Do NOT modify `src/libs/autonomous-distill.ts`.** Wiring `recordDistilled` into the pipeline is ASL-0008's job. This task only exposes the function.
- **Do NOT modify `src/libs/reflect.ts`.** Same reason — wiring `recordReflected` is ASL-0008.
- **Do NOT modify any hook script in `.claude/hooks/`.**
- **Do NOT import from `src/libs/task-queue.ts`.** It may not exist yet (parallel ASL-0003). Even if it does, do not call its API. Use raw SQL for the cross-table read.
- **Do NOT create a CLI tool.** No `src/tools/agent-state.ts`. That's ASL-0009.
- **Do NOT create a daemon entry point or any file under `src/services/self-learning-daemon/`.** That's ASL-0006/0007.
- **Do NOT add a file watcher.** That's ASL-0005.
- **Do NOT add any new dependency to `package.json`.** Use Node stdlib (`crypto`, `fs`, `path`, `os`) and the already-installed `bun:sqlite` + `drizzle-orm`.
- **Do NOT add a CHECK constraint, foreign key, or index migration.** ASL-0002 owns the schema. If a column is missing or wrong, file a TECH BLOCKER, don't patch the schema in this task.
- **Do NOT auto-recurse into knowledge subdirectories.** Top-level `*.md` only. Matches the `distill-cache.ts` convention.
- **Do NOT include inbox files in the distill input hash.** Inbox is consolidated into knowledge first; the distill hash is over the post-consolidation state.
- **Do NOT include raw files in the distill input hash.** Raw layer is excluded from self-learning by architecture.
- **Do NOT include soul files in the distill input hash.** The soul is the OUTPUT of distillation; including it in the input hash would mean every successful distill invalidates its own staleness check. (Soul gets its own hash via `soulVersionHash` for drift detection only.)
- **Do NOT use `Bun.hash`.** It's a non-cryptographic hash (XXHash variant) and produces shorter digests. The codebase uses `crypto.createHash('sha256')` everywhere — match it.
- **Do NOT export `hashKnowledgeDir` publicly.** Only the `_hashKnowledgeDirForTest` internal escape hatch is exported, and only because tests need it. Document `@internal` on it.
- **Do NOT sleep in tests.** Set timestamps explicitly via raw SQL when you need to verify they changed.
- **Do NOT touch real agent rows.** All test fixtures use the `__asl_0004_` namespace. Cleanup is mandatory.
- **Do NOT skip the cleanup queries** because "tests are isolated." The brain.db is a real, shared file; orphaned test rows pollute the agent-state CLI when ASL-0009 ships.
- **Do NOT add observability, metrics, or logging.** This is a pure data library. The daemon adds logging at its own layer.
- **Do NOT memoize the lifecycle row in module state.** Every `getAgentLifecycle` call hits the DB. Caching is a future optimization with non-trivial invalidation logic — out of scope.

## Reporting Back

When complete, Ryan should report:

1. **Summary of files created** — line counts for `agent-lifecycle.ts` and `agent-lifecycle.test.ts`. Brief description of any structural choices the spec didn't dictate.
2. **Public exports list** — paste the names and signatures of every exported function/type so McCall can verify the API surface matches the spec.
3. **Sentinel hex values** — paste the actual computed values of `sha256("empty")`, `sha256("missing")`, and the empty-state composite. Confirm they match the test literals.
4. **Test results** — pass/fail count for the new test file. If `bun test` segfaults, document the fallback path (`bun run` direct execution OR manual smoke script) and paste its output instead.
5. **`bun run tool brain --check` output** — confirm the schema is intact and the module loads cleanly.
6. **Cleanup verification** — paste:
   - `SELECT COUNT(*) FROM agent_lifecycle WHERE agent LIKE '__asl_0004_%'` (must be 0)
   - `SELECT COUNT(*) FROM tasks WHERE agent LIKE '__asl_0004_%'` (must be 0)
   - Confirmation that `vault/studio/memory/__asl_0004_test_agent__/` does not exist on disk
7. **Files-touched list** — paste `git status` and `git diff --stat` to prove only the two files in scope were modified.
8. **Any deviations from this spec** — if Ryan finds something wrong (Drizzle API drift, wrong column name, type mismatch with the schema), fix it AND report it so the doc can be corrected for future reference.
9. **Defensive-reflex output** — for each of the 11 reflexes above, a one-line confirmation ("verified" / "found N issue, fixed by Y").
10. **Decoupling sanity check** — paste the output of `grep -n "task-queue" src/libs/agent-lifecycle.ts` (should be empty) and `grep -n "from.*task-queue" src/libs/__tests__/agent-lifecycle.test.ts` (should be empty).

McCall will then code-review against architectural intent (decoupling integrity, hash algorithm correctness, upsert semantics, test coverage of the staleness state machine) and either approve or send back for fixes before Freddie commits.
