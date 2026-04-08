---
id: ASL-0023
title: Rejection feedback loop gap + missing audit trail (degraded-quorum partially shipped in ASL-0024)
project-code: ASL
status: shipped
shipped: 2026-04-08
commits: [8442263, ae449e8]
priority: P1
created: 2026-04-08
refined: 2026-04-08T23:01+08:00
author: freddie
refined-by: mccall
depends-on: [ASL-0024]
blocks: [daemon-deployment]
related: [ASL-0018, ASL-0019, ASL-0020, ASL-0021, ASL-0024]
tags: [task, asl, distill, audit-trail, rejection-feedback, P1]
---

# ASL-0023 — Rejection feedback loop gap + missing audit trail

> **Status after ASL-0024 (commit `cecd73b`, 2026-04-08 22:24):**
> Bug 2 (degraded-quorum auto-apply) is now FIXED at the orchestrator level.
> `src/libs/autonomous-distill.ts:708` force-stages any proposal when
> `health.alive.length < 3`, emits `degraded_quorum:` frontmatter via
> `StagedProposal.degradedQuorum`, and `review-staged` renders it. The
> 2-alive Penny incident can no longer recur.
>
> What remains for ASL-0023: **Bug 1 (rejection feedback)**, **Bug 3 (audit
> trail)**, and a small residual on Bug 2 (explicit escape-hatch decision).
> Priority dropped from P0 → P1 because the most urgent failure mode (silent
> 2-alive auto-apply) is already neutralized. The daemon is still unsafe to
> run without Bug 3 — there is no audit trail for auto-applied soul changes
> — but the catastrophic "rejected content re-applied without review" path
> is closed.

## Three interlocking bugs, one root problem

On 2026-04-08 during the post-ASL-0017 review queue cleanup, three bugs surfaced together and produced a compounding failure mode that would make the daemon unsafe to run.

### Bug 1 — Rejections don't feed back into distill

The distill tool (`src/tools/distill-soul.ts`) regenerates candidate proposals from an agent's `knowledge/` files on every run. Nothing marks knowledge as "already-proposed-and-rejected." So the same bad content re-proposes on every sync.

**Observed example:** Earlier in the day, 12 Penny proposals were bulk-rejected for "evidence is a single knowledge file, not observed behavior from multiple sessions" (see ASL-0018). 30 minutes later, another asl-sync run produced 2 new Penny proposals whose content was functionally identical to 2 of the rejected ones — just paraphrased. The distill tool had no awareness that it was re-proposing rejected content.

**Why this matters:** Without feedback, the reject queue is Sisyphean. Every rejection is a one-shot block that the next distill run can undo by reshaping the same content. There is no "memory" of rejection in the pipeline.

> **ASL-0024 partial mitigation:** the knowledge-delta gate (Part A) prevents distill from re-firing when knowledge content hasn't changed. So the failure mode now requires: (a) someone touches a knowledge file AND (b) Gemini paraphrases a previously-rejected idea. That's still a live failure path — any consolidation run that edits knowledge can unblock rejected content on the next distill.

### Bug 2 — Degraded quorum auto-applies on 2/2 instead of staging **[FIXED in ASL-0024 Part B]**

~~The degraded-quorum code path (see `src/libs/judge-panel.ts` around the `degradedMode` flag) was designed as a safety net~~ — the ORCHESTRATOR now gates on `health.alive.length < 3` at `autonomous-distill.ts:708`, NOT on `judgePanelResult.degradedMode` (which remains pinned at `aliveCount < 2` per `judge-panel.ts:364` for ASL-0004 compatibility). The panel's `degradedMode` flag was intentionally left alone because ASL-0004 tests pin the 1-alive semantic. Answer: **yes, leave it alone, confirmed.**

What's left here is one open question — see **§Residual work for Bug 2** below.

### Bug 3 — Auto-applied changes leave no audit trail

After the Penny soul was modified by the auto-apply step, the corresponding staged proposal files did NOT land in `vault/studio/memory/penny/approved/`. In fact, there is no `approved/` directory under `penny/` at all — it was never created.

