---
source: notebooklm
notebook: ac765c3e
topic: "AI Coding Agent Persistent Memory — Cross-Platform Comparison"
date: 2026-04-03
---

# AI Coding Agent Persistent Memory — Research Summary

## Platforms Compared

| Platform | Storage | Retrieval | Consolidation | Staleness |
|----------|---------|-----------|---------------|-----------|
| **Cursor** | SQLite + local vector (multilingual-e5) | Hybrid: keyword 40% + vector 60% | Auto-summarize conversations into memos | Manual `/forget` only |
| **Windsurf** | RAG codebase index + Memories + rules files | 768-dim vector semantic search + IDE action tracking | Auto/manual save as persistent Memories | Manual — "stale memories worse than none" |
| **Codex** | In-memory ContextManager | Sequential turn/session tracking | Background CompactTask prunes/summarizes | Token-limit pruning only |
| **Antigravity** | Markdown files (`.gemini/antigravity/brain/`) | Explicit `/start` workflow reads brain dir | Timestamped RECAPs appended to daily logs | Core memory kept under 3KB, history offloaded |
| **Engram** | SQLite + FTS5 | Full-text search + session context tools | Session summary + manual consolidation | Manual delete/prune |
| **freddie-ai (ours)** | SQLite + FTS5 + sqlite-vec (384-dim) | Hybrid FTS5 + vector KNN, agent-context reranking | Automated inbox→knowledge→soul pipeline | Confidence decay (90/180 day), failed outcome 2x decay |

## Key Findings

### What we do that nobody else does (advantages)
1. **Confidence decay** — mathematical time-based staleness. Every other platform relies on manual deletion. This is our biggest differentiator.
2. **3-tier distillation pipeline** (inbox → knowledge → soul) — automated dedup, contradiction resolution, soul evolution with user approval. Others save memories flat.
3. **Per-agent memory partitioning** — each agent has its own memory namespace. Others use a single global store.

### What others do that we don't (gaps)
1. **Episodic memory / session timeline** (highest impact gap) — Antigravity tracks timestamped RECAPs of every action; Windsurf tracks real-time IDE actions. We log decisions at workflow end but not the chronological "what just happened" stream.
2. **Phase-based context loading** — Cursor-memory-bank loads only the rules needed for each phase (`/plan`, `/build`, `/reflect`), reducing token overhead ~70%. We chunk into raw phases but don't formalize phase-specific context limits.
3. **Background context compaction** — Codex continuously monitors and prunes context mid-workflow. We only compact at session boundaries.

### Closest to our architecture
- **Storage:** Cursor (SQLite + FTS5 + vector) — nearly identical stack
- **Conceptual:** Windsurf (rules=soul, memories=knowledge separation)
- **Pipeline:** Nobody matches our 3-tier automated distillation

## Highest-Impact Next Step

**Implement episodic memory / session timeline.**

We log the "why" (decisions) and "what" (knowledge) but not the "right now" (what the agent just did). A lightweight `session_events` stream would enable:
- Crash recovery (read last 10 events to resume)
- Real-time context (agent knows what command just ran)
- Token efficiency (small chronological buffer vs querying full brain.db)

This is the gap every other platform has solved in their own way — Antigravity via RECAPs, Windsurf via action tracking, Engram via timeline tools.

## Sources
- [Codex Memory System](https://deepwiki.com/openai/codex/3.7-memory-system)
- [Cursor Persistent Memory](https://forum.cursor.com/t/cursor-memory-persistent-searchable-memory-for-cursor-ai/156344)
- [Windsurf Flow Context Engine](https://markaicode.com/windsurf-flow-context-engine/)
- [Antigravity Persistent Memory](https://discuss.ai.google.dev/t/how-i-built-a-persistent-memory-system-for-antigravity-with-start-workflow-session-management-lazygravity/128640)
- [Engram — SQLite+FTS5 Memory](https://github.com/Gentleman-Programming/engram)
- [Memory Tools Comparison](https://jeradbitner.com/blog/comparing-ai-coding-memory-tools)
- NotebookLM notebook: ac765c3e
