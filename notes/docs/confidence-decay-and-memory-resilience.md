---
title: "Tech Approach: Confidence Decay & Memory Resilience"
date: 2026-03-29
status: planning
source: https://github.com/affaan-m/everything-claude-code
tags: [memory, brain-db, hooks, optimization]
---

# Tech Approach: Confidence Decay & Memory Resilience

## Overview

Four improvements cherry-picked from everything-claude-code, adapted for our domain-specific creative system. All build on existing infrastructure (brain.db, reflect.ts, hooks).

## 1. Confidence Decay on brain.db Decisions

### Problem
Decisions in brain.db have confidence scores but they never decay. A decision logged 3 weeks ago with 0.8 confidence carries the same weight as one logged today — even if the platform has changed or newer decisions contradict it.

### Approach
- Add a `decayedConfidence` computed field: `confidence * decayFactor(age)`
- Decay function: linear or exponential based on decision age
- Suggested: `decayedConfidence = confidence * max(0.3, 1 - (daysSince / 90))` — full confidence for ~30 days, floor at 0.3 after 90 days
- Agents query `decayedConfidence` instead of raw `confidence` in P1-002 (brain.db query step)
- Decisions reinforced by new outcomes get their timestamp refreshed (resets decay)

### Implementation
- Add `decayedConfidence()` helper to `src/libs/brain/queries.ts`
- Update brain.ts query output to show decayed values
- Update agent workflow nodes (P1-002) to reference decayed scores
- No schema changes needed — computed at query time

### Effort: Low

## 2. Contradiction Tracking in reflect.ts

### Problem
When reflect.ts saves a new learning, it doesn't check if it contradicts existing knowledge. Two contradictory learnings can coexist in knowledge/ until manual consolidation catches them.

### Approach
- After extracting learnings, reflect.ts queries brain.db for existing knowledge with overlapping tags/topics
- If overlap found: flag potential contradiction in the inbox file metadata
- `consolidate-memory.ts` already resolves contradictions — this just surfaces them earlier
- Optional: auto-trigger consolidation when a contradiction is flagged (instead of waiting for 3+ inbox threshold)

### Implementation
- Extend reflect.ts to query brain.db after learning extraction
- Add `contradicts` field to inbox file frontmatter (list of knowledge file names)
- Update consolidate-memory.ts to prioritize agents with contradiction flags
- Gemini consolidation prompt already handles contradictions — no prompt changes needed

### Effort: Low-Medium

## 3. PreCompact Hook — Save Working State

### Problem
When Claude Code compresses context in long sessions, working state is lost — current phase, approved analysis, in-progress decisions. The conversation continues but context is degraded.

### Approach
- Add a `PreCompact` hook that serializes critical working state to `out/session-state.json`
- State includes: current skill/phase, approved inputs, pending decisions, tool call count
- After compaction, the system prompt or a SessionStart-like reload can reference the saved state
- Not a full conversation restore — just the minimum context to continue the current task

### What to save
```json
{
  "timestamp": "ISO",
  "activeSkill": "/create-track",
  "phase": "P3 — Generation",
  "lockedInputs": { "type": "bgm", "genre": "jazz shamisen fusion", ... },
  "decisions": ["approved analysis at P2-002", "..."],
  "toolCallCount": 47,
  "notes": "user approved analysis, generating Style block"
}
```

### Implementation
- New hook in `.claude/hooks/` triggering on `PreCompact` event (if supported — verify Claude Code hook events)
- Hook calls a Node.js script that writes `out/session-state.json`
- Add `out/session-state.json` to context reload logic in SessionStart hook
- If PreCompact event isn't available: fall back to strategic compaction reminder (see #4)

### Effort: Medium
### Risk: PreCompact hook availability — need to verify Claude Code supports this event

## 4. Strategic Compaction Reminder

### Problem
Long sessions accumulate context without awareness. By the time compaction happens, it's forced and uncontrolled — important details may be summarized away.

### Approach
- Track tool call count in the session (via a lightweight PostToolUse hook or counter)
- At ~50 tool calls: suggest the user compact or checkpoint
- At ~80 tool calls: stronger reminder
- This gives the user control over when context is compressed, rather than hitting limits unexpectedly

### Implementation
- Counter in a PostToolUse hook (increment a file: `out/tool-call-count`)
- Check count on Stop hook — emit reminder if threshold reached
- Reset on session start

### Effort: Low

## Priority Order

| # | Feature | Effort | Value | Dependencies |
|---|---------|--------|-------|-------------|
| 1 | Confidence decay | Low | High — prevents stale decisions from influencing generation | None |
| 2 | Contradiction tracking | Low-Medium | Medium — earlier detection, already handled by consolidation | reflect.ts, brain.db |
| 3 | Strategic compaction reminder | Low | Medium — better session hygiene | Hook system |
| 4 | PreCompact state save | Medium | High — but blocked on hook availability | Hook system, verify event support |

## Open Questions

- Does Claude Code expose a `PreCompact` hook event? If not, #4 requires a different trigger (e.g., tool call threshold from #3)
- Should decayed decisions be excluded entirely below a threshold (e.g., < 0.3), or always shown with a staleness warning?
- Should contradiction detection in reflect.ts use Gemini for semantic matching, or just tag/keyword overlap from brain.db FTS5?
