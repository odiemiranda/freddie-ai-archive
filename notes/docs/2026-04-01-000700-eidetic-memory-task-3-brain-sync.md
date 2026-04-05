---
title: "Eidetic Memory Task 3 — brain.db Sync for Raw Files"
type: task
status: pending
parent: 2026-04-01-000700-eidetic-memory-raw-layer
task: 3
depends: [1, 2]
created: 2026-04-01
author: mccall
---

# Task 3 — brain.db Sync for Raw Files

## Objective

Add a `syncRaw()` function that indexes raw context files into brain.db (`raw_files` + `raw_chunks` + embeddings). Also add the sync guard that prevents `syncMemories()` from indexing `memory/raw/` files.

## What to Build

### 1. Sync guard in `syncMemories()` (`src/libs/brain/sync.ts`)

Modify `parseMemoryPath()` (line ~91) to return `null` when `agent === "raw"`:

```typescript
function parseMemoryPath(filePath: string): { agent: string; tier: string } | null {
  const rel = relative(MEMORY_ROOT, filePath).replace(/\\/g, "/");
  const parts = rel.split("/");
  if (parts.length < 2) return null;

  const agent = parts[0];

  // Exclude raw context files — they have their own sync path
  if (agent === "raw") return null;

  const tier = parts.length >= 3 ? parts[1] : "root";
  if (tier === "archive") return null;

  return { agent, tier };
}
```

This is a one-line addition. Without it, raw files leak into the `memories` table and pollute normal search results.

### 2. New file: `src/libs/brain/raw.ts`

This module handles raw-specific sync and insert logic:

```typescript
export interface RawSyncResult {
  filesProcessed: number;
  chunksCreated: number;
  chunksEmbedded: number;
  chunksFailed: number;
}

/**
 * Insert a single raw file into brain.db.
 * Called by syncRaw() and can also be called directly after writeRaw().
 *
 * 1. Parse frontmatter (ts, date, agents, skill, tags, summary)
 * 2. Insert into raw_files
 * 3. Chunk at ## boundaries using chunkMarkdown()
 * 4. Insert chunks into raw_chunks
 * 5. Batch embed chunks
 * 6. Insert embeddings into vec_raw
 */
export async function insertRawFile(filePath: string, opts?: { skipEmbed?: boolean }): Promise<{
  rawId: number;
  chunksCreated: number;
  chunksEmbedded: number;
}> { ... }

/**
 * Sync all raw files from vault/studio/memory/raw/ into brain.db.
 * Incremental: uses file_hash via sync_meta to skip unchanged files.
 */
export async function syncRaw(opts?: { full?: boolean }): Promise<RawSyncResult> { ... }
```

### `syncRaw()` implementation pattern

Follow the same pattern as `syncChunks()` in sync.ts:

1. Find all `.md` files in `vault/studio/memory/raw/` (not recursive, flat directory).
2. For each file:
   a. Hash the content.
   b. Check `sync_meta` for existing hash match — skip if unchanged.
   c. Parse frontmatter to extract `ts`, `date`, `agents`, `skill`, `tags`, `summary`.
   d. Upsert into `raw_files` (use `ts` as the unique identifier via source_path).
   e. Delete old chunks for this file (if re-syncing).
   f. Run `chunkMarkdown()` on the file content.
   g. Insert chunks into `raw_chunks` with `raw_id` FK.
   h. Collect chunks for batch embedding.
3. Batch embed all new chunks via `embedBatch()`.
4. Insert embeddings into `vec_raw`.
5. Update `sync_meta` entries.

### `insertRawFile()` for on-write indexing

This function is called right after `writeRaw()` to immediately index the new file without waiting for the next sync cycle. It:

1. Reads the file, parses frontmatter.
2. Checks if `raw_files` row already exists for this `ts` — if so, returns early.
3. Inserts into `raw_files`.
4. Chunks and inserts into `raw_chunks`.
5. Embeds and inserts into `vec_raw`.
6. Updates `sync_meta`.

### 3. Wire `syncRaw()` into the sync pipeline (`src/libs/brain/sync.ts`)

Add to `getEmbedStats()`:

```typescript
const rawTotal = (raw.prepare("SELECT COUNT(*) as c FROM raw_chunks").get() as any).c;
const rawEmbedded = (raw.prepare("SELECT COUNT(*) as c FROM raw_chunks WHERE embedded_at IS NOT NULL").get() as any).c;
```

Return the raw stats alongside existing stores.

### 4. Export from barrel (`src/libs/brain/index.ts`)

Add:

```typescript
export { syncRaw, insertRawFile } from "./raw.js";
```

## Files to Create

| File | Purpose |
|------|---------|
| `src/libs/brain/raw.ts` | `syncRaw()`, `insertRawFile()`, raw-specific sync logic |

## Files to Modify

| File | Change |
|------|--------|
| `src/libs/brain/sync.ts` | Add `agent === "raw"` guard in `parseMemoryPath()`, add raw to `getEmbedStats()` |
| `src/libs/brain/index.ts` | Export `syncRaw` and `insertRawFile` from `./raw.js` |

## Acceptance Criteria

1. `parseMemoryPath()` returns `null` for files under `memory/raw/`
2. `syncMemories()` does NOT index files in `memory/raw/` (verified: no `raw` agent appears in memories table)
3. `syncRaw()` indexes raw files into `raw_files` and `raw_chunks` tables
4. `syncRaw()` is incremental — re-running with no changes processes 0 files
5. `insertRawFile()` immediately indexes a single file after write
6. Chunks have embeddings in `vec_raw` after sync
7. `getEmbedStats()` includes raw store counts

## Test Criteria

1. Write a test raw file using `writeRaw()` from Task 2.
2. Run `syncRaw()` and verify `raw_files` has 1 row, `raw_chunks` has N rows (one per ## section).
3. Run `syncMemories()` and verify the raw file does NOT appear in `memories` table.
4. Run `syncRaw()` again with no changes — verify 0 files processed.
