---
title: "Eidetic Memory Task 4 — Search Modes (--recall, --search-raw)"
type: task
status: pending
parent: 2026-04-01-000700-eidetic-memory-raw-layer
task: 4
depends: [1, 3]
created: 2026-04-01
author: mccall
---

# Task 4 — Search Modes (--recall, --search-raw)

## Objective

Add two new search modes to `queries.ts` and wire them into the `brain.ts` CLI tool:

1. **`--recall`** — cross-store semantic search that INCLUDES raw (for conversational recall)
2. **`--search-raw`** — raw store ONLY search with optional `--skill` filter

Default search modes (`--search-refs`, `--search-all`, `--hybrid`) remain unchanged and do NOT include raw.

## What to Build

### 1. Raw search function (`src/libs/brain/queries.ts`)

Add a new hybrid search function for raw chunks, following the `hybridSearchChunks` pattern:

```typescript
export interface RawChunkResult {
  chunkId: number;
  rawId: number;
  ts: number;
  date: string;
  agents: string;
  skill: string | null;
  sectionName: string;
  content: string;
  summary: string | null;
  ftsScore: number;
  vecScore: number;
  combinedScore: number;
}

/**
 * Hybrid search on raw chunks: FTS5 keyword + sqlite-vec KNN.
 * Joins raw_chunks back to raw_files for metadata (ts, date, agents, skill, summary).
 *
 * @param opts.skill - Filter to specific skill (e.g. "/create-track")
 * @param opts.dateFrom - Filter by date >= value (YYYY-MM-DD)
 * @param opts.dateTo - Filter by date <= value (YYYY-MM-DD)
 */
export function hybridSearchRaw(
  query: string,
  queryEmbedding: number[],
  opts?: {
    skill?: string;
    dateFrom?: string;
    dateTo?: string;
    limit?: number;
    ftsWeight?: number;
    vecWeight?: number;
  },
): RawChunkResult[] { ... }
```

Implementation follows the same FTS5 + KNN merge pattern as `hybridSearchChunks()`:

1. FTS5 query against `raw_fts` joined to `raw_chunks` joined to `raw_files`.
2. KNN query against `vec_raw` joined to `raw_chunks` joined to `raw_files`.
3. Apply `--skill` filter in both SQL queries (WHERE clause on `raw_files.skill`).
4. Apply date range filters if provided.
5. Normalize scores, merge by chunk_id, sort by combined score.
6. Apply staleness degradation from `knowledgeStaleness()` using `raw_files.created_at`.

### 2. Recall search function (`src/libs/brain/queries.ts`)

Add a wrapper that calls `searchAll()` PLUS `hybridSearchRaw()`:

```typescript
export interface RecallResult extends SearchAllResult {
  raw: RawChunkResult[];
}

/**
 * Recall search — searches ALL stores including raw.
 * Used for conversational memory: "what did we discuss about X?"
 */
export function searchRecall(
  query: string,
  queryEmbedding: number[],
  opts?: {
    stores?: ("refs" | "memories" | "decisions" | "projects" | "raw")[];
    limit?: number;
    collection?: string;
    agent?: string;
    skill?: string;
  },
): RecallResult { ... }
```

Implementation:
1. Call `searchAll()` with the standard stores (refs, memories, decisions, projects).
2. Call `hybridSearchRaw()` with the query.
3. Merge results into `RecallResult`.
4. If `opts.stores` is provided, only search requested stores.

### 3. CLI wiring (`src/tools/brain.ts`)

Add two new CLI flags:

**`--recall <query>`** — calls `searchRecall()` and formats output with all stores including raw:

```
-- References --
  [0.85] references/suno/genres.md > Jazz
-- Memories --
  [0.72] memory/tala/knowledge/prompt-engineering.md (tala)
-- Raw --
  [0.91] memory/raw/1743408000.md > User Input [/create-track, 2026-04-01]
  [0.78] memory/raw/1743408000.md > Critic Feedback [/create-track, 2026-04-01]
```

**`--search-raw <query> [--skill S] [--limit N]`** — calls `hybridSearchRaw()` directly:

```
[0.91] 1743408000 (2026-04-01) /create-track > User Input
  Jazz lo-fi track with brush drums, Echo flagged frequency...
[0.78] 1743408000 (2026-04-01) /create-track > Critic Feedback
  Gemini flagged mid-frequency collision between Rhodes and...
```

### 4. Export from barrel (`src/libs/brain/index.ts`)

Add `hybridSearchRaw` and `searchRecall` to the re-exports from `./queries.js`.

### 5. Update help text (`src/tools/brain.ts`)

Add to the CLI help block at the bottom:

```
  --recall <query> [--skill S]           Cross-store search INCLUDING raw (conversational recall)
  --search-raw <query> [--skill S]       Search raw context only (eidetic lookup)
```

## Files to Modify

| File | Change |
|------|--------|
| `src/libs/brain/queries.ts` | Add `RawChunkResult`, `RecallResult`, `hybridSearchRaw()`, `searchRecall()` |
| `src/libs/brain/index.ts` | Export new functions |
| `src/tools/brain.ts` | Add `--recall` and `--search-raw` CLI handlers, update help text |

## Acceptance Criteria

1. `--search-raw "jazz"` returns results from `raw_chunks` only (no memories, refs, or decisions)
2. `--search-raw "jazz" --skill /create-track` filters results to that skill
3. `--recall "jazz"` returns results from ALL stores including raw
4. `--search-all "jazz"` does NOT return raw results (unchanged behavior)
5. `--search-refs "jazz"` does NOT return raw results (unchanged behavior)
6. Raw results display ts, date, skill, section name, and content preview
7. `hybridSearchRaw()` applies staleness degradation to older raw files
8. Both new CLI flags work with `--limit N`

## Test Criteria

1. Create 2-3 raw files with different skills and tags via `writeRaw()`.
2. Index them via `syncRaw()`.
3. Run `--search-raw` with a term that matches one file — verify correct results.
4. Run `--search-raw` with `--skill` filter — verify filtering works.
5. Run `--recall` — verify raw results appear alongside other stores.
6. Run `--search-all` — verify raw results do NOT appear.