The only records that the 2 auto-applies ever happened:
1. The ephemeral sync log output (not persisted to disk beyond the terminal buffer)
2. The git diff on `.claude/souls/penny.md` (revealing WHAT changed but not WHY or WHEN)

Echo and Tala both have `approved/` dirs with records from manually-approved proposals. Auto-applied proposals go straight to the soul without ever creating that parent dir. This means **auto-applied changes are invisible to post-hoc review tooling**: `review-staged list` doesn't show them, `agent-state` doesn't surface them, and `git blame` on a soul line tells you the commit hash but not the proposal evidence, gates, or judge votes.

> **ASL-0024 did NOT address this.** The force-stage path in ASL-0024 writes to `staged/` (good — audit exists by default for degraded runs) but the normal 3-alive auto-apply path at `autonomous-distill.ts:729-774` still calls `_applySoulChange` and `logSoulChange` only — never `writeStagedProposal` to `approved/`. brain.db gets a `soul_changes` row, but the file-system audit trail is still missing.

## Why this is P1 (was P0)

Bug 2 was the acute P0 driver — silent auto-apply of content that had been rejected the same day. ASL-0024 closes that hole. The remaining bugs are still serious but slower-burning:

- **Bug 1** leaks content that humans have decided to exclude back into the pipeline, but ONLY when someone edits knowledge files between distill runs, AND only if Gemini paraphrases the excluded idea. With ASL-0024's delta gate, this is "once per knowledge edit" not "once per sync."
- **Bug 3** means auto-applied identity changes are invisible to `review-staged list` and hard to audit after the fact. The daemon is still unsafe to run without this — operator loses the "was my soul modified behind my back?" signal.

The daemon deployment is still blocked until Bug 3 ships. Bug 1 is a strong "fix before the next bulk-reject cycle," not an emergency.

## Goals

### Bug 1 — Rejection feedback

1. Rejected proposals must be tracked in a form the distill tool can consult.
2. Distill should skip candidate proposals whose content matches (embedding similarity or keyword overlap above a threshold) something in the rejected set for that agent.
3. The feedback check should be efficient enough to run on every distill invocation.
4. Rejection reason should be queryable so future distill runs can avoid the same failure mode (e.g., "don't propose anything whose evidence is a single knowledge file" after ASL-0018 is fixed the hard way).

### Bug 2 — Degraded quorum behavior **[LARGELY SHIPPED]**

1. ~~When `judgePanelResult.degradedMode === true`, the apply step MUST stage~~ — replaced by `health.alive.length < 3` orchestrator gate in ASL-0024 Part B.
2. ~~The staged file must clearly mark `degradedMode: true`~~ — shipped: `degraded_quorum: {alive, dead}` frontmatter.
3. ~~Auto-apply should require all 3 named judges to have voted approve~~ — shipped.
4. **STILL OPEN:** "If for operational reasons auto-apply must be allowed in degraded mode, that should be an explicit per-agent config flag with a scary name (e.g., `allow_degraded_auto_apply: true`) rather than the silent default." See §Residual work for Bug 2 — recommendation: **drop this goal.** Explicit fail-closed is correct per Principle 4. Re-add only if a specific operational need appears.

### Bug 3 — Audit trail

1. Every auto-applied proposal must be written to `{agent}/approved/` with the full original staged file (frontmatter, diff, gates, judges, evidence, source raw).
2. `approved/` directories must be created lazily on first write; never skipped.
3. `agent-state` CLI should expose a "recent auto-applies" column so the operator can see if the pipeline is modifying souls without review.
4. Optional: include auto-apply events in the `/morning` briefing so the operator sees them at session start.

---

## Refined spec (mccall, 2026-04-08)

### File inventory (verified against HEAD after ASL-0024)

