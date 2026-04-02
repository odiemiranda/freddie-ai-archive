---
title: "Iteration 3 — Research Reflection & Consolidation"
type: research-reflection
project: shirozen
topic: consolidation
date: 2026-04-02
iteration: 3
---

# Iteration 3 — Research Reflection & Consolidation

## Major Discoveries

### Competitor Landscape — 9 Projects Analyzed
The AI agent memory ecosystem is exploding. Mapped 9 competitor projects across 5 approaches (proxy injection, hybrid search, filesystem-based, knowledge graph, self-evolving). **Key finding: Nobody does pattern detection + predictive context loading.** This is Shirozen's unique differentiator.

### Mnemonic's Surprising Result
Filesystem-based memory (74% LoCoMo) beats graph-based (Mem0: 68.5%). The insight: "LLMs are extensively pretrained on filesystem operations, making simple tools more reliable than specialized knowledge graphs." This validates our markdown-first, brain.db-as-index approach.

### Graphiti's Temporal Model
Validity windows > deletion. When facts change, old facts are invalidated with timestamps, not removed. Full episodic provenance. No LLM calls during retrieval. These principles should shape Shirozen's design.

### Event Embedding Strategy
Three tiers identified: bag-of-events (simple), sequence description via ONNX (practical), temporal position encoding (advanced). Recommendation: Start with ONNX embedding of sequence descriptions — leverages existing infrastructure, no new models needed.

### Grov's 10x Speedup
Tasks drop from 10+ minutes to 1-2 minutes with context injection. This is the business case for Shirozen: if we can achieve even 5x improvement through proactive context, the project pays for itself.

## New Repos Found (Iteration 3)
| Repo | Relevance |
|------|-----------|
| [Grov](https://github.com/TonyStef/Grov) | API proxy memory injection, 10x speedup |
| [A-MEM](https://github.com/agiresearch/A-mem) | Self-evolving agentic memory |
| [mnemonic](https://github.com/zircote/mnemonic) | Filesystem > KG for LLMs, bi-temporal |
| [Graphiti](https://github.com/getzep/graphiti) | Temporal KG, validity windows, episodic provenance |
| [engram](https://github.com/199-biotechnologies/engram) | Ebbinghaus + BM25 + ColBERT |
| [nano-brain](https://github.com/nano-step/nano-brain) | Lightweight BM25 + vector + LLM reranking |
| [context-memory](https://github.com/ErebusEnigma/context-memory) | Simple SQLite + FTS5 |

## Remaining Gaps
1. **NDJSON protocol specification** — Need exact message format for Agent SDK communication
2. **Channel plugin testing framework** — How to test custom channels without launching full Claude session
3. **PM4Py XES export** — Bridge from event_log to visual process mining
4. **Contradiction detection prompt engineering** — What LLM prompt reliably detects semantic contradictions
5. **Real-world raw transcript analysis** — Mine actual sessions for patterns (requires separate research task)

## Research Files Saved (Iteration 3)
1. `2026-04-02-003321-competitor-landscape.md`
2. `2026-04-02-003321-temporal-event-embeddings.md`
3. `2026-04-02-003321-service-management.md`

## Cumulative Research Summary (3 Iterations)

### Total Files: 15
- Iteration 1: 7 files (protocol, memory arch, patterns, provenance, session persistence, brain.db, reflection)
- Iteration 2: 5 files (channels, working memory, event log, bun server, reflection)
- Iteration 3: 4 files (competitors, event embeddings, service management, reflection)  

### Coverage Assessment

| Topic | Status | Key Decisions Made |
|-------|--------|-------------------|
| **Communication** | **Complete** | Hybrid: Channels (real-time) + Hooks (lifecycle) + HTTP (API) |
| **Storage retention** | **Complete** | FSRS decay (ts-fsrs), provenance tracking, validity windows |
| **Pattern prediction** | **Strong** | VMSP (@smartesting/vmsp), event_log schema, ONNX embedding |
| **brain.db integration** | **Strong** | 4 new tables, FSRS columns, event sourcing pipeline |
| **Working memory** | **Complete** | LRU warm cache, 5 warming strategies, semantic caching |
| **Competitor landscape** | **Complete** | 9 projects analyzed, unique differentiators confirmed |
| **Service management** | **Complete** | PM2 + npm scripts, health check, auto-start |
| **Security model** | **Complete** | Channels 3-layer (allowlist + session opt-in + whitelist) |

### Ready for Next Phase
The research provides sufficient foundation to move to PRD. Key architectural decisions are validated by competitor analysis and proven in production by projects like Grov, subcog, and Graphiti. The remaining gaps (NDJSON protocol, channel testing, contradiction prompts) can be resolved during implementation.
