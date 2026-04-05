---
title: Phase 2 — Stress-Test the Learning Loop
created: 2026-03-28 02:24:20 AM
tags: [action-plan, phase-2, learning-loop, self-learning, validation]
parent: "[[notes/inbox/action-plan-validation-and-improvements]]"
status: pending
---

# Phase 2: Stress-Test the Learning Loop

> Generate enough real data to validate reflection, consolidation, and decision tracking.

## Context

The self-learning loop exists end-to-end but has only been run a handful of times. Current state:
- brain.db: 24 memories, 70 references, ~6 decisions
- reflect.ts: works but trace quality varies
- consolidation: works but triggered manually
- decision queries: implemented but agents don't consistently check them

We need volume to know if this compounds into real intelligence or just accumulates noise.

## What to test

### 2.1 — Run 5+ real generation sessions

Spread across different genres and styles to generate diverse learnings:

| Session | Genre/Style | Skill | Purpose |
|---------|------------|-------|---------|
| A | Lo-fi hip hop | /suno | Test common genre, should be well-supported |
| B | Vocal R&B | /minimax-music | Test vocal-forward, MiniMax strength |
| C | Dark ambient + lyrics | /lyrics + /suno | Test lyrics→suno pipeline |
| D | Fusion (jazz + electronic) | /suno | Test complex multi-genre, stress discovery |
| E | Artist reference (music card) | /musiccard + /suno | Test full card→generation pipeline |

Each session should exercise the orchestrator pattern (Phase 1 validated) and generate reflections.

### 2.2 — Audit reflection quality

After each session, check the inbox files:

**Good learning (keep):**
- Specific, actionable, not derivable from reference files
- Example: "Shamisen plucks above 4kHz compete with hi-hats — rolled-off highs needed when both present"

**Bad learning (noise):**
- Generic, obvious, already in reference files
- Example: "Style blocks should be concise" (already in craft-rules.md)

**Metric:** At least 60% of extracted learnings should be "good" (specific + actionable).

### 2.3 — Audit trace quality

Compare traces across sessions:

| Trace type | Example | Quality |
|------------|---------|---------|
| Rich | "V1: strings-forward. Gemini flagged timpani collision at 80Hz. Applied fix: moved to accent-only. User approved. V2: shamisen co-lead. Grok suggested removing timpani entirely — accepted. User preferred this groove." | High |
| Thin | "Built 3 variations. User approved all." | Low |

**Goal:** Identify minimum trace requirements that produce useful reflections.

### 2.4 — Design trace template

Based on 2.3 findings, create a standard trace structure:

```
## Request
[what the user asked for]

## Key Decisions
- [decision 1]: [choice made] because [reason]. Alternatives: [what was rejected]
- [decision 2]: ...

## Critic Feedback
- Gemini: [key flags, what was accepted/rejected]
- MiniMax: [key flags]
- Grok: [key flags]

## User Reactions
- V1: [approved/adjusted/rejected] — [user's feedback if any]
- V2: ...

## Surprises
- [anything unexpected about platform behavior, user preference, or output quality]
```

Update reflect.ts or workflow nodes to enforce this structure.

### 2.5 — Run consolidation

After 3+ inbox items per agent, run:
```bash
bun src/tools/consolidate-memory.ts --auto
```

### 2.6 — Audit consolidated knowledge

Check each output knowledge file:
- [ ] No contradictions with existing knowledge
- [ ] No duplication of reference file content
- [ ] Actionable and specific
- [ ] Would actually change behavior if loaded during generation

### 2.7 — Verify decision queries

During sessions A-E, check that node 002 actually queries brain.db decisions:
- Does `brain.ts --recall --agent tala` return relevant past decisions?
- Do those decisions influence the current generation?
- If a past decision had a negative outcome, does the agent avoid repeating it?

### 2.8 — Test outcome tracking

After session A:
1. Find a decision logged during the session
2. Update it with an outcome: `brain.ts --outcome <id> --result "success — user loved the shamisen texture"`
3. In session B, verify the outcome appears when querying related decisions

## Definition of done

- [ ] 5+ real generation sessions completed
- [ ] 20+ decisions in brain.db with meaningful context
- [ ] 60%+ of reflections produce actionable learnings
- [ ] Standard trace template designed and documented
- [ ] Consolidation produces useful, non-redundant knowledge
- [ ] At least 1 decision outcome influences a future generation
