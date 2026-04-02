---
id: project-shirozen
title: "Shirozen — Cognitive Brain Engine"
status: brief
created: 2026-04-01
tags: [project, core, brain, cognition]
---

# Shirozen

**The cognitive layer for Freddie AI.** Named after Shiro from *No Game, No Life* — a quiet mastermind who sees patterns others miss — combined with Zen: calm, efficient, precise. Shirozen doesn't shout. It observes, learns, and surfaces the right context at the right time.

## The Problem

The current system is **reactive**. Agents query brain.db when asked, reflect after workflows, consolidate when inbox fills up. Knowledge exists but doesn't compound — learnings drift during consolidation, decisions don't close the feedback loop, and context loading is manual. The system remembers, but it doesn't *learn*.

### Symptoms

1. **Memory drift** — Consolidation rounds subtly shift learnings. New insights silently override older, valid ones because embedding weight favors recency. Contradictions resolve by averaging rather than surfacing.

2. **No decision feedback** — brain.db logs decisions with confidence decay, but outcomes never flow back. A decision that failed twice looks the same as one that succeeded twice — the data exists in raw transcripts but isn't mined.

3. **Manual context loading** — Agents load context on-demand via `--query` and `--search`. If the user has been working on music cards for three days, the system doesn't anticipate that — it waits to be asked.

4. **No pattern recognition** — Recurring workflows, tool sequences, and user habits exist in raw session history but aren't extracted or used. Every session starts cold.

5. **No proactive assistance** — The morning briefing is static. Suggestions are based on inbox counts and task age, not on what the user is likely to need based on recent trajectory.

## The Vision

Shirozen is a **persistent cognitive service** that sits between brain.db and the agents. It:

- **Prevents memory drift** — Every knowledge entry traces to its source raw session. Contradictions are flagged, not silently merged. Provenance is a first-class concept.

- **Closes the decision loop** — When a decision leads to a good or bad outcome, that signal flows back to update confidence. "We tried X twice and abandoned it" becomes a searchable pattern.

- **Anticipates context** — Observes conversation topics, recent file edits, time patterns, and pre-stages relevant knowledge before the user asks. Context is pushed, not pulled.

- **Learns patterns** — Extracts recurring workflow fingerprints from raw transcripts. Recognizes "this looks like the start of a create-track session" and pre-loads accordingly.

- **Streams intelligence** — An HTTP service (not MCP) that hooks, tools, and agents call. Pushes context into sessions proactively. Runs background analysis between sessions.

## Architecture (Initial Sketch)

```
┌──────────────────────────────────────────┐
│  Shirozen Engine (persistent service)    │
│                                          │
│  ┌──────────────┐  ┌─────────────────┐   │
│  │ Pattern       │  │ Contradiction   │   │
│  │ Detector      │  │ Detector        │   │
│  └──────┬───────┘  └────────┬────────┘   │
│         │                   │            │
│  ┌──────▼───────────────────▼─────────┐  │
│  │  brain.db (existing storage)       │  │
│  │  + pattern_log                     │  │
│  │  + decision_feedback               │  │
│  │  + context_predictions             │  │
│  └────────────────┬──────────────────┘   │
│                   │                      │
│  ┌────────────────▼──────────────────┐   │
│  │  Context Streamer                 │   │
│  │  push context to sessions/hooks   │   │
│  └───────────────────────────────────┘   │
│                                          │
│  ┌───────────────────────────────────┐    │
│  │  Transports                      │    │
│  │  HTTP  ·  Stdio  ·  WebSocket    │    │
│  └───────────────────────────────────┘    │
└──────────────────────────────────────────┘
         │          │            │
         ▼          ▼            ▼
┌──────────────────────────────────────────┐
│  Consumers                               │
│  HTTP  → hooks, tools, one-shot queries  │
│  Stdio → CLI pipes, scripted workflows   │
│  WS    → live sessions, real-time push   │
│                                          │
│  startup.ts · morning.ts · agents        │
│  consolidate-memory.ts · reflect.ts      │
└──────────────────────────────────────────┘
```

## Core Modules

### 1. Provenance Tracker
Every knowledge file traces to the raw session(s) that produced it. During consolidation, the system can answer: "where did this learning come from?" and "is it still supported by evidence?"

### 2. Contradiction Detector
Runs during consolidation and on-demand. When two knowledge entries conflict, it flags the contradiction with both sources rather than silently resolving. Surfaces for human decision.

### 3. Decision Feedback Loop
After a workflow completes, reflect.ts logs the outcome. Shirozen matches it against prior decisions and updates confidence scores. Failed approaches decay faster. Validated approaches strengthen.

### 4. Pattern Detector
Analyzes raw session transcripts for recurring sequences — workflow fingerprints, tool usage clusters, topic trajectories. Stores as `pattern_log` entries in brain.db.

### 5. Context Streamer
The transport layer that pushes predicted context into consumers. Uses pattern detection + recent activity + time signals to pre-stage relevant knowledge. Multi-transport:

| Transport | Use Case |
|-----------|----------|
| **HTTP** | Request/response — agents and tools query for context on demand, hooks call at session start |
| **Stdio** | Pipe integration — CLI tools and hooks stream context in/out without a running server |
| **WebSocket** | Persistent connection — real-time context push during active sessions, live updates as patterns are detected |

All three transports share the same engine and context resolution logic. The transport is just the delivery mechanism — the intelligence lives in the engine. A session can connect via WebSocket for live pushes while a hook calls HTTP for a one-shot context load.

### 6. Working Memory
A warmer layer between conversation context and brain.db cold storage. Holds recently-relevant knowledge, active patterns, and predicted needs. Decays faster than brain.db but persists across sessions within a work streak.

## Research Phase

Before building, use NotebookLM to digest raw session transcripts and extract:
- What patterns actually exist in our workflow history
- Where memory drift has occurred (compare knowledge files against their source sessions)
- Which decisions had implicit outcomes that were never captured
- What context would have been useful if pre-loaded

This research informs the design of the pattern detector and context streamer.

## Constraints

- **Builds on brain.db** — Not a replacement. Adds tables and a service layer on top of the existing SQLite + FTS5 + vec architecture.
- **Multi-transport, not MCP** — HTTP for request/response, stdio for CLI pipes, WebSocket for real-time push. No MCP protocol overhead. Same engine, multiple delivery mechanisms.
- **Agents stay the same** — Tala, Rune, Sol, Echo, Penny, McCall, Wick keep their personalities and workflows. They just get smarter inputs.
- **Developed in this repo** — `src/` for the engine, `vault/studio/projects/shirozen/` for project docs.
- **Incremental delivery** — Each module is independently useful. Provenance tracker can ship before pattern detection.

## Success Criteria

1. Zero memory drift — Knowledge files maintain provenance, contradictions are surfaced not merged
2. Decision confidence reflects reality — Outcomes feed back into scores
3. Context anticipation — Morning briefing includes predicted context based on recent work trajectory
4. Pattern recognition — System identifies recurring workflows and pre-loads relevant knowledge
5. Sub-second response — The HTTP service adds no perceptible latency to session boot or agent queries

## Open Questions

- How much raw transcript history is needed for meaningful pattern detection?
- Should the context streamer push into startup.ts hook or maintain a separate channel?
- What's the right decay rate for working memory?
- Can we use Gemini embeddings for pattern similarity, or do we need a different model for sequence matching?
- How does Shirozen interact with the existing `--recall-all` and `--search-raw` flows?
