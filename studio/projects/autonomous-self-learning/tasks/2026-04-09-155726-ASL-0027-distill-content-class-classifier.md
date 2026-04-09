---
title: "Distill Content-Class Classifier"
type: task
project: autonomous-self-learning
project-code: ASL
task-id: ASL-0027
status: cancelled
cancelled: 2026-04-09
cancellation-reason: "ASL subsystem archived 2026-04-09. See ARCHIVED.md. Classifier was not implemented."
created: 2026-04-09
author: mccall
assignee: ryan
effort: 2 days
depends-on: [ASL-0023, ASL-0024]
supersedes-brief: BRIEF-knowledge-classifier.md
tags: [asl, distill, classification, fail-closed, eval-fixture, cancelled]
---

> **CANCELLED 2026-04-09.** The entire ASL subsystem was archived in the same session this task was scoped. The classifier was never implemented. The cost-benefit analysis that killed ASL applies directly: the classifier would have been ~60% noise reduction on a pipeline we're no longer running. See `../ARCHIVED.md` for the full rationale and `src/archive/asl/README.md` for the code location. This task doc is preserved as engineering history — it represents the last serious attempt to rescue ASL before the decision to shelve it.

# ASL-0027 — Distill Content-Class Classifier

## What to Build

Add a **content-class classification step** to the autonomous distill pipeline that labels every parsed proposal as `IDENTITY | TECHNIQUE | DUPLICATE | UNCERTAIN` before gates run. The classifier enforces the `memory-boundaries.md` taxonomy in code: soul = WHO (identity, reflexes, judgment), knowledge = WHAT (techniques, rules, mechanics). TECHNIQUE and DUPLICATE drop with audit. IDENTITY continues to gates. UNCERTAIN stages for human review with the classifier's reasoning attached.

### Pipeline position (exact call order)

```
distillSoul (Gemini)
  → parseDistillOutput
  → ASL-0023 rejection-blacklist hash filter       (unchanged — runs first, cheap)
  → classifyProposals  ← NEW                       (this task)
      • IDENTITY   → continue
      • TECHNIQUE  → drop + log + logSoulChange(status="filtered-technique")
      • DUPLICATE  → drop + log + logSoulChange(status="filtered-duplicate")
      • UNCERTAIN  → stage with reason=`classifier-uncertain: {reasoning}`
      • error/unknown label → stage with reason=`classifier-error: {detail}`
  → runAllGates        (unchanged)
  → judge panel        (unchanged)
  → apply/stage        (unchanged)
```

The classifier runs **after** the ASL-0023 rejection-hash filter so cheap exact-match rejections short-circuit before any LLM call. It runs **before** gates so TECHNIQUE content never consumes gate or judge budget.

### Architecture decisions

**1. Classifier file.** New module: `src/libs/distill-classifier.ts`. Standalone from `autonomous-distill.ts` for testability and because `autonomous-distill.ts` is already 1029 lines. Exports the classifier function, types, and the prompt builder.

**2. Standalone LLM call (not piggyback).** `callDistillGemini` in `src/tools/distill-soul.ts` is a single-shot prompt with a locked output schema (`=== PROPOSAL N ===` blocks) and a cache key (`DISTILL_PROMPT_VERSION`). Extending that prompt to also return classification labels would:
- invalidate the distill cache on every classifier tweak
- couple prompt-engineering for two different tasks to one temperature and token budget
- make it harder to test the classifier in isolation
- make it harder to fail closed (if classification output is malformed, the whole distill call has to be retried)