| File | Lines touched | Why |
|------|--------------|-----|
| `src/libs/brain/schema.ts` | +1 table `rejectedProposals` (after `stagedProposals` at ~L211) | New durable rejection store |
| `src/libs/brain/index.ts` | +~25 lines in `initBrain()` after L341 | `CREATE TABLE IF NOT EXISTS rejected_proposals` + idempotent migration |
| `src/libs/brain/queries.ts` | +3 new functions (~60 lines) | `recordRejection`, `listRejections`, `findSimilarRejections` |
| `src/libs/brain/sync.ts` | Modify orphan cleanup at L744-752; add rejected/ scan ~L656 | Stop deleting rows on move to rejected/; keep them with `status=rejected` |
| `src/libs/autonomous-distill.ts` | Modify `applySoulChange` call path at L728-774; add helper `_archiveAutoApplied` | Bug 3: write audit record to `{agent}/approved/` on every auto-apply |
| `src/libs/staged.ts` | +1 export `writeApprovedArchive(proposal, decision)` (~40 lines) | Shared writer reused by both review-staged and auto-apply paths |
| `src/tools/distill-soul.ts` | Modify `callDistillGemini` L69-200; +1 helper `buildRejectionBlacklist` | Bug 1: inject rejection context into prompt |
| `src/libs/distill-cache.ts` | Bump `DISTILL_PROMPT_VERSION` from `v1-2026-04-07` to `v2-2026-04-09` | Prompt template changed — invalidate all caches |
| `src/tools/agent-state.ts` | +1 column "auto-applied (7d)" | Bug 3 goal #3 |
| `src/libs/__tests__/rejection-feedback.test.ts` | NEW | Bug 1 coverage |
| `src/libs/__tests__/auto-apply-audit.test.ts` | NEW | Bug 3 coverage |
| `src/libs/brain/__tests__/rejected-proposals.test.ts` | NEW | Schema + query coverage |

**Files explicitly NOT touched:**
- `src/libs/judge-panel.ts` — `degradedMode` semantic stays pinned at `aliveCount < 2` (ASL-0004 tests).
- `src/libs/agent-lifecycle.ts` — no new lifecycle columns needed; rejection state lives in its own table.
- `src/libs/gates.ts` — rejection feedback fires at the distill-prompt layer, not as a gate (fewer LLM calls).

### Bug 1 — implementation strategy

**Decision: brain.db table + prompt injection, NOT embedding similarity.** Rationale:
1. We already have a `staged_proposals` table with FTS5. Adding a `status='rejected'` filter is free.
2. Embeddings add a dependency on the Gemini embedding call per proposal, per distill run. Not worth it for a rejection count that is currently measured in dozens.
3. Prompt injection ("here are things you already tried, don't repeat") leverages Gemini's own semantic understanding — it's better than keyword matching AND better than cosine similarity because it reasons about intent.
4. Keyword/hash filtering as a *post-LLM* safety net catches the edge case where Gemini ignores the instruction.

**Two-layer defense:**

**Layer A — Prompt injection (primary, in `callDistillGemini`):**
Before building the distill prompt, query brain.db for rejected proposals for this agent within the last 90 days. For each, extract: `title`, `proposed_change` (first 300 chars), and `rejection_reason`. Inject a new prompt section just before `## Task`:

```
## Previously Rejected (do not re-propose)

The following proposals were rejected by a human reviewer. Do not propose
anything substantively similar to these — even if paraphrased or reframed.
If a knowledge file contains content matching one of these, explicitly skip it.

1. "Adhere to Vault Conventions" (rejected 2026-04-08): evidence was a single
   knowledge file, not observed behavior from multiple sessions
2. "LLM Output Integrity" (rejected 2026-04-08): same reason
...
```

Gemini must see the rejection reason so it can avoid the PATTERN, not just the wording.

**Layer B — Post-LLM hash filter (defense in depth, in `autonomousDistill` after `parseDistillOutput`):**
For each parsed proposal, compute a normalized-content hash (lowercase, strip punctuation, collapse whitespace — reuse the normalization from `memory-hash.ts`). Check against the set of rejected proposal hashes. If match → skip this proposal, emit `[autonomous-distill] filtered rejected duplicate: ${title}` to stderr, log a `soul_changes` row with status `filtered-rejected` for telemetry. No LLM call, no staging, no user-visible noise.

