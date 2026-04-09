---
title: "Distill Knowledge Classifier"
type: brief
project: autonomous-self-learning
project-code: ASL
status: approved
created: 2026-04-09
updated: 2026-04-09
author: holmes
revised_by: mccall, freddie
supersedes: BRIEF-reference-drift-filter.md
tags: [asl, distill, classification, memory-boundaries]
---

# Distill Knowledge Classifier

> **Revision note.** This brief was originally scoped as a reference-drift similarity gate. McCall's discovery phase falsified both core assumptions: (1) the named "reference" files don't exist in `vault/studio/references/` — they live in `memory/{agent}/knowledge/`; (2) cosine-similarity has no clean threshold gap (known-negative Echo proposal scored 0.40, higher than all four known-positive Tala proposals at 0.17–0.31). The similarity-filter approach is dead. This revision reframes around the actual root cause: **distill has no step that classifies a knowledge item as IDENTITY vs TECHNIQUE before promoting it to soul.**

## Problem

On 2026-04-09, the autonomous distill pipeline staged 6 Tala soul proposals. All 6 were rejected during human review. **Four of the six were TECHNIQUE-class content** — structural generation rules, not identity material — that distill promoted to soul proposals anyway:

| # | Proposed "soul rule" | Actual source | Class |
|---|---|---|---|
| 2 | BGM Canonical Envelope | `memory/tala/knowledge/suno-prompt-design.md` | TECHNIQUE |
| 3 | Descriptive Prompting Philosophy | `memory/shared/knowledge/suno-prompt-engineering-and-references.md` | TECHNIQUE |
| 4 | Word Repetition Awareness | `memory/tala/knowledge/suno-prompt-design.md` | TECHNIQUE |
| 5 | Music Card Integration | `memory/shared/knowledge/studio-architecture-and-tools.md` | TECHNIQUE |

**Who feels it.** Odie. He spent the review session triaging dead-on-arrival proposals — TECHNIQUE-class content that already lived in the right place (`knowledge/`) and had no business being promoted to soul. This is the third ASL incident in 48 hours where proposal noise consumed human review budget for zero soul-level value (Sol redundancy 04-08, Penny evidence 04-08, now Tala classification-miss 04-09).

**Cost.**
- Direct: ~30 min of human review time per batch like this.
- Indirect: erosion of trust in distill output. When 4 of 6 proposals are noise, the reviewer starts skim-rejecting — and a real identity-level surprise will be missed.
- Pipeline-shape: this is a **funnel violation** per `self-learning-pipeline.md`. Distill is supposed to *reduce*. Instead it is re-promoting content that `consolidate` already placed in its correct home (`knowledge/`), by classifying it as soul material when it isn't.

**Five Whys.**
1. Why were these proposals staged? → Distill generated them and gates/judges didn't catch them.
2. Why didn't gates catch them? → The 5 existing gates (safety, size, drift, regression, constitution) check soul-vs-soul shape concerns, not content-class. There is no step that distinguishes IDENTITY from TECHNIQUE.
3. Why is there no IDENTITY/TECHNIQUE distinction in distill? → Distill was built with the implicit assumption that anything that reaches `knowledge/` and isn't already in soul is a candidate for promotion. The taxonomy lives in `memory-boundaries.md` but is not enforced in code.
4. Why does that assumption break? → Because `knowledge/` holds BOTH kinds of content. The memory-boundaries rule is explicit: soul = WHO (identity, reflexes, judgment), knowledge = WHAT (techniques, rules, mechanics). Consolidation correctly puts both in `knowledge/`. Distill incorrectly treats both as equally promotable.
5. Why does the pipeline have no enforcement of the memory-boundaries taxonomy? → It was never wired in. `memory-boundaries.md` is a human-facing doc; no pipeline stage checks it. The rule existed on paper but not in code.

**Root cause (5th why):** The ASL pipeline has no step that enforces the `memory-boundaries.md` taxonomy on proposal promotion. Distill treats `knowledge/` content as soul-candidate without asking whether the content is identity material or technique material. The fix is a classification step in distill that labels each proposal IDENTITY / TECHNIQUE / DUPLICATE and only stages IDENTITY proposals.

