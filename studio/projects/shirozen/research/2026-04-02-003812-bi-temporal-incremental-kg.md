---
title: "Bi-Temporal Tracking + Incremental Knowledge Graph Updates"
type: research
project: shirozen
topic: storage-retention
date: 2026-04-02
sources:
  - https://www.sqliteforum.com/p/sqlite-and-temporal-tables
  - https://www.ohnekontur.de/2024/02/19/unlocking-time-harnessing-the-power-of-temporal-tables-in-sqlite/
  - https://www.evalapply.org/posts/poor-mans-time-oriented-data-system/index.html
  - https://neo4j.com/blog/developer/graphiti-knowledge-graph-memory/
  - https://github.com/getzep/graphiti
tags: [bi-temporal, sqlite, temporal-tables, incremental, knowledge-graph, validity-windows]
---

# Bi-Temporal Tracking + Incremental Knowledge Graph Updates

## Bi-Temporal Concepts

### Two Time Dimensions
1. **Valid time** (t_valid) — When the fact is/was true in the real world
2. **Transaction time** (t_recorded) — When the fact was recorded in the database

### Why Both Matter for Shirozen
- A knowledge entry about "Suno v5.5 pipe syntax" has:
  - valid_from: 2026-03-27 (when v5.5 was released and we tested it)
  - valid_to: NULL (still true)
  - recorded_at: 2026-03-28 (when we wrote the knowledge entry)
- When v5.6 changes pipe syntax:
  - Old entry: valid_to set to 2026-04-15 (invalidated, not deleted)
  - New entry: valid_from = 2026-04-15, recorded_at = 2026-04-16

### Queries Enabled
- "What was true on March 30?" → Filter by valid_time
- "What did we know on March 30?" → Filter by recorded_at
- "What changed between then and now?" → Compare both dimensions

## SQLite Implementation

### Schema Pattern
```sql
-- Extend existing memories table
ALTER TABLE memories ADD COLUMN valid_from TEXT;      -- when fact became true
ALTER TABLE memories ADD COLUMN valid_to TEXT;         -- when fact stopped being true (NULL = still valid)
ALTER TABLE memories ADD COLUMN recorded_at TEXT;      -- when we learned it
ALTER TABLE memories ADD COLUMN superseded_by TEXT;    -- ID of entry that replaced this one

-- History table for full audit trail
CREATE TABLE memory_history (
  id INTEGER PRIMARY KEY,
  memory_id TEXT NOT NULL,
  content TEXT NOT NULL,
  valid_from TEXT NOT NULL,
  valid_to TEXT,
  recorded_at TEXT NOT NULL,
  operation TEXT NOT NULL,  -- insert | update | invalidate
  FOREIGN KEY (memory_id) REFERENCES memories(id)
);
```

### Trigger-Based Tracking
```sql
-- Auto-record history on update
CREATE TRIGGER memory_update_history
AFTER UPDATE ON memories
BEGIN
  INSERT INTO memory_history (memory_id, content, valid_from, valid_to, recorded_at, operation)
  VALUES (OLD.id, OLD.content, OLD.valid_from, OLD.valid_to, datetime('now'), 'update');
END;

-- Auto-record history on invalidation
CREATE TRIGGER memory_invalidate_history
AFTER UPDATE OF valid_to ON memories
WHEN NEW.valid_to IS NOT NULL AND OLD.valid_to IS NULL
BEGIN
  INSERT INTO memory_history (memory_id, content, valid_from, valid_to, recorded_at, operation)
  VALUES (OLD.id, OLD.content, OLD.valid_from, NEW.valid_to, datetime('now'), 'invalidate');
END;
```

### Query Patterns
```sql
-- What's currently valid
SELECT * FROM memories WHERE valid_to IS NULL;

-- What was valid on a specific date
SELECT * FROM memories 
WHERE valid_from <= '2026-03-30' 
AND (valid_to IS NULL OR valid_to > '2026-03-30');

-- What did we know at a specific time
SELECT * FROM memories 
WHERE recorded_at <= '2026-03-30';

-- Invalidated entries (potential contradictions)
SELECT m1.content AS old, m2.content AS new, m1.valid_to
FROM memories m1
JOIN memories m2 ON m1.superseded_by = m2.id
WHERE m1.valid_to IS NOT NULL
ORDER BY m1.valid_to DESC;
```

## Incremental Knowledge Graph Updates (Graphiti Pattern)

### Core Principles
1. **No batch recomputation** — New data integrates immediately
2. **Streaming ingestion** — Process episodes as they arrive
3. **Conflict resolution** — Semantic + keyword + graph search to detect conflicts
4. **Invalidation over deletion** — Old facts get validity windows, never removed
5. **Episodic provenance** — Every entity traces to source episodes

### Shirozen Incremental Update Flow
```
New knowledge arrives (from consolidation or reflection)
    ↓
1. Embed new entry
2. Search existing entries (FTS5 + vector KNN)
3. Check for contradictions (see contradiction-detection.md)
    ↓ if no contradiction:
4. Insert with valid_from = now, recorded_at = now
5. Update entity_relations if new relationships detected
    ↓ if contradiction found:
6. Log to contradictions table
7. Surface in morning briefing
8. Wait for user resolution
    ↓ when user resolves:
9. Invalidate losing entry (valid_to = now, superseded_by = winner)
10. Record in memory_history
```

### Hash-Based Change Detection (Already Exists)
brain.db's `sync_meta` table already tracks file hashes for incremental sync. Extend pattern:
- On sync, compare hash → if changed, process only the diff
- Track which chunks changed within a file
- Only re-embed changed chunks, not entire file

## Design Decisions for Shirozen

1. **Adopt bi-temporal** — Add valid_from/valid_to/recorded_at to memories and decisions
2. **Never delete** — Invalidate with valid_to timestamp + superseded_by reference
3. **History table** — Full audit trail via triggers
4. **Incremental updates** — Process new knowledge as it arrives, no batch
5. **Hash-based diffing** — Extend existing sync_meta pattern for chunk-level changes