**Data model — `rejected_proposals` table:**

```typescript
// schema.ts — append after stagedProposals (L211)
export const rejectedProposals = sqliteTable("rejected_proposals", {
  id: integer("id").primaryKey({ autoIncrement: true }),
  agent: text("agent").notNull(),
  title: text("title").notNull(),
  proposedChange: text("proposed_change").notNull(),    // full content body
  section: text("section").notNull(),
  changeType: text("change_type").notNull(),
  evidence: text("evidence").notNull(),
  rejectionReason: text("rejection_reason"),            // from recordDecision; nullable
  normalizedHash: text("normalized_hash").notNull(),    // sha256 of normalized proposedChange, 16 hex chars
  sourcePath: text("source_path").notNull().unique(),   // memory/{agent}/rejected/{file}.md
  rejectedAt: text("rejected_at").notNull(),            // ISO-8601 — when review-staged reject fired
  createdAt: text("created_at").notNull(),              // row create ts
});
```

Indexes: `(agent, rejected_at DESC)` for the prompt-injection query; `(agent, normalized_hash)` for the Layer B lookup.

**Where the rejection data gets written:**

Two code paths populate `rejected_proposals`:

1. **`src/tools/review-staged.ts` `applyReject` (L644-692):** After `recordDecision(newPath, ...)` succeeds, call a new `recordRejection({ agent, sf, reason, timestamp })` helper from `brain/queries.ts`. This inserts/upserts into `rejected_proposals`. Replaces the current `DELETE FROM staged_proposals` behavior that `syncStagedProposals` does on orphan cleanup (which was silently discarding the rejection).

2. **`src/libs/brain/sync.ts` `syncStagedProposals` (L744-752):** Stop deleting rows whose source file moved to `rejected/`. Instead, detect the move (file gone from `staged/` but present in `rejected/` under same filename) and call `recordRejection` as a backfill path. This handles the existing ~12 rejected Penny proposals without requiring re-rejection.

**Human override — "unreject" command:**

New subcommand: `bun run tool review-staged unreject <id>`. Removes the row from `rejected_proposals` but leaves the file in `rejected/` (operator can manually delete if desired). Rationale: if an operator rejected something yesterday and now wants distill to re-propose it, one CLI call removes the block. Keeping the file is for audit — the rejection event happened, we just no longer want it to block future proposals.

Add a one-line warning to `review-staged list`: if the filter-count for this agent ≥ 3 on the last sync, show `(3 rejected proposals were filtered from this run)` so the operator knows the suppression is working.

**Time window:**

Rejection entries are **permanent by default**, but the prompt-injection layer only includes rejections from the last 90 days. Rationale:
- Permanent on disk: audit trail is forever.
- 90-day prompt window: Gemini context budget. 50 rejections × 300 chars = 15 KB; a year of rejections at 2/day rate is ~60 KB which starts eating the 32k input budget.
- Hash filter (Layer B) consults ALL rejections regardless of age — cheap, no LLM cost.
- If operator wants older rejections surfaced, they can bump the window via env var `ASL_REJECTION_WINDOW_DAYS` (default 90).

### Bug 2 — residual work

**Recommendation: close the escape-hatch goal. Do not ship `allow_degraded_auto_apply`.**

Rationale:
1. Principle 4 says degraded modes must fail closed. A per-agent override is still a silent permissive fallback — the operator sets it once, forgets, and the next minimax outage auto-applies again.
2. The 1-alive path and 2-alive path ARE already unified where it matters: both write `degraded_quorum:` frontmatter OR the panel flag, both surface as `(degraded mode)` in `review-staged`, both force-stage. The only unification gap is that `judgeResults.degradedMode` fires at `aliveCount < 2` while the orchestrator gate fires at `alive.length < 3`. This is intentional (ASL-0004 pin) and correct in effect — the gate runs BEFORE the panel flag is consulted so the 2-alive case never reaches the panel-degradedMode check.
3. If operators need to push through a soul change during a judge outage, the correct path is: `review-staged approve <id> --reason "..." --yes`. That's the documented human override. It leaves a proper audit trail and requires explicit intent.

