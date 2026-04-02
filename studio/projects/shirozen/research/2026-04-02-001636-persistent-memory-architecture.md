---
title: "Persistent Cross-Session Memory Architecture"
type: research
project: shirozen
topic: storage-retention
date: 2026-04-02
sources:
  - notebook: ad80b848 (Claude-Mem)
  - https://github.com/thedotmack/claude-mem
  - https://docs.claude-mem.ai/introduction
  - https://github.com/zircote/subcog
  - https://github.com/iAchilles/memento
  - https://milvus.io/blog/adding-persistent-memory-to-claude-code-with-the-lightweight-memsearch-plugin.md
tags: [memory, sqlite, fts5, vector, retention, consolidation, claude-mem]
---

# Persistent Cross-Session Memory Architecture

## Claude-Mem Architecture

### Storage Dual-Layer
1. **SQLite Database** — sessions, observations, summaries + FTS5 schema for keyword search
2. **Chroma Vector Database** — semantic retrieval via hybrid semantic + keyword search

### Memory Lifecycle
- **Capture**: 5 lifecycle hooks capture tool usage observations automatically during sessions
- **Compress**: AI (Claude Agent SDK) compresses raw session data into semantic summaries
- **Inject**: Relevant context injected into future sessions automatically

### Progressive Disclosure Pattern (3-Layer Workflow)
Key design pattern — **~10x token savings** by forcing filter-before-fetch:
1. **`search`** — Compact index: IDs + brief summaries (~50-100 tokens/result)
2. **`timeline`** — Chronological context around results
3. **`get_observations`** — Full heavy details only for specific IDs (~500-1000 tokens/result)

### Worker Service
- Bun-based HTTP API running persistently in background
- Databases and search endpoints operate as background service

### Endless Mode (Beta)
- "Biomimetic memory architecture" for extended session retention
- Designed to maintain memory coherence during long sessions

## Related Projects

### subcog (Rust)
- Hybrid search: semantic + BM25
- SQLite persistence with knowledge graph
- Captures decisions, learnings, context from sessions
- **Proactive memory surfacing** — key feature for Shirozen

### memento
- SQLite + FTS5 + sqlite-vec
- Knowledge graph with BGE-M3 embeddings
- MCP server interface

### memsearch (Lightweight)
- 4 shell hooks + background watch process
- No Node.js, MCP server, or Web UI needed
- Minimal footprint approach

### vector-memory-mcp
- sqlite-vec + sentence-transformers
- 384-dimensional vectors for semantic meaning
- Configurable memory limits

## Knowledge Retention Patterns

### Spaced Repetition (FSRS Algorithm)
- Forgetting curve: R = e^{-t/S} (R=retention, t=time, S=stability)
- Review intervals set when predicted retention drops to threshold
- SM-2 algorithm: 5-point scoring scale
- FSRS (Free Spaced Repetition Scheduler) — modern replacement for SM-2

### Staleness Detection
- Time-based decay: exponential forgetting curve
- Outcome-based: decisions that failed decay faster
- Provenance tracking: trace knowledge back to source sessions
- Contradiction detection: flag conflicting entries instead of silently merging

## Shirozen Implications

### What to adopt:
- **Progressive disclosure** — essential for token efficiency in context loading
- **Dual storage** (SQLite FTS5 + vector) — already have this in brain.db
- **Background worker service** — Shirozen runs as persistent HTTP service
- **FSRS-inspired decay** — adapt forgetting curve for knowledge confidence scoring
- **Provenance tracking** — every knowledge entry traces to source raw session

### What to improve on:
- Claude-mem consolidation is simple AI compression — Shirozen needs contradiction detection
- No existing system does proactive context prediction — this is Shirozen's differentiator
- subcog's "proactive memory surfacing" is closest but details are sparse