## Hypothesis

> We believe that **adding a content-class classification step to distill** will **eliminate TECHNIQUE-class proposals from reaching human review** for **Odie (the human reviewer)** because **all 4 known-positive Tala rejections from 04-09 were TECHNIQUE-class (structural generation rules), and a classifier that labels each proposal as IDENTITY/TECHNIQUE/DUPLICATE before staging would have dropped all 4 without a similarity threshold.**

## Why not a similarity filter (the original approach)

McCall's discovery phase produced cosine scores via `hybridSearchChunks` for 4 known-positives and 2 known-negatives:

| Proposal | Status | Top cosine |
|---|---|---|
| BGM Canonical Envelope | rejected (positive) | 0.31 |
| Descriptive Prompting | rejected (positive) | 0.17 |
| Word Repetition | rejected (positive) | 0.24 |
| Music Card Integration | rejected (positive) | 0.28 |
| Echo Gemini critique filter | **approved** (negative) | **0.40** |
| Echo seven-factor critique | approved (negative) | <0.10 |

The known-negative Echo proposal scores **higher** than every known-positive. No threshold exists that separates signal from noise. Any threshold low enough to filter the Tala noise (≤0.30) would also kill the legitimate Echo approved changes. Similarity-as-scalar is not the right signal.

A classifier (IDENTITY vs TECHNIQUE) is a **discrete taxonomy** with no calibration problem. It's also semantically correct: the 4 Tala rejections weren't "too similar to something" — they were the *wrong class* of content for the soul file.

## Goals