**Additional test coverage on top of `autonomous-distill-gate.test.ts`:**

- Test that 3-alive + all-approve path does NOT write `degraded_quorum:` frontmatter. (Sanity — current coverage exists, verify.)
- Test that 2-alive + all-approve path DOES write `degraded_quorum: {alive: 2, dead: ["minimax"]}`. (Exists in ASL-0024.)
- Test that 1-alive + approve path writes EITHER the panel flag OR the orchestrator frontmatter but not double-labels in `review-staged` output. (Exists in review-staged.test.ts per ASL-0024 commit message.)

**No new test work required for Bug 2.** Mark it closed in the refinement notes.

### Bug 3 — audit trail implementation

**The auto-apply path lives at `autonomous-distill.ts:729-774`.** Specifically:

- L730: `_applySoulChange(agent, soulPath, proposal)` — mutates the soul file on disk.
- L752-761: `logSoulChange(...)` — writes a brain.db `soul_changes` row.
- **No file-system audit record is written anywhere.**

**Fix: new helper `writeApprovedArchive` in `staged.ts`.** Called from `autonomous-distill.ts` immediately after `_applySoulChange` succeeds and BEFORE the `logSoulChange` call. Same content format as a manually-approved staged file (i.e., what `writeStagedProposal` produces, then `recordDecision` overlays with approved status). Single atomic write, not a write-then-rename.

**New function signature (`src/libs/staged.ts`):**

```typescript
/**
 * Write a full audit record for an auto-applied proposal directly to
 * {agent}/approved/. Creates the dir lazily on first write. Same frontmatter
 * schema as writeStagedProposal, but status="approved" from the start and
 * the ## Decision section is pre-filled with "Decided by: autonomous pipeline".
 *
 * Returns the relative path: memory/{agent}/approved/{filename}
 *
 * Called exclusively by autonomous-distill.ts after _applySoulChange succeeds.
 * review-staged's applyApprove continues to use writeStagedProposal→renameSync
 * because its source file already exists in staged/.
 */
export function writeApprovedArchive(
  proposal: StagedProposal,
  appliedAt: string,      // ISO-8601
): string;
```

**Filename convention:** `{YYYYMMDD}-{HHmmss}-{slug}.md` — identical to `writeStagedProposal`. Prefix matches the `_applySoulChange` timestamp so filesystem-sorted order matches apply order.

**Content format (bytes on disk):** copy the `writeStagedProposal` template verbatim, but:
- `status: approved` in frontmatter
- Add `applied_by: autonomous-pipeline` in frontmatter
- Add `applied_at: ${appliedAt}` in frontmatter
- `## Decision` section pre-filled:
  ```
  ## Decision

  Status: approved
  Decided at: ${appliedAt}
  Decided by: autonomous pipeline
  Judge panel: ${judgeResults.votes.map(v => `${v.model}:${v.vote}`).join(", ")}
  Applied via: autonomousDistill
  ```

**Insertion point in `autonomous-distill.ts`:**

