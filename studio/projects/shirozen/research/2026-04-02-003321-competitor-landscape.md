---
title: "Competitor Landscape — AI Agent Memory Systems"
type: research
project: shirozen
topic: competitors
date: 2026-04-02
sources:
  - https://github.com/TonyStef/Grov
  - https://github.com/agiresearch/A-mem
  - https://github.com/zircote/subcog
  - https://github.com/zircote/mnemonic
  - https://github.com/199-biotechnologies/engram
  - https://github.com/getzep/graphiti
  - https://github.com/mem0ai/mem0
  - https://github.com/thedotmack/claude-mem
  - https://github.com/nano-step/nano-brain
  - https://github.com/ErebusEnigma/context-memory
tags: [competitors, memory, knowledge-graph, proactive, comparison]
---

# Competitor Landscape — AI Agent Memory Systems

## Tier 1: Most Relevant to Shirozen

### Grov — API Proxy Memory Injection
- **What**: Local proxy at localhost:8080, intercepts Anthropic API calls
- **How**: Captures reasoning via LLM extraction → SQLite (~/.grov/memory.db) → auto-injects into future sessions
- **Results**: 10+ minute tasks → 1-2 minutes with context injection
- **Team sync**: Optional shared memory across developers
- **Relevance to Shirozen**: Proxy approach is interesting but fragile (breaks on API changes). Shirozen's hook-based approach is more stable. The 10x speedup metric is validating.

### subcog — Rust Memory System
- **Architecture**: Access Layer (CLI/MCP/Hooks) → Service Layer (Capture/Recall/GC) → Data Layers
- **Search**: Hybrid BM25 (FTS5) + Vector (usearch HNSW, 384-dim) + Knowledge Graph (SQLite)
- **Proactive surfacing**: Key feature but details sparse
- **Performance**: 97% factual recall (LongMemEval), 57% personal context (LoCoMo)
- **Relevance to Shirozen**: Closest competitor architecturally. Our brain.db already has FTS5 + sqlite-vec. Shirozen adds pattern detection + proactive context prediction (which subcog lacks).

### mnemonic — Filesystem-Based Bi-Temporal
- **What**: Pure markdown files + git, no database
- **Key insight**: **74% LoCoMo accuracy** vs Mem0's 68.5% (graph-based)
- **Why**: "LLMs are extensively pretrained on filesystem operations, making simple tools more reliable than specialized knowledge graphs"
- **Bi-temporal tracking**: Valid time vs recorded time for every memory
- **Proactive hooks**: hookSpecificOutput.additionalContext
- **Relevance to Shirozen**: Validates our markdown-first approach. Bi-temporal tracking is something Shirozen should adopt. The "filesystem > knowledge graph" insight is surprising and worth testing.

### Graphiti/Zep — Temporal Knowledge Graph
- **What**: Python framework for temporally-aware knowledge graphs
- **Key features**:
  - Facts have validity windows (not deleted, invalidated)
  - Incremental graph construction (no batch recomputation)
  - Episodic provenance: every entity traces to source episodes
  - Hybrid: semantic + BM25 + graph traversal, NO LLM calls during retrieval
- **Performance**: 17.7% higher scores on temporal benchmark
- **Relevance to Shirozen**: Validity windows + episodic provenance is exactly our provenance tracker design. The "no LLM during retrieval" principle is good — Shirozen should keep retrieval LLM-free.

### Engram — Ebbinghaus Forgetting + Knowledge Graph
- **What**: MCP server, BM25 + ColBERT + Knowledge Graph
- **Memory model**: Mirrors brain — memories fade if not revisited, strengthen on recall
- **Entity linking**: People → places → events, asking about one surfaces related
- **Relevance to Shirozen**: Ebbinghaus model aligns with our FSRS approach. Entity linking is something Shirozen should consider (entities table already exists in brain.db).

## Tier 2: Useful Reference

### A-MEM — Self-Evolving Agentic Memory
- Writes notes on discovery → links with related → updates related → ChromaDB
- Self-evolution: memory network grows and restructures autonomously
- **Relevance**: The auto-linking pattern is interesting for Shirozen's entity_relations table

### claude-mem — Session Capture + Compression
- 5 lifecycle hooks → capture → AI compress → inject
- Progressive disclosure (3-layer) for token efficiency
- Worker service (Bun HTTP API) running in background
- **Relevance**: We studied this in iteration 1. Progressive disclosure pattern adopted.

### nano-brain — Lightweight Hybrid Search
- BM25 + vector + LLM reranking
- Local MCP server, no cloud
- **Relevance**: LLM reranking during search is interesting but adds latency

### context-memory — SQLite + FTS5 for Claude Code
- Simple and focused: searchable context storage
- **Relevance**: Validates SQLite + FTS5 as the right foundation

## Competitive Positioning

| Feature | Shirozen | subcog | Grov | mnemonic | Graphiti |
|---------|----------|--------|------|----------|---------|
| Storage | SQLite + FTS5 + vec | SQLite + FTS5 + usearch | SQLite | Markdown + git | Neo4j |
| Search | Hybrid FTS5 + vector | Hybrid BM25 + vector | Keyword | Filesystem | Semantic + BM25 + graph |
| Proactive context | **Yes (predicted)** | Partial | Injection only | Hook-based | No |
| Pattern detection | **Yes (VMSP)** | No | No | No | No |
| Temporal tracking | FSRS decay | Unknown | No | Bi-temporal | Validity windows |
| Contradiction detection | **Yes (planned)** | No | No | No | No |
| Provenance | Source raw tracking | Unknown | No | Yes | Episodic |
| Decision feedback | **Yes (planned)** | No | No | No | No |
| Working memory | **LRU warm cache** | No | No | No | No |

### Shirozen's Unique Differentiators
1. **Pattern detection from session history** — No competitor does this
2. **Predictive context loading** — No competitor anticipates what's needed
3. **Decision feedback loop** — No competitor closes the loop on outcomes
4. **Contradiction detection** — No competitor flags conflicting knowledge
5. **Working memory (warm cache)** — No competitor has a middle tier between cold storage and context

## Design Insights from Competitors

1. **Filesystem > KG for LLMs** (mnemonic): 74% vs 68.5% — consider keeping markdown as primary, brain.db as index
2. **No LLM during retrieval** (Graphiti): Keep search fast and deterministic
3. **Validity windows** (Graphiti): Don't delete, invalidate with timestamps
4. **Bi-temporal tracking** (mnemonic): Record both "when it happened" and "when we learned it"
5. **10x speedup** (Grov): Context injection genuinely transforms productivity — validates Shirozen's mission
6. **FSRS-like decay** (Engram): Ebbinghaus forgetting model independently validates our approach
