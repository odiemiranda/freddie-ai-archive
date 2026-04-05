---
title: "Tech Approach: Memory Types & Semantic Search"
date: 2026-03-29
status: in-progress
source: https://github.com/EverMind-AI/EverOS
tags: [memory, brain-db, search, architecture]
---

# Tech Approach: Memory Types & Semantic Search

## Source

Cherry-picked from EverOS — a heavy memory infrastructure (MongoDB + Milvus + Elasticsearch + Redis). We take the concepts, not the stack. Our system stays on bun:sqlite + markdown files.

## Three Improvements

### 1. Memory Type Distinction

**Problem:** All memories are untyped. A session event, a general rule, and a concept profile sit side by side in the same inbox/knowledge files with no structural distinction.

**Solution:** Add `memoryType` field to memory frontmatter.

| Type | What it captures | Example |
|------|-----------------|---------|
| `episodic` | What happened in a specific session/workflow | "User rejected 3 variations before approving the shamisen track" |
| `semantic` | General knowledge, rules, patterns | "Harsh consonant clusters in soft sections sound bad when sung" |
| `entity` | About a specific concept, platform, tool | "Suno v5.5: bracket format doesn't affect output quality" |
| `decision` | Already in brain.db | "Chose pipe format over stack for readability" |

**Where it changes:**
- `reflect.ts` — Gemini reflection prompt asks for `memoryType` per learning
- Inbox file frontmatter — new field: `memoryType: episodic | semantic | entity`
- `brain.db` queries — filter by type: `--type episodic` or `--type entity`
- `consolidate-memory.ts` — Gemini consolidation prompt groups by type for cleaner knowledge files
- `memories` table in brain.db — add `memory_type` column (optional, can also rely on frontmatter tag in FTS)

**Query patterns:**
```bash
# What do we know about Suno? (entity memories)
bun src/tools/brain.ts --query "suno" --memory-type entity

# What happened last time we built a shamisen track? (episodic)
bun src/tools/brain.ts --query "shamisen" --memory-type episodic

# What rules do we have about Style blocks? (semantic)
bun src/tools/brain.ts --query "style block" --memory-type semantic
```

**Effort:** Low — one field addition to reflect prompt, frontmatter, and optional brain.db column.
**Value:** Medium — better query precision, cleaner consolidation, typed knowledge files.

### 2. Semantic / Vector Search

**Problem:** FTS5 is keyword-only. "warm low-end sound" won't match "bass-heavy mix with analog warmth" even though they describe the same thing. As memory volume grows, keyword misses will increase.

**Current retrieval:** brain.db FTS5 with BM25 ranking. Works well when memories are keyword-rich (our structured frontmatter helps). Breaks down for:
- Synonym matching (warm ≠ analog warmth)
- Conceptual similarity (lo-fi techniques ≈ tape saturation methods)
- Cross-domain connections (music production pattern ≈ software architecture pattern)

**Solution options:**

| Option | Stack | Pros | Cons |
|--------|-------|------|------|
| **A. Gemini embeddings** | Gemini embedding API + SQLite float column | Semantic matching, no new infra | API cost per embedding, latency |
| **B. Local embeddings** | bun + onnxruntime + all-MiniLM-L6-v2 | Free, fast, offline | Model download (~80MB), bun onnx support unclear |
| **C. Hybrid FTS5 + embeddings** | Either A or B + existing FTS5 | Best of both — keywords + semantics | Merge ranking complexity |

**Recommended: Option C (hybrid) with Option A (Gemini) initially.**

Implementation:
1. Add `embedding` BLOB column to `memories` and `decisions` tables
2. On sync: compute embedding via Gemini for new/changed entries
3. On query: run both FTS5 (keyword) and vector similarity (semantic), merge by weighted score
4. Fallback: if embedding API fails, FTS5 still works alone

**Embedding storage in SQLite:**
```sql
ALTER TABLE memories ADD COLUMN embedding BLOB;
-- Store as Float32Array, query with custom cosine similarity function
```

**Cost estimate:** Gemini embedding API is cheap (~$0.00001 per 1K tokens). With ~200 memory files averaging 500 tokens, full re-embed costs ~$0.001.

**Effort:** Medium — new embedding pipeline, vector storage, hybrid ranking.
**Value:** High (long-term) — becomes critical as memory volume grows past ~500 files.

### 3. Entity Tracking & Relations

**Problem:** We track isolated facts but not relationships between concepts. "Shamisen" appears in memories about fusion, jazz, Celtic, and lo-fi — but there's no graph connecting these. Queries return individual matches, not the concept map.

**Solution:** New brain.db tables for entities and relationships.

```sql
CREATE TABLE entities (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL UNIQUE,
  type TEXT NOT NULL,  -- instrument, platform, genre, technique, agent, project
  summary TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL
);

CREATE TABLE entity_relations (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  source_id INTEGER REFERENCES entities(id),
  target_id INTEGER REFERENCES entities(id),
  relation TEXT NOT NULL,  -- related_to, used_with, part_of, conflicts_with, evolved_from
  weight REAL DEFAULT 1.0,
  evidence TEXT,  -- source memory/decision that established this relation
  created_at TEXT NOT NULL
);
```

**Population:** During consolidation, Gemini extracts entities and relationships from knowledge files:
- "Shamisen jazz fusion works best at 72-85 BPM" → entity: `shamisen` (instrument), entity: `jazz fusion` (genre), relation: `shamisen → used_with → jazz fusion`
- "Suno v5.5 treats all bracket formats equivalently" → entity: `suno-v5.5` (platform), entity: `bracket format` (technique), relation: `suno-v5.5 → related_to → bracket format`

**Query patterns:**
```bash
# What do we know about shamisen? (entity + all relations)
bun src/tools/brain.ts --entity shamisen

# What's connected to jazz fusion?
bun src/tools/brain.ts --entity "jazz fusion" --relations

# What instruments work with what genres?
bun src/tools/brain.ts --entity-type instrument --relations genre
```

**Graph visualization (optional, future):** Export entity_relations as JSON → render in Obsidian Canvas or a simple D3 view.

**Effort:** Medium-High — new tables, extraction prompt, relation management.
**Value:** Medium — powerful for cross-domain queries, but FTS5 handles most current needs.

## Priority Order

| # | Feature | Effort | Value | When |
|---|---------|--------|-------|------|
| 1 | Memory type distinction | Low | Medium | Next — one field addition |
| 2 | Semantic/vector search | Medium | High (long-term) | When memory exceeds ~500 files |
| 3 | Entity tracking | Medium-High | Medium | When cross-domain queries become frequent |

## What We Explicitly Skip from EverOS

- **MongoDB / Milvus / Elasticsearch / Redis** — we stay on bun:sqlite + markdown
- **REST API service** — we use MCP
- **Docker deployment** — we run locally, zero-dependency
- **Memory sparse attention** — different problem (100M token contexts), not applicable
- **Generic Claude Code plugin** — our memory is domain-specific and richer

## Open Questions

- Should memory types be strict enum or freeform tags? Strict is cleaner for queries, freeform is more flexible.
- For vector search: Gemini embeddings vs local model? Gemini is simpler but adds API dependency. Local is free but needs onnx runtime in bun.
- Entity extraction: run during consolidation (batch) or during reflect (per-learning)? Consolidation is cleaner but delayed.
- Should entity relations have confidence scores like decisions? Would let stale relations decay.
