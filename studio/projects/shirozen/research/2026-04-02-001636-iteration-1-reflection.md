---
title: "Iteration 1 — Research Reflection & Consolidation"
type: research-reflection
project: shirozen
topic: consolidation
date: 2026-04-02
iteration: 1
---

# Iteration 1 — Research Reflection & Consolidation

## What We Learned

### Communication (Strong coverage)
- Three distinct communication approaches for Shirozen: MCP Channels, Agent SDK subprocess, HTTP Hooks
- **Key insight**: Hybrid approach is best — Channels for real-time push, Hooks for lifecycle injection, Stdio for batch
- NDJSON protocol over WebSocket is well-documented, 13 control request subtypes
- Channels system is Anthropic's native solution for pushing events into running sessions
- `PreCompact` hook is critical — captures context before compression destroys it

### Storage Retention (Strong coverage)
- FSRS algorithm has native TypeScript implementation (`ts-fsrs` on npm)
- DSR model (Difficulty, Stability, Retrievability) maps perfectly to knowledge entries
- Progressive disclosure pattern from claude-mem saves ~10x tokens
- Provenance tracking: every knowledge entry traces to source raw sessions
- Contradiction detection: flag conflicts instead of silently merging

### Pattern Prediction (Moderate coverage — needs more)
- PrefixSpan is the best algorithm for sequential pattern mining
- Python implementations exist but no TypeScript (need to port or bridge)
- PM4Py for process mining, but Python-only
- **Gap**: No concrete examples of pattern detection applied to AI agent sessions
- **Gap**: No TypeScript sequential pattern mining library found

### brain.db Integration (Strong coverage)
- sqlite-vec + Bun integration already working, well-documented
- New tables needed: pattern_log, decision_feedback, context_predictions, contradictions
- FSRS columns can extend existing memories and decisions tables
- Integration points mapped to existing tools (reflect, consolidate, morning, brain)

## Key GitHub Repos Found
| Repo | Relevance |
|------|-----------|
| [claude-code-server](https://github.com/Kurogoma4D/claude-code-server) | WebSocket + Socket.io for remote CLI |
| [claude-relay](https://github.com/gvorwaller/claude-relay) | Inter-instance WS + MCP communication |
| [claude-agent-server](https://github.com/dzhng/claude-agent-server) | E2B sandbox, WS bidirectional |
| [ts-fsrs](https://github.com/open-spaced-repetition/ts-fsrs) | TypeScript FSRS algorithm |
| [PrefixSpan-py](https://github.com/chuanconggao/PrefixSpan-py) | Sequential pattern mining (Python) |
| [PM4Py](https://github.com/process-intelligence-solutions/pm4py) | Process mining (Python) |
| [subcog](https://github.com/zircote/subcog) | Proactive memory surfacing (Rust) |
| [memento](https://github.com/iAchilles/memento) | SQLite + FTS5 + sqlite-vec MCP |
| [Semantica](https://github.com/Hawksight-AI/semantica) | Provenance tracking framework |
| [agent-memory](https://github.com/bolnet/agent-memory) | SQLite WAL, sub-5ms retrieval |
| [sqlite-vec](https://github.com/asg017/sqlite-vec) | Vector search extension |
| [claude-mem](https://github.com/thedotmack/claude-mem) | Cross-session memory plugin |

## Gaps to Fill in Iteration 2
1. **TypeScript PrefixSpan** — Port or find JS sequential pattern mining
2. **Working memory architecture** — How to implement a "warm" layer between context and cold storage
3. **Context prediction accuracy** — How other systems measure prediction quality
4. **WebSocket server in Bun** — Practical implementation patterns for the Shirozen HTTP/WS service
5. **Event log schema** — How to structure session events for pattern mining input
6. **YouTube tutorials** — Initial search returned nothing; try broader terms
7. **Forum threads** — Reddit blocked; try HackerNews, dev.to, Stack Overflow
8. **subcog deep dive** — Proactive memory surfacing details
9. **Anthropic Channels docs** — Official docs on channel plugin architecture
10. **Real-time context streaming** — Server-sent events vs WebSocket for context push

## Research Files Saved
1. `2026-04-02-001636-websocket-communication-protocol.md`
2. `2026-04-02-001636-persistent-memory-architecture.md`
3. `2026-04-02-001636-pattern-detection-algorithms.md`
4. `2026-04-02-001636-provenance-contradiction-detection.md`
5. `2026-04-02-001636-session-persistence-context.md`
6. `2026-04-02-001636-brain-db-integration.md`