```typescript
// Line 730 currently:
try {
  _applySoulChange(agent, soulPath, proposal);
} catch (err: any) { ... }

// NEW — immediately after the try/catch, before logSoulChange:
const appliedAt = new Date().toISOString();
const auditProposal: StagedProposal = {
  agent,
  date: today,
  title: proposal.title,
  proposedChange: proposal.content,
  section: proposal.section,
  changeType: proposal.type,
  evidence: proposal.evidence,
  gateResults,
  judgeResults,
  reason: `Auto-applied: unanimous approval (${judgeResults.votes.map(v => v.model).join(", ")})`,
  status: "approved",
  degradedQuorum: undefined,  // auto-apply path never runs when degraded
};
try {
  writeApprovedArchive(auditProposal, appliedAt);
} catch (auditErr) {
  // PRINCIPLE 5: audit failure is FATAL for the auto-apply, not non-fatal.
  // Revert the soul change and stage instead.
  process.stderr.write(
    `[autonomous-distill] FATAL: audit trail write failed for ${agent}/${proposal.title}: ${auditErr.message}. Reverting soul change.\n`
  );
  // Re-read the original (pre-apply) soul — we saved it as currentSoul at L572
  writeFileSync(soulPath, currentSoul, "utf-8");
  await _stageProposal(agent, proposal, gateResults, judgeResults,
    `Audit trail write failed: ${auditErr.message}`);
  // Then continue to the outcomes.push as "staged"
  outcomes.push({ ... action: "staged" ... });
  continue;
}
```

**Why the revert-and-stage on audit failure:** Principle 5 (from `PRINCIPLES.md`): *"no soul modification without a matching persistent record. If the record can't be written (disk full, permissions, whatever), the modification must fail and log loudly — not proceed silently."* The current ASL-0024 path logs brain.db failures as non-fatal. That's acceptable because brain.db is secondary storage; the file-system audit is the primary record. A missing audit record with a modified soul is exactly the Bug 3 failure mode.

**Dir creation:** lazy, via `mkdirSync(approvedDir, { recursive: true })` at the top of `writeApprovedArchive`. No bootstrap needed.

**Retroactive fix for existing auto-applied-without-audit proposals:**

**Scope: OUT.** The 3 Tala proposals the user mentioned were committed to `.claude/souls/tala.md` on earlier sync runs. We cannot reconstruct their gate results, judge votes, or source raw from git history. Writing synthetic audit records would be lying about the audit trail. Document them in the commit body as "known blind spots" and move on. The fix forward-dates from deployment.

### Test shapes

**`src/libs/brain/__tests__/rejected-proposals.test.ts`:**
1. insert rejection → query by agent returns the row
2. insert same `normalized_hash` twice → second insert is idempotent (upsert-by-hash)
3. `findSimilarRejections(agent, "some content")` returns matches when normalized hash collides
4. 90-day window filter excludes rejections older than threshold
5. unreject removes the row without affecting the file on disk

