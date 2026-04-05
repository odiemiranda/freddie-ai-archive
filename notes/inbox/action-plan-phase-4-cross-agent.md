---
title: Phase 4 — Cross-Agent Learning
created: 2026-03-28 02:24:20 AM
tags: [action-plan, phase-4, cross-agent, shared-learning]
parent: "[[notes/inbox/action-plan-validation-and-improvements]]"
status: pending
---

# Phase 4: Cross-Agent Learning

> Learnings propagate between agents, not just within them.

## Prerequisites
- Phase 3 complete — stable consolidation with confidence tracking

## Tasks

### 4.1 — Audit shared/knowledge/
What's currently in shared? Is any agent reading it? grep for shared knowledge references in workflow traces.

### 4.2 — Push mechanism
When consolidation writes to shared/knowledge/, generate a manifest of affected topics. Next time a relevant agent runs, brain.db query surfaces the shared learning.

### 4.3 — Relevance rules
Define which shared learnings apply to which agents:
- BPM/key insights → Tala, Sol
- Vocal techniques → Sol, Rune
- Instrument frequency rules → Tala, Sol
- Lyric structure → Rune
- Critique patterns → Echo

### 4.4 — Cross-agent test
Tala discovers something about BPM handling → shared/inbox/ → consolidate → shared/knowledge/ → Sol queries brain.db → learning surfaces. Verify end-to-end.

### 4.5 — Cross-agent decision queries
New brain.ts flag: `--query-decisions "jazz fusion" --all-agents` — searches decisions across all agents, not just the current one.

## Definition of done
- [ ] Shared learnings surface in individual agent workflows
- [ ] At least 1 cross-agent learning verified end-to-end
- [ ] Cross-agent decision queries work
