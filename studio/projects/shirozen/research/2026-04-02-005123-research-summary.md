---
title: "Shirozen Research Summary — Complete Findings"
type: research-summary
project: shirozen
date: 2026-04-02
iterations: 4
total_files: 20
brain_db_chunks: 113
tags: [summary, research, shirozen, complete]
---

# Shirozen Research Summary — Complete Findings

## Overview

4 iterations of research using NotebookLM + web search across GitHub repos, HackerNews, dev.to, academic papers, and official documentation. 20 research files, 113 brain.db chunks embedded.

## Research Topics & Key Findings

### 1. Communication Architecture

**Three-layer hybrid transport recommended:**

| Transport | Use Case | Mechanism |
|-----------|----------|-----------|
| **Channels** (MCP) | Real-time context push into active sessions | `notifications/claude/channel` over stdio |
| **HTTP Hooks** | Session lifecycle events (start, pre-compact, end) | POST to Shirozen service |
| **Agent SDK** (NDJSON) | Headless batch processing, background analysis | stdin/stdout streaming JSON |

**Protocol details:**
- NDJSON over stdio: 6 message types (system, assistant, user, result, stream_event, compact_boundary)
- Control requests: `can_use_tool` with request_id for permission relay
- Channels: `claude/channel` capability declaration, `<channel>` XML tag injection
- WebSocket: Bun.serve() supports HTTP + WS on same port, native pub/sub

**Key repos:** claude-code-server, claude-relay, claude-agent-server, companionfork

### 2. Storage Retention & Knowledge Decay

**FSRS (Free Spaced Repetition Scheduler):**
- npm: `ts-fsrs` — native TypeScript, ESM/CJS/UMD
- DSR model: Difficulty, Stability, Retrievability
- Forgetting curve: R = e^{-t/S}
- Maps to brain.db: FSRS columns on memories + decisions tables
- Validated by Engram (Ebbinghaus model) independently

**Bi-Temporal Tracking:**
- valid_time (when fact is true) + transaction_time (when recorded)
- SQLite: valid_from, valid_to, recorded_at, superseded_by columns
- Trigger-based automatic history recording
- Invalidation over deletion (Graphiti pattern)

**Provenance:**
- Every knowledge entry traces to source raw session(s)
- provenance_chain: raw → inbox → consolidation → knowledge
- Episodic provenance (Graphiti) validates design

**Contradiction Detection:**
- Retrieve-then-compare: embed → top-5 similar → LLM check
- Chain-of-thought prompting: ~80% F1
- Three types: value, temporal, logical
- Embedding pre-filter before LLM calls

### 3. Pattern Prediction

**Sequential Pattern Mining:**
- npm: `@smartesting/vmsp` — TypeScript VMSP algorithm
- Supports maximal/closed patterns, configurable gap, min/max length
- Better than PrefixSpan for our use case (gap support)

**Event Log Schema:**
- Event sourcing in SQLite: event_log table
- Fields: session_id, timestamp, event_type, agent, tool_name, action, metadata, sequence_num
- XES format compatibility for process mining tools

**Pattern Embedding:**
- ONNX embedding of sequence descriptions (384-dim, existing infrastructure)
- vec_patterns table for vector similarity search
- Three tiers: bag-of-events → sequence description → temporal position encoding

**Pipeline:**
```
event_log → VMSP mining → pattern_log → prefix matching → context_predictions → channel push
```

### 4. brain.db Integration

**New Tables:**
- `pattern_log` — Workflow fingerprints with FSRS decay
- `decision_feedback` — Outcome tracking for decision loop
- `context_predictions` — Prediction accuracy tracking
- `contradictions` — Flagged knowledge conflicts
- `memory_history` — Bi-temporal audit trail
- `event_log` — Tool call event sourcing
- `vec_patterns` — Pattern embeddings (384-dim)

**FSRS Columns on Existing Tables:**
- memories: fsrs_difficulty, fsrs_stability, fsrs_retrievability, fsrs_last_review, fsrs_reps, fsrs_lapses
- decisions: fsrs_difficulty, fsrs_stability

**Integration Points:**
- reflect.ts → decision feedback + pattern extraction
- consolidate-memory.ts → contradiction detection + FSRS updates
- morning.ts → predicted context from pattern_log
- startup hook → push predicted context via channels
- brain.ts --search → factor FSRS retrievability into ranking

### 5. Working Memory

**LRU warm cache between brain.db and context window:**
- In-memory Map with TTL (4 hours default)
- Max 100 entries
- 5 warming strategies: session start, topic detection, work streak, tool usage, time-based
- Semantic caching: cache query-response pairs (threshold > 0.9)

### 6. Competitor Landscape (9 projects)

| Project | Approach | Key Metric |
|---------|----------|------------|
| **Grov** | API proxy injection | 10x speedup (10min → 1-2min) |
| **subcog** | Hybrid BM25 + vector | 97% factual, 57% personal recall |
| **mnemonic** | Filesystem + bi-temporal | 74% LoCoMo (beats graph-based) |
| **Graphiti** | Temporal knowledge graph | No LLM during retrieval |
| **Engram** | BM25 + ColBERT + KG | Ebbinghaus forgetting model |
| **A-MEM** | Self-evolving ChromaDB | Auto-linking memory network |
| **claude-mem** | Hooks + compress + inject | Progressive disclosure (10x token savings) |
| **nano-brain** | BM25 + vector + reranking | Lightweight MCP server |
| **context-memory** | SQLite + FTS5 | Simple and focused |

**Shirozen's unique differentiators (no competitor has these):**
1. Pattern detection from session history
2. Predictive context loading
3. Decision feedback loop
4. Contradiction detection
5. Working memory (warm cache)

### 7. Infrastructure

**Service:** Bun.serve() on port 3456, HTTP + WebSocket dual protocol
**Daemon:** PM2 with Bun interpreter on Windows, npm scripts pattern
**Health:** GET /api/health endpoint, auto-start from startup hook

## npm Packages

| Package | Purpose | Validated |
|---------|---------|-----------|
| `ts-fsrs` | Knowledge decay (FSRS algorithm) | Yes — native TS, widely adopted |
| `@smartesting/vmsp` | Sequential pattern mining | Yes — TypeScript, gap support |
| `sqlite-vec` | Vector search for bun:sqlite | Yes — already in use |
| `pm2` | Process management | Yes — Windows stable |

## Architecture Decisions

1. Hybrid transport: Channels + Hooks + HTTP + Stdio
2. FSRS decay for knowledge confidence scoring
3. VMSP for workflow fingerprint mining
4. Bi-temporal tracking with validity windows
5. Contradiction detection via CoT prompting
6. LRU working memory as warm cache
7. Incremental updates, hash-based diffing
8. Invalidation over deletion
9. No LLM during retrieval
10. ONNX embeddings for pattern vectors

## NotebookLM Notebooks Used

| ID | Title | Queries |
|----|-------|---------|
| 4ca3782d | Claude Code Stream communication | 3 (WS protocol, persistent service, channels) |
| ad80b848 | Claude-Mem | 1 (persistent memory architecture) |
| a3de6702 | AI Agent Development | 1 (pattern detection) |
| 10f1a658 | Claude Code | 1 (session persistence) |
| fe95fa22 | Shirozen Research | Created, sources pending |