**`src/libs/__tests__/rejection-feedback.test.ts`:**
1. distill prompt contains "Previously Rejected" section when agent has rejections
2. distill prompt has NO "Previously Rejected" section when agent has zero rejections (don't pollute the prompt)
3. `DISTILL_PROMPT_VERSION` changed — cached distill results from v1 are invalidated
4. Layer B hash filter: proposal whose normalized content matches a rejection is dropped before staging
5. Layer B: filter event writes a `soul_changes` row with `status='filtered-rejected'`
6. unrejected proposal (removed from `rejected_proposals`) is no longer injected into prompt
7. End-to-end: reject a proposal → re-run distill with same knowledge → parse returns zero proposals

**`src/libs/__tests__/auto-apply-audit.test.ts`:**
1. successful auto-apply writes a file to `{agent}/approved/` with all frontmatter fields present
2. approved/ dir created lazily on first auto-apply for an agent with no prior approved history
3. audit file content includes gate results, judge votes, evidence
4. `writeApprovedArchive` throwing causes the soul file to be reverted AND the proposal to be staged with reason "Audit trail write failed"
5. after audit failure + revert, the outcome in `PipelineResult.agents[].proposals[]` has `action: "staged"` not `"auto-applied"`
6. degraded-quorum force-stage path (ASL-0024) does NOT call `writeApprovedArchive` (because it stages, doesn't apply)
7. `agent-state` column "auto-applied (7d)" counts files in approved/ within 7 days modulo mtime

### Acceptance criteria (Ryan checklist)

#### Bug 1 — rejection feedback

- [ ] `rejected_proposals` table exists and is created idempotently in `initBrain`
- [ ] `recordRejection` called from `review-staged.applyReject` after successful move
- [ ] `syncStagedProposals` no longer deletes rows for files that moved to `rejected/`; calls `recordRejection` as backfill
- [ ] `buildRejectionBlacklist(agent, windowDays=90)` helper returns formatted prompt block
- [ ] `callDistillGemini` injects the blacklist when rejections exist; does NOT inject empty header
- [ ] `DISTILL_PROMPT_VERSION` bumped to `v2-2026-04-09` — all existing caches invalidated
- [ ] Post-LLM hash filter in `autonomousDistill` drops proposals whose normalized hash matches a rejection
- [ ] Filter emits `[autonomous-distill] filtered rejected duplicate: ${title}` to stderr
- [ ] Filter logs `soul_changes` row with status `filtered-rejected`
- [ ] `review-staged unreject <id>` removes `rejected_proposals` row, leaves file on disk
- [ ] `review-staged list` shows filtered-count banner when > 0 for the current sync
- [ ] Env var `ASL_REJECTION_WINDOW_DAYS` overrides the 90-day prompt window
- [ ] Backfill path populates `rejected_proposals` for existing `memory/*/rejected/*.md` on first `brain --check` after deploy
- [ ] Test file `rejection-feedback.test.ts` all tests green
- [ ] Test file `rejected-proposals.test.ts` all tests green

#### Bug 2 — residual (mostly non-work)

- [ ] No change to `judge-panel.ts` (degradedMode semantic stays pinned)
- [ ] No `allow_degraded_auto_apply` flag shipped (closed as WONTFIX — Principle 4)
- [ ] Sanity test added: 3-alive + unanimous approve path does NOT emit `degraded_quorum:` frontmatter
- [ ] Document the Principle 4 rationale in a comment at `autonomous-distill.ts:708`

#### Bug 3 — audit trail

- [ ] New `writeApprovedArchive(proposal, appliedAt)` exported from `staged.ts`
- [ ] Called from `autonomousDistill` immediately after `_applySoulChange` succeeds
- [ ] Audit failure = revert soul + stage + continue (NOT non-fatal continue)
- [ ] Frontmatter fields: `status: approved`, `applied_by: autonomous-pipeline`, `applied_at: <iso>`
- [ ] `## Decision` section pre-filled with judge votes
- [ ] `approved/` dir created lazily via `mkdirSync(..., { recursive: true })`
- [ ] `agent-state` CLI shows `auto-applied (7d)` column counting files in approved/ by mtime
- [ ] The 3 unattributed pre-existing Tala auto-applies are noted in commit body as "known blind spots"
- [ ] Test file `auto-apply-audit.test.ts` all tests green
- [ ] `bun test` full suite: no regressions vs the 662-pass baseline from ASL-0024

### Dependency order (ship in this sequence)

1. **Bug 3 first (audit trail).** Smallest, most urgent for daemon deployment, zero dependencies on the other bugs. Ships the fail-closed invariant required by Principle 5. Without this, the daemon is still unsafe.
2. **Bug 1 second (rejection feedback).** Depends on nothing, but touches more surface area (prompt, schema, sync, review-staged, distill cache). Ship after Bug 3 so the audit trail is in place before we start relying on it for "what was rejected when."
3. **Bug 2 residual (docs + sanity test).** Trivial, can piggyback on either of the above commits or ship as a doc-only PR. No blocker.

**Rationale for this order:** Bug 3 is a pure safety invariant — it doesn't change what the pipeline proposes, it just ensures what the pipeline APPLIES is visible afterward. Bug 1 changes pipeline behavior (fewer duplicates), which is more subtle and benefits from having Bug 3 in place for post-hoc investigation when Bug 1's heuristics misfire. Bug 2 is paperwork.

**Can they ship independently?** Yes — each bug's code paths are disjoint. But the ORDER matters because the testing story for Bug 1 becomes cleaner when Bug 3's audit file is there to assert against in end-to-end tests.

## Investigation path (historical)

1. Read `src/libs/autonomous-distill.ts` — find where the "degraded quorum" branch lives and see how it reaches the apply call. Verify whether there's any `if (degradedMode) { stage } else { apply }` gate, or whether degraded just logs and proceeds.
2. Read `src/libs/apply-soul-change.ts` (or wherever apply lives) — see whether it writes to `approved/` and under what conditions.
3. Read `src/tools/distill-soul.ts` — see whether it has any "exclude recently rejected" logic. Expected answer: no.
4. `rg -n "approved/" src/` — find every write site and confirm they all follow the same pattern.
5. Check whether `vault/studio/memory/penny/approved/` ever existed in git history (it's gitignored under `vault/` so there's no git history to check — but there may be local traces).
6. Look at the decision log in brain.db — do rejected proposals appear there? Can distill query them?

**Investigation results (mccall, 2026-04-08):**

- `applySoulChange` lives in `autonomous-distill.ts:278`, not a separate file. It does NOT write to `approved/`. Confirmed.
- `callDistillGemini` at `distill-soul.ts:69-200` has zero rejection awareness. The prompt is hardcoded; only `soulContent`, `agentKnowledge`, `sharedKnowledge` are injected.
- `syncStagedProposals` at `brain/sync.ts:636` walks ONLY `{agent}/staged/` dirs. On orphan cleanup (L744-752) it DELETES the staged_proposals row when a file moves to `rejected/` — actively discarding the rejection signal. This is the root cause of "brain.db has no rejection memory."
- `staged_proposals` table has FTS5 + embeddings (`embedding` column at schema L206). We could reuse it instead of creating `rejected_proposals`, but the semantics are confused — status transitions would need a migration, and the "rejected" rows would need a separate lifecycle. Cleaner to have a dedicated table.

## Immediate mitigation (while the fix is being built)

1. **Do NOT start the ASL daemon** until Bug 3 is fixed.
2. **Run asl-sync only when the operator is present** and ready to `git diff .claude/souls/` immediately after.
3. **Treat every asl-sync run as suspect** — assume the soul files may have been modified without staging.
4. **Document the manual revert recipe** in case auto-applies happen again: `git checkout .claude/souls/{agent}.md` restores to the last committed version.

> Post-ASL-0024: manual revert is still the recipe. The daemon is still blocked. But the acute 2-alive auto-apply failure mode is closed — running `asl-sync` manually during a minimax outage will now force-stage instead of apply.

## Not in scope

- Fixing minimax stability (ASL-0020 — separate task).
- Fixing Penny distill evidence quality (ASL-0018).
- Fixing Sol distill redundancy (ASL-0019).
- Fixing the refinement apply logic (ASL-0021).
- Redesigning the judge panel architecture.
- Retroactively reconstructing audit trails for the 3 pre-existing Tala auto-applies.
- Embedding-based rejection similarity (explicitly declined in favor of prompt injection + hash filter).
- Per-agent `allow_degraded_auto_apply` flag (declined per Principle 4).

## Context

- Surfaced during the 2026-04-08 review queue cleanup session.
- The Penny auto-apply was reverted manually via `git checkout .claude/souls/penny.md` immediately after being noticed.
- The 2 auto-applied Penny proposal contents were:
  1. "Always respects the established vault structure and templates..." — functional equivalent of rejected "Adhere to Vault Conventions"
  2. "Vigilantly guards against silent data loss from LLM token limits..." — functional equivalent of rejected "LLM Output Integrity"
- Both of these were approved by gemini AND grok in the degraded-quorum run, despite gemini having rejected functionally equivalent content earlier the same day.
- The "gemini flipped its vote" observation is itself a data point worth investigating — either judge verdicts are noisy across paraphrased content, or the evidence was reframed enough to pass the bar. Either way it reinforces that rejection feedback is load-bearing.
- ASL-0024 shipped the fail-closed degraded quorum gate on 2026-04-08 at 22:24 PHT (commit `cecd73b`), closing the acute P0 driver for this task. Priority dropped to P1 as a result.
