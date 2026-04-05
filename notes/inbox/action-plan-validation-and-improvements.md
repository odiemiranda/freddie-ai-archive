---
title: Action Plan — Validation & Improvements
created: 2026-03-28 02:24:20 AM
tags: [action-plan, architecture, validation, improvements]
status: active
---

# Action Plan — Validation & Improvements

> Post-architecture review. Systematic validation of what we built, then targeted improvements based on real usage data.

## Guiding Principle

**Validate before improving.** Every improvement should be informed by real test results, not assumptions. Run the system, observe what breaks, then fix what matters.

---

## Phase 1: Validate the Orchestrator Pattern

> **Goal:** Prove that subagent dispatches work for all 3 variation-loop skills.
> **Why first:** This is the newest and most impactful untested change. If it doesn't work, we need to know before building on top of it.
> **Detail doc:** [[action-plan-phase-1-orchestrator-validation]]

| # | Task | Skill | Status |
|---|------|-------|--------|
| 1.1 | Run /suno with orchestrator — full 3-variation generation | suno | pending |
| 1.2 | Run /lyrics with orchestrator — full 3-variation generation | lyrics | pending |
| 1.3 | Run /minimax-music with orchestrator — full variation + API generation | minimax-music | pending |
| 1.4 | Test adjust path — trigger a user adjustment, verify new subagent dispatches correctly | suno | pending |
| 1.5 | Document friction points — what broke, what was awkward, what needs tuning | all | pending |
| 1.6 | Fix issues found in 1.1–1.5 | all | pending |

**Success criteria:** All 3 skills produce complete output through the orchestrator pattern without manual intervention beyond normal user review.

---

## Phase 2: Stress-Test the Learning Loop

> **Goal:** Generate enough real data to validate reflection, consolidation, and decision tracking.
> **Why second:** Phase 1 generates real workflow traces. Phase 2 uses those traces to test whether the learning pipeline actually works.
> **Detail doc:** [[action-plan-phase-2-learning-loop]]

| # | Task | Status |
|---|------|--------|
| 2.1 | Run 5+ real generation sessions across different genres/styles | pending |
| 2.2 | Verify reflect.ts produces useful learnings (not generic filler) for each session | pending |
| 2.3 | Audit trace quality — compare rich traces vs thin traces, identify minimum trace requirements | pending |
| 2.4 | Design a standard trace template/schema for consistent quality | pending |
| 2.5 | Run consolidation after 3+ inbox items accumulate per agent | pending |
| 2.6 | Verify consolidated knowledge is non-contradictory and useful | pending |
| 2.7 | Check brain.db decisions — are agents actually querying past decisions in node 002? | pending |
| 2.8 | Test decision outcome tracking — update an existing decision with outcome, verify it influences future queries | pending |

**Success criteria:** 20+ decisions in brain.db with meaningful context. Consolidation produces knowledge that agents actually reference. Traces follow a consistent format.

---

## Phase 3: Automate Consolidation & Sync

> **Goal:** Remove manual steps from the learning pipeline.
> **Why third:** We need enough data (Phase 2) to know the automation won't just process noise.
> **Detail doc:** [[action-plan-phase-3-automation]]

| # | Task | Status |
|---|------|--------|
| 3.1 | Add SessionStop hook that checks inbox counts and auto-consolidates when 3+ items | pending |
| 3.2 | Test auto-consolidation doesn't produce worse results than manual | pending |
| 3.3 | Add confidence tracking to knowledge files — "times confirmed" counter | pending |
| 3.4 | Update consolidation to merge confidence when deduplicating | pending |
| 3.5 | Add brain.db pruning strategy — archive decisions older than N days with no outcome | pending |
| 3.6 | Add staleness detection — flag knowledge files not referenced in 30+ days | pending |

**Success criteria:** Zero manual consolidation needed. Knowledge files carry confidence scores. Stale entries are flagged automatically.

---

## Phase 4: Cross-Agent Learning

> **Goal:** Learnings propagate between agents, not just within them.
> **Why fourth:** Requires stable consolidation (Phase 3) before cross-pollination makes sense.
> **Detail doc:** [[action-plan-phase-4-cross-agent]]

| # | Task | Status |
|---|------|--------|
| 4.1 | Audit shared/knowledge/ — what's there, is it being read by individual agents? | pending |
| 4.2 | Add push mechanism — when consolidation writes to shared/, notify relevant agents | pending |
| 4.3 | Define relevance rules — which shared learnings apply to which agents | pending |
| 4.4 | Test: Tala discovers a BPM handling insight, verify Sol picks it up in next generation | pending |
| 4.5 | Add cross-agent decision queries — "what did any agent learn about jazz fusion?" | pending |

**Success criteria:** A learning from one agent surfaces in another agent's workflow within 1 session of being consolidated.

---

## Phase 5: Soul Distillation (End-to-End)

> **Goal:** Run distill-soul.ts for real and validate the full 3-layer pipeline.
> **Why fifth:** Requires 20+ consolidated knowledge items (Phase 2+3) and cross-agent data (Phase 4).
> **Detail doc:** [[action-plan-phase-5-soul-distillation]]

| # | Task | Status |
|---|------|--------|
| 5.1 | Check knowledge counts per agent — do any have 5+ consolidated items? | pending |
| 5.2 | Run distill-soul.ts for the agent with most knowledge (likely Tala) | pending |
| 5.3 | Review the proposed soul diff — is it meaningful? Does it capture proven patterns? | pending |
| 5.4 | User approves or rejects the soul update | pending |
| 5.5 | If approved, verify the soul change improves next generation | pending |
| 5.6 | Document what makes a good soul update vs noise | pending |

**Success criteria:** At least one soul file updated with a proven learning. The update visibly improves output quality in subsequent generations.

---

## Phase 6: Advanced Improvements

> **Goal:** Polish and optimize based on everything learned in Phases 1-5.
> **Detail doc:** [[action-plan-phase-6-advanced]]

| # | Task | Status |
|---|------|--------|
| 6.1 | Tune FTS5 ranking — boost title field weight for reference lookups | pending |
| 6.2 | Add negative learning loop — surface past rejections as warnings during generation | pending |
| 6.3 | Add second-model validation for reflections (Grok validates Gemini's extracted learnings) | pending |
| 6.4 | Split workflow.md into planning-nodes and build-nodes for cleaner subagent context | pending |
| 6.5 | Build /usage skill — manual session summary command for total token/cost breakdown | pending |
| 6.6 | Investigate terminal popup on Windows for SessionStart/Stop hooks | pending |

**Success criteria:** Measurable improvement in discovery accuracy, learning quality, and developer experience.

---

## Execution Notes

- **One phase at a time.** Don't start Phase 2 until Phase 1 is validated.
- **Phase 1 and 2 can overlap** — real generation sessions (Phase 2) naturally test the orchestrator (Phase 1).
- **Phase-gated:** At each phase boundary, review results, save to memory, update this doc.
- **Each session picks 2-3 tasks** from the current phase. Don't try to do everything at once.
- **This doc is the source of truth** for progress. Update status as tasks complete.
