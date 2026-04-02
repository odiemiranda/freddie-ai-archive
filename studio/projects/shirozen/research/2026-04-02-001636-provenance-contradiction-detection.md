---
title: "Knowledge Provenance & Contradiction Detection"
type: research
project: shirozen
topic: storage-retention
date: 2026-04-02
sources:
  - https://github.com/Hawksight-AI/semantica
  - https://github.com/bolnet/agent-memory
  - https://dbis.rwth-aachen.de/dbis/index.php/2026/incremental-knowledge-graph-ingestion-with-change-detection-and-provenance-tracking/
  - https://discourse.joplinapp.org/t/joplin-atlas-ai-generated-knowledge-graph-with-concept-trails-and-contradiction-detection/49498
  - https://arxiv.org/html/2502.19023v1
tags: [provenance, contradiction, knowledge-graph, retention, sqlite]
---

# Knowledge Provenance & Contradiction Detection

## Provenance Tracking

### Core Principle
Every knowledge entry must trace back to its source sessions. "Where did this learning come from?" and "Is it still supported by evidence?" must be answerable.

### Existing Approaches

#### Semantica Framework
- Entity provenance tracking
- Algorithm provenance — tracks computation lineage
- Explainability built-in: every decision traces to source data
- **Relevant**: Decision intelligence with provenance is exactly Shirozen's model

#### Incremental Knowledge Graph Ingestion (RWTH Aachen, 2026)
- Change detection + provenance tracking for KGs
- Incremental ingestion — don't reprocess entire graph
- **Relevant**: brain.db already uses hash-based incremental sync; extend with provenance

#### RDF Provenance Challenge
- Standard RDF lacks mechanism to attach provenance
- Named graphs or reification needed to track source per triple
- **Relevant**: Our flat-file → brain.db model naturally tracks source via `sync_meta`

### Shirozen Provenance Model
```
knowledge_entry:
  id: uuid
  content: "Suno v5.5 handles pipe syntax differently..."
  source_raw_ids: [1775042718, 1775055942]  # raw session timestamps
  source_type: consolidation | reflection | manual
  created_at: timestamp
  last_validated: timestamp
  confidence: 0.0-1.0
  provenance_chain: [raw → inbox → consolidation → knowledge]
```

## Contradiction Detection

### Types of Contradictions
1. **Value conflicts** — Two entries say different things about the same fact
2. **Type conflicts** — Entry categorized differently in different contexts
3. **Temporal conflicts** — Entry was true once but isn't anymore
4. **Logical conflicts** — Two entries imply mutually exclusive conclusions

### Resolution Strategies (from literature)
- **Detection**: Multi-source conflict identification (value, type, relationship, temporal, logical)
- **Fixing**: Priority-based resolution (newer source wins? more-validated source wins?)
- **Reasoning despite inconsistency**: Continue operating even with detected contradictions

### Joplin Atlas Approach (GSoC 2026)
- AI-generated knowledge graph with concept trails
- **Contradiction detection** as explicit feature
- Concept trails = provenance paths through knowledge

### Shirozen Contradiction Model
```sql
-- New table for contradiction tracking
CREATE TABLE contradictions (
  id INTEGER PRIMARY KEY,
  entry_a_id TEXT NOT NULL,
  entry_b_id TEXT NOT NULL,
  contradiction_type TEXT NOT NULL,  -- value|temporal|logical
  description TEXT NOT NULL,
  detected_at TEXT NOT NULL,
  resolved_at TEXT,
  resolution TEXT,  -- which entry won and why
  resolved_by TEXT  -- user|auto|decay
);
```

### Detection Algorithm
1. During consolidation, before merging inbox → knowledge:
   - Embed new entry
   - Find top-5 similar existing entries (cosine similarity > 0.8)
   - For each similar pair, ask LLM: "Do these contradict? If so, how?"
   - Flag contradictions instead of silently merging
2. Periodic scan: batch-compare knowledge entries within same topic cluster
3. User-facing: surface contradictions in morning briefing

## Agent-Memory Project (SQLite reference)
- SQLite with WAL mode
- 17 columns, 8 indexes
- Sub-5ms retrieval
- **Relevant schema patterns** for brain.db extension

## Shirozen Implications

### What to build:
- `provenance_chain` field on every knowledge entry
- `contradictions` table in brain.db
- Contradiction detection during consolidation (not after)
- Morning briefing includes unresolved contradictions
- Confidence scoring that factors in provenance depth + validation recency