Make the classifier a **separate Gemini call** that runs on `ParsedProposal[]` after parsing. One batch call per agent per run (all of that agent's proposals in one call). Model: `gemini-2.5-flash` (same as distill). Temperature `0.0` (classification should be deterministic). Timeout `30_000ms`. `maxOutputTokens: 2048`.

**3. Batch call, with per-proposal fallback.** Send all proposals for one agent in a single prompt and parse a JSON array response. If the batch call returns fewer labels than proposals, or any label is missing/unknown, fall back to per-proposal calls for the affected items. Fallback is mandatory — do not silently drop unclassified proposals.

**4. Four labels, no numeric confidence.** Discrete taxonomy. No thresholds, no calibration. UNCERTAIN is the escape valve for borderline cases — the model is explicitly instructed to use it rather than guess.

**5. Prompt sourcing — runtime-assembled from `memory-boundaries.md`.** The classifier prompt reads `.claude/rules/memory-boundaries.md` at runtime and embeds the taxonomy table verbatim into the system portion of the prompt. This keeps `memory-boundaries.md` as the single source of truth and prevents prompt drift from the rule doc. The classifier module also computes a SHA-256 hash of the rule content and logs it with every decision. If the rule file is missing or unreadable, the classifier fails closed — every proposal stages with `classifier-error: memory-boundaries.md unreadable`.

**6. Audit trail — brain.db decisions + staged frontmatter fields.** Two writes per classification:
- **brain.db**: one `decisions` row per classification via `logDecision` (or the pattern already used by gates/judges). Fields: agent, proposal hash, label, reasoning, model name, prompt hash, source file if identifiable, timestamp.
- **staged frontmatter** (only when the proposal stages — UNCERTAIN or classifier-error): two new optional fields on `StagedProposal`, `classifierLabel` and `classifierReasoning`. Persisted via `writeStagedProposal` frontmatter. Read back by `listStagedProposals`. A third-party log file is rejected as scope creep — brain.db already handles structured decision history.

**7. Source-file traceability.** The classifier prompt asks Gemini to identify which knowledge file (from the `## Agent Knowledge` / `## Shared Knowledge` blocks of the distill call) the proposal came from. This is best-effort — if the model returns null, the audit log records null. Future ASL work can tighten this if needed. Do not block on perfect attribution.

### Label definitions (embedded in the classifier prompt, sourced from memory-boundaries.md)

| Label | Definition | Route |
|-------|------------|-------|
| `IDENTITY` | WHO the agent is. Reflexes, judgment, core beliefs, pet peeves, the line it won't cross, how it talks, permanent character traits. Material that belongs in `.claude/souls/{agent}.md`. | Continue to gates. |
| `TECHNIQUE` | WHAT the agent knows. Generation rules, platform mechanics, prompt-engineering recipes, structural templates, tool-usage workflows, technical quirks, anti-patterns with specific fixes. Material that belongs in `memory/{agent}/knowledge/*.md`, not soul. | Drop. Log with source file. Never stages. |
| `DUPLICATE` | Semantically equivalent to content already present in the current soul file (same reflex expressed differently, same pet peeve in different words). | Drop. Log with the soul snippet it duplicates. Never stages. |
| `UNCERTAIN` | The proposal legitimately straddles the IDENTITY/TECHNIQUE line (e.g., a signature move that's a technique but has become part of the agent's identity). Or the proposal is ambiguous to the classifier. | Stage with reasoning. Human decides. |

## Function Signatures

```typescript
// src/libs/distill-classifier.ts

export type ClassifierLabel = "IDENTITY" | "TECHNIQUE" | "DUPLICATE" | "UNCERTAIN";

export interface ClassifierResult {
  /** The original proposal — passed through by reference for convenience. */
  proposal: ParsedProposal;
  /** The label assigned by the classifier. */
  label: ClassifierLabel;
  /** One-to-three-sentence explanation of the decision, written by the model. */
  reasoning: string;
  /** Best-effort — the knowledge file this proposal appears to come from. Nullable. */
  sourceFile: string | null;
  /** SHA-256 of the memory-boundaries.md content used to build the prompt.
   *  Logged with the decision so drift is detectable. */
  rulePromptHash: string;
  /** Gemini model name used. */
  model: string;
}

export interface ClassifyProposalsInput {
  agent: string;
  proposals: ParsedProposal[];
  /** The current soul file content — used by the DUPLICATE check. */
  currentSoul: string;
}

export interface ClassifyProposalsOutput {
  results: ClassifierResult[];
  /** Set when the batch call failed and fallback also failed for some items.
   *  Items in this array MUST be staged by the caller with reason
   *  `classifier-error: {error}`. They do NOT appear in `results`. */
  errored: Array<{ proposal: ParsedProposal; error: string }>;
}

/**
 * Classify each proposal as IDENTITY / TECHNIQUE / DUPLICATE / UNCERTAIN.
 *
 * Execution:
 *   1. Load memory-boundaries.md. On read failure, every proposal goes to `errored`.
 *   2. Build the prompt (system taxonomy + agent context + proposals JSON).
 *   3. Single batch Gemini call.
 *   4. Parse response. If incomplete or any label is malformed/unknown, retry the
 *      missing items per-proposal.
 *   5. Per-proposal retries that still fail go to `errored`.
 *
 * FAIL-CLOSED CONTRACT:
 *   - LLM timeout / API error / malformed response → the affected proposal
 *     goes to `errored`, NOT dropped, NOT auto-passed.
 *   - Unknown label value → errored.
 *   - memory-boundaries.md unreadable → ALL proposals errored with
 *     `memory-boundaries.md unreadable: {detail}`.
 */
export async function classifyProposals(
  input: ClassifyProposalsInput,
): Promise<ClassifyProposalsOutput>;

/** Build the classifier prompt. Exported for tests and audit. */
export function buildClassifierPrompt(
  agent: string,
  currentSoul: string,
  proposals: ParsedProposal[],
  taxonomyDoc: string,
): string;

/** SHA-256 of memory-boundaries.md content. Exported for tests. */
export function hashTaxonomyDoc(content: string): string;

/** Load memory-boundaries.md. Throws on missing/unreadable. */
export function loadTaxonomyDoc(): string;
```

### Changes to `src/libs/autonomous-distill.ts`

Insert the classifier call between the ASL-0023 hash filter (line ~676) and the `for (const proposal of parsedProposals)` gate loop (line ~697):

```typescript
// ── ASL-0027: content-class classification ──────────────────────────────
// Runs after the cheap rejection-hash filter and before gates.
// IDENTITY → continue. TECHNIQUE/DUPLICATE → drop with audit.
// UNCERTAIN → stage for human review. errors → stage with classifier-error.
const classifyResult = await classifyProposals({
  agent,
  proposals: parsedProposals,
  currentSoul: readFileSync(soulPath, "utf-8"),
});

// Partition: what proceeds to gates vs. what drops vs. what stages now
const proposalsToGate: ParsedProposal[] = [];
for (const r of classifyResult.results) {
  if (r.label === "IDENTITY") {
    proposalsToGate.push(r.proposal);
    continue;
  }
  if (r.label === "TECHNIQUE" || r.label === "DUPLICATE") {
    // Drop with audit — never stages
    logSoulChange({
      agent,
      date: dayjs().format("YYYY-MM-DD"),
      linesAdded: 0,
      linesRemoved: 0,
      changeSummary: `[filtered-${r.label.toLowerCase()}] ${r.proposal.title}: ${r.reasoning}`,
      gateResults: undefined,
      judgeResults: undefined,
      status: r.label === "TECHNIQUE" ? "filtered-technique" : "filtered-duplicate",
    });
    outcomes.push({ /* same shape as the ASL-0023 filtered branch */ });
    continue;
  }
  // UNCERTAIN → stage with classifier reasoning
  await _stageProposal(
    agent, r.proposal, emptyGateResult, null,
    `classifier-uncertain: ${r.reasoning}`,
    undefined,
    { classifierLabel: "UNCERTAIN", classifierReasoning: r.reasoning },
  );
  outcomes.push({ /* staged */ });
}
// Errored proposals (classifier failure) — stage fail-closed
for (const e of classifyResult.errored) {
  await _stageProposal(
    agent, e.proposal, emptyGateResult, null,
    `classifier-error: ${e.error}`,
    undefined,
    { classifierLabel: null, classifierReasoning: null },
  );
  outcomes.push({ /* staged */ });
}

// Replace parsedProposals with the IDENTITY-only subset for the gate loop
for (const proposal of proposalsToGate) {
  // ... existing gate/judge loop unchanged
}
```

The new `status` values for `logSoulChange` are `"filtered-technique" | "filtered-duplicate"` — add to the existing status union. Extend `logSoulChange`'s type if needed. Do not rename the existing `"filtered-rejected"` from ASL-0023.

### Changes to `src/libs/staged.ts`

Extend `StagedProposal`:

```typescript
export interface StagedProposal {
  // ... existing fields ...
  /** ASL-0027: classifier label when the proposal staged because of the classifier
   *  (UNCERTAIN path). Null when staging happened for any other reason. */
  classifierLabel?: "IDENTITY" | "TECHNIQUE" | "DUPLICATE" | "UNCERTAIN" | null;
  /** ASL-0027: classifier's explanation text. Null when not applicable. */
  classifierReasoning?: string | null;
}
```

Wire the two new fields into:
- `writeStagedProposal` (frontmatter write path — use `fmEscape` for `classifierReasoning`)
- `readStagedFile` and `listStagedProposals` (frontmatter read path)
- `writeApprovedArchive` (same fields echoed through the archive)

### Changes to `_stageProposal` in `autonomous-distill.ts`

Add an optional fourth param for classifier metadata:

```typescript
async function _stageProposal(
  agent: string,
  proposal: ParsedProposal,
  gateResults: AllGatesResult,
  judgeResults: JudgePanelResult | null,
  reason: string,
  degraded?: StagedProposal["degradedQuorum"],
  classifier?: { classifierLabel: StagedProposal["classifierLabel"]; classifierReasoning: string | null },
): Promise<void>;
```

Pass-through only — write the fields into the `StagedProposal` object passed to `writeStagedProposal`.

### Eval fixture

**File:** `src/libs/__fixtures__/distill-classifier-eval.ts` (new directory `__fixtures__/`).

**Contents:**

```typescript
export interface ClassifierEvalCase {
  id: string;
  agent: string;
  proposal: ParsedProposal;
  expectedLabel: ClassifierLabel;
  notes?: string;
}

export const KNOWN_POSITIVES: ClassifierEvalCase[] = [
  // The four 04-09 Tala rejections — all expected TECHNIQUE
  // Proposal #2: BGM Canonical Envelope
  // Proposal #3: Descriptive Prompting Philosophy
  // Proposal #4: Word Repetition Awareness
  // Proposal #5: Music Card Integration
];

export const KNOWN_APPROVED: ClassifierEvalCase[] = [
  // At least 5 recent known-approved soul changes, expected IDENTITY.
  // Curate from git log .claude/souls/ and vault/studio/memory/*/approved/.
  // Candidates (discovery must verify each is unambiguous before including):
  //   - Tala "subordination to references / intent paramount" (04-08)
  //   - Rune "authenticity over rhyme fabrication" (04-08)
  //   - Sol "mastering the room" (04-08)
  //   - Echo "multi-stage workflow" (04-08)
  // DO NOT include borderline historical approvals (see Notes).
];

export const SYNTHETIC_UNCERTAIN: ClassifierEvalCase[] = [
  // 2-3 hand-crafted borderline cases that should classify UNCERTAIN.
  // Examples:
  //   - A signature move that is technique-like but part of agent identity
  //   - A judgment heuristic phrased with platform-specific language
];

export const ALL_CASES = [...KNOWN_POSITIVES, ...KNOWN_APPROVED, ...SYNTHETIC_UNCERTAIN];
```

**Eval runner:** New test file `src/libs/__tests__/distill-classifier.eval.test.ts` that loads the fixture and runs `classifyProposals` against each case. Expected assertion: `result.label === case.expectedLabel`. The eval test is gated behind the environment variable `RUN_CLASSIFIER_EVAL=1` because it makes live Gemini calls — running in CI is opt-in via the existing async test runner, not the default unit test path. Add an npm script `"test:classifier-eval": "RUN_CLASSIFIER_EVAL=1 bun test src/libs/__tests__/distill-classifier.eval.test.ts"`.

**Failure mode:** Any case mismatching its `expectedLabel` fails the test. A CI badge/workflow addition is out of scope — this task only checks in the fixture and runner. Freddie wires CI separately.

## Acceptance Criteria

- [ ] `src/libs/distill-classifier.ts` exists with the signatures above.
- [ ] Classifier reads `.claude/rules/memory-boundaries.md` at runtime and embeds the taxonomy in the prompt.
- [ ] Classifier fails closed on: LLM error, timeout (30s), malformed JSON, unknown label, missing taxonomy file. All failure modes stage with reason `classifier-error: {detail}`.
- [ ] `autonomous-distill.ts` calls `classifyProposals` between the ASL-0023 hash filter and the gate loop.
- [ ] TECHNIQUE and DUPLICATE proposals are dropped with `logSoulChange` status `filtered-technique` / `filtered-duplicate` and never reach gates.
- [ ] UNCERTAIN proposals stage with `classifier-uncertain: {reasoning}` and include `classifierLabel` + `classifierReasoning` in frontmatter.
- [ ] `StagedProposal` has two new optional fields persisted by `writeStagedProposal` and read back by `listStagedProposals`.
- [ ] The ASL-0023 rejection-hash filter still runs first — classifier never sees a previously-rejected proposal.
- [ ] Eval fixture `src/libs/__fixtures__/distill-classifier-eval.ts` is checked in with ≥4 known-positives (the 04-09 Tala batch) + ≥5 known-approved (expected IDENTITY) + ≥2 synthetic UNCERTAIN cases.
- [ ] `test:classifier-eval` npm script exists and passes against live Gemini when run locally.
- [ ] Unit tests cover: batch happy path, batch partial-response → per-proposal fallback, forced LLM error → errored[], forced malformed JSON → errored[], forced unknown label → errored[], missing `memory-boundaries.md` → all errored.
- [ ] Integration test re-runs the 04-09 Tala batch end-to-end with mocked Gemini and verifies 4 of 6 proposals are filtered at classify (TECHNIQUE) and 2 reach gates.
- [ ] Every classification decision is written to the `decisions` table in brain.db with: agent, proposal-hash, label, reasoning, model, rule-prompt-hash, source-file, timestamp.
- [ ] No auto-apply path changes. No gate logic changes. No judge logic changes. No soul file changes.
- [ ] `memory-boundaries.md` is **not** modified by this task.

## Files in Scope

**New:**
- `src/libs/distill-classifier.ts`
- `src/libs/__fixtures__/distill-classifier-eval.ts`
- `src/libs/__tests__/distill-classifier.test.ts` (unit — mocked Gemini)
- `src/libs/__tests__/distill-classifier.eval.test.ts` (eval — live Gemini, env-gated)
- `src/libs/__tests__/autonomous-distill-classify-integration.test.ts` (integration — replay 04-09 Tala batch with injected classifier deps)

**Modified:**
- `src/libs/autonomous-distill.ts` — insert classifier call, extend `_stageProposal` signature, extend `_AutonomousDistillTestDeps` with `classifyProposalsFn` for tests
- `src/libs/staged.ts` — add `classifierLabel` + `classifierReasoning` to `StagedProposal`, wire through writer/reader/archive
- `src/libs/soul-change-log.ts` (or wherever `logSoulChange` lives — grep for the existing `"filtered-rejected"` status to find it) — extend status union with `filtered-technique` and `filtered-duplicate`
- `package.json` — add `test:classifier-eval` script

**Read-only (referenced but not modified):**
- `.claude/rules/memory-boundaries.md` — loaded at runtime by the classifier
- `src/libs/gates.ts` — type references only
- `src/tools/distill-soul.ts` — prompt structure reference only
- `src/libs/gemini.ts` — `generateText` call

## Architectural Constraints

1. **`memory-boundaries.md` is the single source of truth for the taxonomy.** The classifier prompt derives from it at runtime. Do not hardcode taxonomy text in `distill-classifier.ts`. Do not import it as a string constant. Read the file, hash it, embed it.
2. **Fail closed — always.** Any classifier failure (timeout, API error, malformed JSON, unknown label, missing rule file) → stage with `classifier-error: {detail}`. Never auto-drop. Never auto-pass. Never skip classification silently on error.
3. **Funnel integrity.** Classification is a reduction. TECHNIQUE + DUPLICATE + rejected hash matches never expand into staged or auto-applied. UNCERTAIN still reduces vs. the pre-classify count because TECHNIQUE/DUPLICATE have been removed.
4. **ASL-0023 rejection blacklist runs first.** The cheap hash filter short-circuits known-rejected content before the classifier burns an LLM call. Do not reorder.
5. **One LLM call per agent per run (batch).** Do not fan out per-proposal as the primary path. Per-proposal fallback fires only when the batch response is incomplete or malformed.
6. **Gates and judges are untouched.** This task is additive to the distill pipeline. No existing gate is modified. No judge prompt is modified. No auto-apply quorum rule is modified.
7. **Audit trail is mandatory.** Every classification decision writes to brain.db `decisions`. Staged frontmatter carries classifier metadata when it's the staging reason. The approved-archive path preserves both.
8. **Prompt + model + rule-doc hash are versioned.** Log them with every decision so classifier drift is queryable after the fact.
9. **Eval fixture is checked in.** If the fixture ever starts failing, the build signal is immediate. The fixture is the regression guard against classifier drift.

## Out of Scope

- Any modification of `memory-boundaries.md`, `.claude/souls/*.md`, or any approved soul change.
- Any modification of `consolidate-memory.ts`, `reflect.ts`, existing gate logic, judge prompts, or apply logic.
- Retro-classifying existing staged/approved content (the fixture samples a handful; wholesale retro-audit is follow-up work).
- Per-agent classifier specialization — one shared prompt for all agents.
- Filtering at the reflect or consolidate stages.
- A "content-class" field on `ParsedProposal` itself — the label lives on `ClassifierResult`, not on the proposal, to preserve parse-vs-classify separation.
- CI workflow changes. Freddie wires the eval-runner into CI separately.
- A dedicated jsonl log file for classifier decisions — brain.db already handles this.

## Test Plan

**Unit (mocked Gemini via `_AutonomousDistillTestDeps` and a new `classifyProposalsFn` injection):**

1. Batch happy path — 4 proposals, 4 labels returned, all parse. Expect 4 results, 0 errored.
2. Batch partial response — 4 proposals, 2 labels returned. Expect per-proposal fallback on the missing 2. Both succeed. Expect 4 results, 0 errored.
3. Batch partial + fallback partial — 4 proposals, 2 returned by batch, per-proposal fallback succeeds for 1 and fails for 1. Expect 3 results, 1 errored.
4. Forced LLM timeout on batch and all per-proposal retries → all errored.
5. Forced malformed JSON response on batch → per-proposal fallback → all succeed → 0 errored.
6. Forced unknown label `"MAYBE"` → that proposal → errored.
7. `memory-boundaries.md` unreadable (mock `loadTaxonomyDoc` throw) → all proposals errored with taxonomy-missing reason. Zero LLM calls made.
8. Pipeline integration (`autonomousDistill` with injected `classifyProposalsFn`):
   - All 4 proposals labeled TECHNIQUE → 0 reach gates, 4 logSoulChange `filtered-technique` entries, no stage writes.
   - 2 IDENTITY + 2 TECHNIQUE → 2 reach gates, 2 filtered.
   - 1 UNCERTAIN → 1 staged with `classifier-uncertain` reason and correct frontmatter fields.
   - 1 errored → 1 staged with `classifier-error` reason.
9. `writeStagedProposal` + `readStagedFile` round-trip for the new fields. Extend existing `staged-roundtrip.test.ts`.
10. ASL-0023 hash filter still runs before classifier — verify ordering with a spy.

**Eval (live Gemini, env-gated):**

11. `RUN_CLASSIFIER_EVAL=1 bun test src/libs/__tests__/distill-classifier.eval.test.ts` — the fixture passes: all 4 known-positives → TECHNIQUE, all ≥5 known-approved → IDENTITY, all ≥2 synthetic borderline → UNCERTAIN.

**Integration (replay, mocked Gemini with the real 04-09 Tala output):**

12. Seed a fake `callDistillGeminiFn` that returns the verbatim 04-09 Tala distill output (6 proposals). Seed a `classifyProposalsFn` that returns the real classifier output from the eval run (4 TECHNIQUE, 2 non-TECHNIQUE). Run `autonomousDistill(["tala"])`. Assert: pipeline result has ≤2 staged proposals, 4 `filtered-technique` soul-change-log entries, no auto-applied proposals, no BGM Canonical Envelope / Descriptive Prompting / Word Repetition / Music Card Integration content in any written file.

## Notes

### A2/A3 validation — done by hand-classification, Ryan must replicate with real Gemini in discovery phase 1

I did not invoke a live LLM from this sub-agent session. I hand-classified each case against `memory-boundaries.md`. Ryan's **first task before writing the classifier prompt** is to replicate this with a one-shot Gemini call using a draft prompt, and confirm the labels match. If Gemini disagrees with my hand-labels on any case, Ryan must flag it to Freddie before continuing — it's either a prompt bug or a latent disagreement about the taxonomy that needs resolving before this ships.

**A2 — 4 known-positive Tala proposals from 04-09 (expected TECHNIQUE):**

| # | Proposal | Hand-label | Reasoning |
|---|----------|------------|-----------|
| 2 | BGM Canonical Envelope | **TECHNIQUE** | Structural generation recipe — a fixed template for BGM prompt structure. Pure mechanic. Belongs in `memory/tala/knowledge/suno-prompt-design.md` (where it already lives). |
| 3 | Descriptive Prompting Philosophy | **TECHNIQUE** | "Describe, don't prescribe" is a prompt-engineering rule. It's a WHAT-to-do mechanic, not a WHO-you-are reflex. Belongs in knowledge. |
| 4 | Word Repetition Awareness | **TECHNIQUE** | Suno-specific platform quirk — "Suno repeats words you emphasize." Platform mechanic, belongs in knowledge. |
| 5 | Music Card Integration | **TECHNIQUE** | Tool-usage workflow — "when the user mentions a music card, look it up." Tool-invocation rule, belongs in knowledge. |

All 4 hand-classify cleanly as TECHNIQUE. **A2 passes** against memory-boundaries.md. High confidence an LLM with the same rule doc will agree.

**A3 — recent approved soul changes (expected IDENTITY):**

Pulled from `git show eb8920a` (Apr 8 batch) and `git show 5e5f432` (Apr 8 follow-up) plus `vault/studio/memory/echo/approved/`.

| # | Change | Hand-label | Confidence |
|---|--------|------------|------------|
| 1 | Tala "core task goals always override critic suggestions / intent paramount" | **IDENTITY** | High — judgment reflex under pressure, pure WHO material |
| 2 | Rune "authenticity over rhyme fabrication, restructure rather than distort" | **IDENTITY** | High — craft reflex, personal commitment |
| 3 | Sol "mastering the room" (from commit message) | **IDENTITY** | Medium-High — philosophy statement |
| 4 | Tala "3-5 rerolls are typical, prompt works if worst generation is recognizable" | **UNCERTAIN / borderline** | Medium — expectation-setting is identity-ish but "3-5 rerolls" feels more TECHNIQUE |
| 5 | Tala "Subtle Intros [Short Fade In] trick" (04-08 approved) | **UNCERTAIN / arguably TECHNIQUE** | Low — this is a signature move, but the content is a specific tag trick. Historical approval may have been generous. |
| 6 | Echo "Expanded Suno Critique Filter Rules" (04-08 approved) | **IDENTITY-leaning / UNCERTAIN** | Medium — rules about how Echo weights critiques is a judgment reflex, but the content is Suno-specific which drags toward TECHNIQUE |
| 7 | Echo "Prioritized Suno Structural Critique" (04-08 approved) | **UNCERTAIN / arguably TECHNIQUE** | Medium — "seven-factor structural critique" is a methodology, closer to TECHNIQUE |
| 8 | Rune "Build with positive constraints, max three instruments per section" | **TECHNIQUE** | High — this is a generation rule, should not be in soul |

**A3 does NOT pass cleanly.** At least 4 of 8 recent approved soul changes classify as borderline UNCERTAIN or outright TECHNIQUE by strict application of memory-boundaries.md. **This is expected and does not abort the task.** It means:

1. The UNCERTAIN label is **mandatory** — not optional. Several legitimate historical approvals are genuinely borderline.
2. The "known-approved" eval fixture must be curated **carefully**. Only cases #1–#3 above are safe to use as IDENTITY-expected. Cases #4–#8 must not be baselined as IDENTITY because a correct classifier will reasonably label them UNCERTAIN or TECHNIQUE.
3. Ryan's discovery phase should **flag this finding to Freddie** — there's a latent question about whether the historical approval process was lenient, or whether memory-boundaries.md needs clarification. That's a separate brief, out of scope for ASL-0027. But the finding must be recorded.
4. A follow-up retro-audit task on existing soul content should be created after this ships.

**Recommendation to Ryan:** when building the fixture, use only the 3–5 highest-confidence IDENTITY approvals (judgment reflexes, philosophy statements, pet peeves, the line). Do not pad the fixture with borderline cases — the eval is a regression guard, not a coverage study.

### Recommendation — batch-call vs. per-proposal

**Batch as primary, per-proposal as fallback.** Typical distill runs produce 1–6 proposals per agent (the brief says "1-3 proposals is typical"). A single batch call is cheap and avoids linear LLM cost. Per-proposal fallback handles the rare case where the model returns an incomplete or malformed array. Do not make per-proposal the primary path — it burns budget unnecessarily.

### Recommendation — standalone LLM call vs. piggyback

**Standalone.** Justified in detail above (Architecture decision #2). The distill prompt is already large, cache-sensitive, and tuned for a different task. Coupling classification into it would invalidate the cache on every prompt tweak, entangle two jobs under one temperature, and make fail-closed behavior harder. One extra Gemini call per distill run is cheap — distill already runs async in the daemon.

### Second-order concerns Freddie should know about

1. **Historical soul content may already be contaminated.** My A3 analysis found that several of the last 20-ish approved soul changes straddle the IDENTITY/TECHNIQUE line. The classifier will not retroactively clean this up. A follow-up retro-audit task is worth queuing. Suggested ID: ASL-0028 "Retro-classify existing soul content against memory-boundaries.md."

2. **UNCERTAIN will be a meaningful fraction of output.** Based on A3, expect ~25–40% of real proposals to classify as UNCERTAIN in practice (not the eval fixture — real traffic). That means human review budget is not fully eliminated, only the clean-TECHNIQUE noise is. The brief's goal of "zero TECHNIQUE-class proposals reach review" holds. The goal of "human review budget drops to near-zero" does not. Set expectations accordingly.

3. **Memory-boundaries.md may need a tightening pass.** The taxonomy was written for humans. Some edge cases (judgment heuristics vs. generation rules, signature moves vs. techniques) are genuinely ambiguous on paper. If the eval fixture keeps turning up UNCERTAIN for items we think should be clean, the fix is to tighten memory-boundaries.md, not to tune the classifier prompt around a vague rule. That's a Holmes-owned brief if it happens. Not this task.

4. **`logSoulChange` status union extension is a data-migration concern, not code-only.** Any dashboards, CLIs, or review tooling that filters on status values needs to be aware of the new `filtered-technique` / `filtered-duplicate` values. Ryan should grep for all consumers of `ChangeLogStatus` (or whatever the union is called) before shipping and either extend them or explicitly document the omission.

5. **The eval fixture runs live Gemini, which means CI cost and flakiness.** Gating on `RUN_CLASSIFIER_EVAL=1` keeps day-to-day test runs fast. Nightly or on-merge-to-main is a reasonable CI cadence for the eval. That's Freddie's call, out of scope here.

6. **`hybridSearchChunks` from the original brief is unused in this task.** The cosine-similarity approach is dead; no brain.db vector search in the classifier path. If anyone sees a future PR trying to add a similarity prefilter "for efficiency," reject it — the evidence already says similarity isn't the right signal.

7. **Prompt hash should be logged in brain.db, not just the rule-doc hash.** The classifier wraps the taxonomy in a template. If the template changes but the rule doc doesn't, classification results may drift. Log the full prompt hash (template + taxonomy) so drift is detectable end-to-end. Added to the signature as `rulePromptHash` — name is slightly inaccurate; consider renaming to `promptHash` during implementation.
