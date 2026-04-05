---
title: "Eidetic Memory Task 1 — Schema + Migration"
type: task
status: pending
parent: 2026-04-01-000700-eidetic-memory-raw-layer
task: 1
depends: []
created: 2026-04-01
author: mccall
---

# Task 1 — Schema + Migration

## Objective

Add the `raw_files` and `raw_chunks` tables to brain.db, plus the `raw_ref` column on `decisions`. Create corresponding Drizzle schema definitions, FTS5 virtual table, vec0 virtual table, and sync triggers.

## What to Build

### 1. Drizzle schema (`src/libs/brain/schema.ts`)

Add two new table definitions after `projectChunks`:

```typescript
/** Raw context files — full workflow transcripts, append-only */
export const rawFiles = sqliteTable("raw_files", {
  id: integer("id").primaryKey({ autoIncrement: true }),
  ts: integer("ts").notNull().unique(),          // unix timestamp, file identity
  date: text("date").notNull(),                   // YYYY-MM-DD for display
  agents: text("agents").notNull(),               // JSON array: ["tala", "echo"]
  skill: text("skill"),                           // e.g. "/create-track"
  tags: text("tags"),                             // comma-separated
  summary: text("summary"),                       // one-line summary
  sourcePath: text("source_path").notNull().unique(),
  fileHash: text("file_hash").notNull(),
  createdAt: text("created_at").notNull(),
});

/** Raw context chunks — phase-level splits (## sections) */
export const rawChunks = sqliteTable("raw_chunks", {
  id: integer("id").primaryKey({ autoIncrement: true }),
  rawId: integer("raw_id").notNull(),             // FK to raw_files.id
  sectionName: text("section_name").notNull(),
  content: text("content").notNull(),
  lineStart: integer("line_start").notNull(),
  lineEnd: integer("line_end").notNull(),
  tokenCount: integer("token_count"),
  embedding: text("embedding"),
  fileHash: text("file_hash").notNull(),
  embedModel: text("embed_model"),
  embeddedAt: text("embedded_at"),
  createdAt: text("created_at").notNull(),
  updatedAt: text("updated_at").notNull(),
});
```

### 2. Table creation (`src/libs/brain/index.ts`)

Add raw SQL `CREATE TABLE IF NOT EXISTS` blocks in `initBrain()` for both tables, following the existing pattern (see `reference_chunks` block for structure). Add indexes:

```sql
CREATE INDEX IF NOT EXISTS idx_raw_files_ts ON raw_files(ts);
CREATE INDEX IF NOT EXISTS idx_raw_files_skill ON raw_files(skill);
CREATE INDEX IF NOT EXISTS idx_raw_files_date ON raw_files(date);
CREATE INDEX IF NOT EXISTS idx_raw_chunks_raw_id ON raw_chunks(raw_id);
CREATE INDEX IF NOT EXISTS idx_raw_chunks_source ON raw_chunks(file_hash);
```

Add migration for `decisions` table:

```sql
ALTER TABLE decisions ADD COLUMN raw_ref INTEGER
```

(Wrapped in try/catch like existing migrations on lines 165-174.)

Add vec0 virtual table:

```sql
CREATE VIRTUAL TABLE IF NOT EXISTS vec_raw USING vec0(
  chunk_id INTEGER PRIMARY KEY,
  embedding float[768]
);
```

### 3. FTS5 virtual table + triggers (`src/libs/brain/fts.ts`)

Add after the `projects_fts` block:

```typescript
// --- raw_fts ---
db.exec(`
  CREATE VIRTUAL TABLE IF NOT EXISTS raw_fts USING fts5(
    section_name, content,
    content=raw_chunks, content_rowid=id
  );
`);

createSyncTriggers(db, "raw_chunks", "raw_fts", ["section_name", "content"]);
```

Add `raw_fts` rebuild to `rebuildFts()`:

```typescript
db.exec(`INSERT INTO raw_fts(raw_fts) VALUES ('rebuild');`);
```

### 4. Update `DecisionInput` type (`src/libs/brain/queries.ts`)

Add `rawRef?: number` to the `DecisionInput` interface (line ~181). Update `logDecision()` to include `rawRef` in the insert values, mapping to the `raw_ref` column.

### 5. Re-export new schema (`src/libs/brain/index.ts`)

The barrel export `export * from "./schema.js"` already covers new schema exports. No change needed here, but verify.

## Files to Modify

| File | Change |
|------|--------|
| `src/libs/brain/schema.ts` | Add `rawFiles` and `rawChunks` table definitions |
| `src/libs/brain/index.ts` | Add CREATE TABLE, indexes, ALTER TABLE migration, vec0 table |
| `src/libs/brain/fts.ts` | Add `raw_fts` FTS5 table, triggers, rebuild entry |
| `src/libs/brain/queries.ts` | Add `rawRef` to `DecisionInput`, thread through `logDecision()` |

## Files Unchanged

All other files. No new files in this task.

## Acceptance Criteria

1. `initBrain()` succeeds without errors on a fresh database (no existing tables)
2. `initBrain()` succeeds without errors on an existing database (tables already exist, migration is idempotent)
3. `raw_files`, `raw_chunks`, `raw_fts`, `vec_raw` tables exist after init
4. `decisions` table has `raw_ref` column after init
5. FTS5 triggers fire on raw_chunks insert/update/delete (verified by inserting a test row and querying `raw_fts`)
6. `logDecision()` accepts and persists `rawRef` value
7. `logDecision()` still works without `rawRef` (backwards compatible)

## Test Criteria

Run `bun src/tools/brain.ts --stats` after changes — should not error. Manually verify tables exist via:

```bash
bun -e "
  import { initBrain } from './src/libs/brain/index.js';
  import { getRawDb } from './src/libs/sqlite.js';
  initBrain();
  const db = getRawDb();
  const tables = db.prepare(\"SELECT name FROM sqlite_master WHERE type='table' ORDER BY name\").all();
  console.log(tables.map(t => t.name).join(', '));
"
```

Expected output includes: `raw_chunks`, `raw_files`, `raw_fts` (and `vec_raw` as a virtual table).
