---
title: Phase 5 — Soul Distillation (End-to-End)
created: 2026-03-28 02:24:20 AM
tags: [action-plan, phase-5, soul-distillation, self-learning]
parent: "[[notes/inbox/action-plan-validation-and-improvements]]"
status: pending
---

# Phase 5: Soul Distillation (End-to-End)

> Run distill-soul.ts for real and validate the full 3-layer pipeline.

## Prerequisites
- Phase 2+3 — 20+ consolidated knowledge items with confidence scores
- Phase 4 — cross-agent data enriches the knowledge pool

## Tasks

### 5.1 — Check readiness
Run `bun src/tools/consolidate-memory.ts --list` — which agents have 5+ knowledge items? Target the richest agent (likely Tala).

### 5.2 — Run distill-soul.ts
```bash
bun src/tools/distill-soul.ts --agent tala
```
Review the proposed diff. Does it identify truly proven patterns (high confidence, repeated across sessions)?

### 5.3 — Evaluate the diff
- Is it meaningful? Would it change how Tala approaches future generations?
- Does it avoid noise? Generic advice shouldn't make it to the soul level.
- Does it respect the soul's voice? The diff should sound like Tala, not like a knowledge file.

### 5.4 — User approval
Present the diff. User decides: approve, reject, or modify.

### 5.5 — Validate impact
After soul update, run a generation with the same genre as a previous session. Compare output quality. Does the soul update produce measurably better results?

### 5.6 — Document criteria
What makes a good soul update? Write guidelines for future distillation runs.

## Definition of done
- [ ] At least one soul file updated with a proven learning
- [ ] The update visibly improves output quality
- [ ] Criteria documented for future runs
