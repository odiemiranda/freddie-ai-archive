---
title: "Iteration 4 — Final Research Reflection & Consolidation"
type: research-reflection
project: shirozen
topic: consolidation
date: 2026-04-02
iteration: 4
---

# Iteration 4 — Final Research Reflection & Consolidation

## Gaps Resolved

### NDJSON Protocol — RESOLVED
- Complete message type catalog: system, assistant, user, result, stream_event, compact_boundary
- Control request/response format for tool permission flow
- CLI flags for streaming: --print, --output-format stream-json, --permission-prompt-tool stdio
- Hidden --sdk-url flag for WebSocket mode
- DirectConnectSessionManager for external WebSocket clients

### Contradiction Detection Prompts — RESOLVED
- Chain-of-thought prompting achieves ~80% F1
- Retrieve-then-compare pattern (embed → top-5 similar → LLM check)
- Three contradiction types: value, temporal, logical
- Embedding pre-filter before expensive LLM calls
- Optional NLI model for fast, no-LLM screening

### Bi-Temporal Tracking — RESOLVED
- Two dimensions: valid_time (real-world truth) + transaction_time (database recording)
- SQLite implementation: valid_from, valid_to, recorded_at, superseded_by columns
- Trigger-based automatic history recording
- Powerful temporal queries: "what was true on X" vs "what did we know on X"

### Incremental Knowledge Graph Updates — RESOLVED
- Graphiti pattern: streaming ingestion, no batch recomputation
- Conflict resolution via semantic + keyword + graph search
- Invalidation over deletion (validity windows)
- Hash-based change detection (already exists in sync_meta)

## Final Research Inventory — 18 Files

### Iteration 1 (7 files)
1. `websocket-communication-protocol.md` — 3 communication layers, NDJSON, channels
2. `persistent-memory-architecture.md` — claude-mem, subcog, FSRS, progressive disclosure
3. `pattern-detection-algorithms.md` — PrefixSpan, PM4Py, Cortado
4. `provenance-contradiction-detection.md` — Semantica, agent-memory, contradiction types
5. `session-persistence-context.md` — 6 memory layers, auto-compaction, hooks
6. `brain-db-integration.md` — ts-fsrs, sqlite-vec+bun, new tables
7. `iteration-1-reflection.md`

### Iteration 2 (5 files)
8. `channels-plugin-architecture.md` — Custom channel plugin, MCP notification, security
9. `working-memory-architecture.md` — Mem0, LRU cache, warming strategies
10. `event-log-schema-design.md` — Event sourcing, XES, VMSP, @smartesting/vmsp
11. `bun-server-architecture.md` — Bun.serve(), hooks, transport matrix
12. `iteration-2-reflection.md`

### Iteration 3 (4 files)
13. `competitor-landscape.md` — 9 projects, competitive positioning, differentiators
14. `temporal-event-embeddings.md` — Event2vec, RMTPP, weg2vec, ONNX approach
15. `service-management.md` — PM2, BM2, Windows daemon
16. `iteration-3-reflection.md`

### Iteration 4 (4 files)
17. `ndjson-protocol-sdk.md` — Message types, control requests, CLI flags
18. `contradiction-detection-prompts.md` — 4 approaches, prompt templates, 80% F1
19. `bi-temporal-incremental-kg.md` — SQLite temporal tables, Graphiti pattern
20. `iteration-4-reflection.md`

## Key npm Packages Identified
| Package | Purpose |
|---------|---------|
| `ts-fsrs` | Spaced repetition decay algorithm |
| `@smartesting/vmsp` | Sequential pattern mining |
| `sqlite-vec` | Vector search for bun:sqlite |
| `pm2` | Process management for daemon |

## Key Architecture Decisions (Validated by Research)

1. **Hybrid transport**: Channels (real-time push) + Hooks (lifecycle) + HTTP (API) + Stdio (batch)
2. **FSRS decay**: ts-fsrs for knowledge confidence, decision scoring
3. **VMSP pattern mining**: @smartesting/vmsp for workflow fingerprints from event_log
4. **Bi-temporal tracking**: valid_time + transaction_time on all knowledge entries
5. **Contradiction detection**: Retrieve-then-compare with CoT prompting (~80% F1)
6. **Working memory**: LRU warm cache between brain.db cold storage and context window
7. **Incremental updates**: Hash-based diffing (existing) + streaming ingestion (new)
8. **Invalidation over deletion**: Validity windows, never remove knowledge
9. **No LLM during retrieval**: Keep search fast and deterministic (Graphiti principle)
10. **ONNX embeddings for patterns**: Embed sequence descriptions with existing 384-dim model

## Research Status: COMPREHENSIVE

All four research topics now have deep, actionable coverage with concrete implementation patterns, npm packages, and validated architecture decisions. The research provides sufficient foundation to proceed to PRD.

### Suggested Next Steps
1. `/create-prd` from the BRIEF + this research corpus
2. `/architect` for detailed technical design
3. `/create-task` to slice into implementable tasks
4. Begin with Provenance Tracker (Module 1) — smallest, independently useful
