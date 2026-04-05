---
title: "G4: Self-Learning Evolution Path"
type: research-finding
project: autonomous-agent-research
project-code: AAR
goal: G4
status: complete
created: 2026-04-05
author: freddie
recommendation: adopt-autonomous-with-guardrails
---

# G4: Self-Learning Evolution Path

## Summary

Evolve Freddie's reflect→consolidate→distill pipeline to autonomous operation with 5 validation gates. Routine refinements auto-apply. Unusual changes stage for human review. User notified in /morning briefing.

## Current vs Proposed Pipeline

| Stage | Current | Proposed |
|-------|---------|----------|
| **Reflect** | Gemini call after workflow, saves to inbox | Same + **importance scoring** (1-10). Only score 7+ enter pipeline. |
| **Consolidate** | Manual trigger at 3+ inbox items | **Autonomous** — runs after every session, no threshold needed |
| **Validate** | None | **NEW: 5 validation gates + LLM judge panel** |
| **Distill** | User reviews and approves soul diff | **Auto-apply if all gates pass**. Stage for human if any gate fails. |
| **Notify** | None | **Morning briefing** reports what changed overnight |

## The 5 Validation Gates

### Gate 1: Constitution
**What it checks:** Does the proposed change align with the agent's core identity as defined in their soul file's opening sections (backstory, philosophy)?
**Implementation:** LLM call with the current soul file + proposed diff. Prompt: "Does this addition contradict or dilute the agent's established character? Answer YES to block, NO to pass."
**Failure mode:** Blocks changes that would make McCall less cautious, Holmes less investigative, etc.

### Gate 2: Regression
**What it checks:** Does the new rule contradict any existing soul trait or hard rule?
**Implementation:** Extract all existing rules/instincts from the soul file. Compare proposed addition against each for semantic contradiction. Use embedding similarity — if a new rule is >0.85 similar to an existing rule but says the opposite, block.
**Failure mode:** Blocks "always trust the first approach" when soul already says "measure twice, cut once."

### Gate 3: Size
**What it checks:** Is the change small enough to be a refinement, not a rewrite?
**Implementation:** Max 3 lines added per distill cycle. Max 1 line removed. No modifications to existing lines (additions only for auto-apply).
**Failure mode:** Large diffs get staged for human review — they represent significant identity shifts.

### Gate 4: Drift
**What it checks:** Has the soul file changed too much recently?
**Implementation:** Track total lines added/removed over rolling 30-day window. If cumulative changes exceed 10% of the soul file's total lines, pause auto-apply and stage all pending changes for human review.
**Failure mode:** Prevents gradual personality drift that's invisible per-change but significant in aggregate.

### Gate 5: Safety
**What it checks:** Does the change touch protected sections?
**Implementation:** Soul files have sections marked as immutable: backstory, core philosophy, hard rules. Changes to these sections ALWAYS require human approval regardless of other gates.
**Failure mode:** Protects the foundational identity from any automated modification.

## LLM Judge Panel

After the 5 gates pass, a cross-model judge panel provides final validation:

- **3 judges**: Gemini, MiniMax, Grok (our existing tri-critic models)
- **Voting**: Unanimous approval required. One dissent = block.
- **Prompt**: "Review this proposed soul file change. Is it (a) supported by sufficient evidence, (b) non-contradictory to existing traits, (c) genuinely useful for future work? Vote APPROVE or REJECT with reason."
- **If blocked**: Change is staged for human review with judge reasoning attached.

## Notification Flow

```
Session ends → reflect (importance scoring)
  → score 7+? → consolidate (autonomous)
    → distill proposes soul diff
      → 5 gates pass? → judge panel pass?
        → YES: auto-apply, log change, notify in /morning
        → NO: stage for human review, notify in /morning
```

**Morning briefing addition:**
```
Soul Updates:
  ✓ McCall: added "boundary-first" instinct (12 sessions, 3 judges approved)
  ⏸ Holmes: proposed "always triangulate" — blocked by drift gate (10% cap reached). Review needed.
```

## Gap Analysis vs Phantom

| Mechanism | Phantom | Freddie (Proposed) |
|-----------|---------|-------------------|
| Session observation | Every session | Every session (reflect) |
| Importance filtering | None (all observations processed) | **Score 7+ only** — reduces noise |
| Consolidation | Periodic compression | **Per-session** autonomous |
| Validation gates | 5 gates (constitution, regression, size, drift, safety) | **Same 5 gates** — adapted for soul files |
| Judge panel | 3x Sonnet, minority veto | **Gemini + MiniMax + Grok**, unanimous required |
| Auto-apply | Always (if gates pass) | **Same** — with morning notification |
| Human gate | None | **On gate failure only** — safer than Phantom |
| Protected sections | Not documented | **Backstory + philosophy + hard rules** — immutable |

## Implementation Priority

| Component | Effort | Impact | Priority |
|-----------|--------|--------|----------|
| Importance scoring in reflect | Low | Medium | P1 — reduces noise immediately |
| Autonomous consolidation (remove threshold) | Low | Medium | P1 — already works, just remove the 3-item gate |
| 5 validation gates | Medium | High | P2 — core safety infrastructure |
| LLM judge panel for distill | Medium | High | P2 — final quality filter |
| Morning briefing integration | Low | Medium | P3 — notification layer |
| Drift tracking (30-day window) | Low | Medium | P3 — long-term safety |

## Applicability to The Foundry
- Self-evolution with guardrails is essential for multi-tenant agents that need to improve per-client without manual tuning
- The 5-gate model ports directly to any agent system
- Tag: **Foundry-applicable**
