---
title: Phase 3 — Automate Consolidation & Sync
created: 2026-03-28 02:24:20 AM
tags: [action-plan, phase-3, automation, consolidation]
parent: "[[notes/inbox/action-plan-validation-and-improvements]]"
status: pending
---

# Phase 3: Automate Consolidation & Sync

> Remove manual steps from the learning pipeline.

## Prerequisites
- Phase 2 complete — enough data to know automation won't process noise

## Tasks

### 3.1 — Auto-consolidation hook
Add to SessionStop hooks: check inbox counts per agent, auto-consolidate when 3+ items. Must handle errors gracefully — don't block session exit.

### 3.2 — Validate auto vs manual
Compare auto-consolidated output with manually triggered output. Same quality? Any regressions?

### 3.3 — Confidence tracking
Add `confidence: N` (integer, starts at 1) to knowledge file frontmatter. Incremented when the same learning is confirmed in a subsequent consolidation.

### 3.4 — Confidence-aware consolidation
When Gemini merges inbox items that match existing knowledge, increment confidence rather than creating duplicates. High-confidence items (5+) are candidates for soul distillation.

### 3.5 — Pruning strategy
- Decisions with no outcome after 30 days → archive
- Knowledge files with confidence 1 and no reference in 30 days → flag for review
- Implement as a `/brain prune` command

### 3.6 — Staleness detection
Track last-referenced date per knowledge file in brain.db sync_meta. Surface stale files in `/brain stats` output.

## Definition of done
- [ ] Consolidation runs automatically without manual triggers
- [ ] Knowledge files carry confidence scores
- [ ] Pruning command exists and works
- [ ] Stale files are detectable