1. **Zero TECHNIQUE-class proposals reach human review.** Measured by: re-running the 04-09 Tala batch through the new pipeline produces ≤2 staged proposals (the non-TECHNIQUE rejections #1 and #6), not 6.
2. **No regression on legitimate IDENTITY proposals.** Measured by: replay the last 20 *approved* soul changes across all agents — all are classified IDENTITY and pass through.
3. **Fail-closed behavior preserved.** If the classifier errors (LLM API down, malformed response, timeout), the proposal is *staged for human review with the error noted*, not silently dropped and not silently passed.
4. **Cheap enough to run on every proposal.** Classification adds at most one LLM call per proposal batch. No per-proposal LLM fan-out that would multiply cost linearly with proposal count.

## Constraints

- **Taxonomy is authoritative.** The IDENTITY / TECHNIQUE / DUPLICATE labels must be derived from `memory-boundaries.md`. Any drift between the rule doc and the classifier prompt is a bug.
- **Must respect the funnel.** The classification is a *reduction* — TECHNIQUE and DUPLICATE proposals are dropped (or routed back to knowledge), they do not expand the output.
- **Must fail closed** per `self-learning-pipeline.md`. Classifier errors → stage for review with reason `classifier-error: {detail}`, never silent skip.
- **Must not break the existing ASL-0023 rejection-blacklist behavior.** Classification is an additional filter, not a replacement.
- **Auditability.** Every classification decision must be logged: the proposal text, the label assigned, the classifier's reasoning, the source knowledge file. Future debuggers need this to detect misclassification.
- **Single LLM call per batch where possible.** Distill already makes LLM calls; the classifier should piggyback on the existing generation call or run as a single follow-up call over the full proposal batch, not one call per proposal.
- **Route, don't just drop.** TECHNIQUE-class proposals should not vanish silently. If distill classifies an item as TECHNIQUE and its content is not already in a `knowledge/` file, the classifier must surface this as an anomaly (the content came from somewhere — where?). This guards against the content originating from `raw/` or an inbox item that never consolidated properly.

## Assumptions (flagged)

| # | Assumption | Confidence | Risk if wrong |
|---|---|---|---|
| A1 | An LLM given `memory-boundaries.md` + a proposal can reliably label IDENTITY / TECHNIQUE / DUPLICATE | 80% | Requires prompt engineering; a small eval fixture (today's 4 + 20 known-approved) tests this cheaply |
| A2 | Today's 4 known-positives would all classify as TECHNIQUE by the classifier | 85% | Directly falsifiable with one LLM call during discovery — must be validated before task finalization |
| A3 | The last 20 known-approved soul changes would all classify as IDENTITY | 75% | Same eval fixture as A1. If any approved change classifies as TECHNIQUE, either the classifier is wrong or the historical approval was wrong — either way, needs inspection |
| A4 | Classification is robust enough that we don't need a confidence score + human-review escalation for borderline cases | 60% | Borderline cases will exist (e.g., "reflexes" that are simultaneously identity and technique). Consider a 4th label UNCERTAIN that stages for review with the classifier's reasoning |
| A5 | Running the classifier as a single batch call (all proposals at once) is reliable enough that we don't need per-proposal calls | 70% | If the batch call is noisy or drops proposals, fall back to per-proposal. Measurable in the discovery phase |

A2 and A3 are the gating assumptions — the task should not be written until they're validated against a real LLM call.

## Success Criteria

- [ ] Re-running the 04-09 Tala distill batch produces ≤2 staged proposals (not 6), and the 4 TECHNIQUE-class items are routed to `knowledge/` or dropped with reason logged.
- [ ] No legitimate approved soul change from the last 20 days classifies as TECHNIQUE or DUPLICATE.
- [ ] Classification decisions are logged to brain.db (or equivalent) with: proposal text, label, classifier reasoning, source knowledge file. Queryable for debug.
- [ ] When the classifier errors (forced via LLM failure injection in tests), the proposal is staged with reason `classifier-error: {detail}`, not auto-applied and not silently dropped.
- [ ] The Tala soul does not gain `BGM Canonical Envelope`, `Descriptive Prompting Philosophy`, `Word Repetition Awareness`, or `Music Card Integration` rules — those stay in knowledge.
- [ ] A small eval fixture (4 known-positives + 20 known-approved negatives) is checked into the repo and run in CI. It acts as the regression guard against classifier drift.

## Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| **Classifier mislabels a legitimate IDENTITY proposal as TECHNIQUE** — silent loss of real soul material | Medium | High | Log all classifications; weekly review of TECHNIQUE-routed items; eval fixture in CI; consider UNCERTAIN label that stages instead of drops |
| **Classifier is noisy** — same proposal gets different labels on re-run | Medium | Medium | Lock the prompt and model version; log model name + prompt hash per decision; add determinism test to the eval fixture |
| **Taxonomy ambiguity** — real content that is genuinely both IDENTITY and TECHNIQUE gets dropped | Medium | Medium | Add UNCERTAIN label; escalate borderline cases to human review with the classifier's reasoning attached |
| **Cost** — LLM call per distill run adds latency and API spend | Low | Low | Distill already runs async in the daemon; piggyback on existing call if possible |
| **Memory-boundaries rule drifts from classifier prompt** | Low | High (taxonomy becomes fiction) | Classifier prompt is generated from `memory-boundaries.md` at runtime, not hardcoded. Single source of truth. |
| **Scope creep** — classifier ambition expands into "rewrite memory-boundaries.md" | Medium | Medium | This brief is strictly about enforcing the existing taxonomy, not revising it. Explicit out-of-scope. |

## Options

### Option 1 — Knowledge-vs-soul delta gate (SIMILARITY-BASED)

A 6th gate that compares each proposal against the agent's own `knowledge/` corpus using `hybridSearchChunks`. High similarity → filter.

- **Pros:** Reuses existing brain.db infra. No new LLM dependency. Deterministic.
- **Cons:** Hits the same cosine-gap problem McCall found. The known-negative Echo approved proposal scored higher (0.40) than all known-positives (0.17–0.31) against its own knowledge file — similarity is a structural signal about *where content is mirrored in the memory store*, not about *whether the content is IDENTITY or TECHNIQUE*.
- **Effort:** 1 day.
- **Verdict:** **REJECT.** Dead on arrival per McCall's cosine evidence.

### Option 2 — Distill input scoping

Change distill's contract to read only `memory/{agent}/inbox/` deltas since the last soul update, not all of `knowledge/`.

- **Pros:** Treats a funnel-shape concern at its origin. Architecturally aligned with `self-learning-pipeline.md`.
- **Cons:** Creates a race condition with consolidation — if consolidate merges inbox into knowledge before distill runs (which is the current ordering), inbox is empty and distill has nothing to read. Fundamentally misidentifies the bug: the issue isn't WHAT distill reads, it's HOW it decides what's promotable. Narrowing the input window just hides the classification problem, doesn't solve it.
- **Effort:** 2-3 days + consolidation synchronization work.
- **Verdict:** **REJECT.** Wrong fix for the actual bug.

### Option 3 — Classify IDENTITY / TECHNIQUE / DUPLICATE ← **RECOMMENDED**

Add a classification step to distill that labels each proposal before it reaches gates. TECHNIQUE and DUPLICATE labels drop the proposal (with audit log). IDENTITY proposals continue to the existing gate pipeline.

- **Pros:**
  - Sidesteps the cosine-threshold problem entirely. Binary per-class, no calibration.
  - Matches the existing taxonomy in `memory-boundaries.md` — enforces a rule that already exists on paper.
  - Semantically correct: the 4 Tala rejections weren't "too similar to something," they were the wrong class of content for soul.
  - Cheap: one LLM call per batch (not per proposal) piggybacks on distill's existing LLM usage.
  - Auditable: the classifier's reasoning becomes part of the decision log, far more debuggable than "cosine was 0.42."
- **Cons:**
  - Adds LLM dependency to every distill run (mitigated: distill already calls LLMs, and runs async in daemon).
  - Classification can be noisy — needs prompt locking, eval fixture, and UNCERTAIN label for borderline cases.
  - Taxonomy-vs-prompt drift is a new failure mode — mitigated by generating the prompt from `memory-boundaries.md` at runtime.
- **Effort:** 2 days (prompt design + eval fixture + integration + fail-closed error handling).
- **Verdict:** **RECOMMENDED.** The only option that matches the actual shape of the bug.

## Out of Scope

- Tri-critic escalation flow (separate concern, separate brief).
- Staged-proposal writer truncation bugs (separate task).
- Rejection #1 "Agent Autonomy in Critic Reconciliation" and #6 "Consolidate Reroll Expectation" — different root causes; not TECHNIQUE-class.
- Rewriting `memory-boundaries.md` (the rule stands; this brief enforces it).
- Per-agent classifier specialization beyond the single shared prompt.
- Filtering at the reflect stage.
- Filtering at the consolidate stage.
- Retro-classifying existing `knowledge/` content to audit for misplaced IDENTITY items.
- Adding a content-class field to staged-proposal frontmatter (may be a follow-up).

## Unresolved Questions

1. **Does an LLM given `memory-boundaries.md` + a proposal reliably produce the expected label?** McCall must validate A2 and A3 against a real LLM call before task finalization. Run the classifier over the 4 known-positives (expect: all TECHNIQUE) and ≥5 known-approved negatives (expect: all IDENTITY). If A2 or A3 fails, abort and re-brief.
2. **Which LLM model?** Gemini (already in use for reflect/consolidate/distill) is the default. Confirm in task doc.
3. **Batch-call or per-proposal?** Start with batch (cheaper). Fall back to per-proposal if batch is noisy. Measurable during discovery.
4. **UNCERTAIN as a fourth label?** Decision: yes. Borderline cases stage for human review with classifier reasoning, they do not silently drop. Task doc formalizes this.
5. **Where does the classifier prompt live?** Proposed: generated at runtime from `.claude/rules/memory-boundaries.md` + a wrapper template in `src/libs/`. Keeps the taxonomy single-sourced. Task doc confirms.

---

**Confidence in this brief: 80%.** The root cause is validated (cosine evidence from McCall). The fix direction is sound (discrete taxonomy, no calibration). The remaining unknowns are A2/A3 — both cheaply falsifiable in task discovery. Recommend McCall picks this up next.
