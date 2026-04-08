---
id: ASL-0024
title: Gate distill on knowledge-content delta + force-stage in degraded quorum
project-code: ASL
status: shipped
shipped: 2026-04-08
commits: [cecd73b]
priority: P0
created: 2026-04-08
author: freddie
depends-on: []
blocks: [daemon-deployment]
related: [ASL-0023, ASL-0018, ASL-0019, ASL-0020, ASL-0021]
principles: [1, 2, 3, 4]
tags: [task, asl, distill, trigger, degraded-quorum, P0]
---

# ASL-0024 — Gate distill on knowledge-content delta + force-stage in degraded quorum

## TL;DR

Two coupled fixes that together stop the review queue from growing faster than the human can process it, and stop auto-apply from bypassing human review in degraded mode. Together they neutralize the worst symptoms of the 2026-04-08 review queue incident without having to fix the five individual ASL bugs (0018, 0019, 0020, 0021, 0023) first.

See `vault/studio/projects/autonomous-self-learning/PRINCIPLES.md` — this task implements Principles 3 and 4 directly, and is the first concrete use case of the principles document.

## Part A — Knowledge-delta gate for distill (Principle 3)

### Problem

Distill currently fires whenever its knowledge input is judged "stale" by a mechanism that does not check content. Observed trigger paths include cache misses, mtime changes, and hash-of-the-directory checks that are sensitive to whitespace or metadata touches. As a result, distill regenerates proposals from **unchanged knowledge content** on every sync run.

**Concrete incident (2026-04-08):** Sol and Tala distill fired on empty inboxes during a sync run because consolidation had touched their knowledge dirs' mtimes without changing content. The run produced 14 Sol proposals and 2 Tala proposals, plus 3 "auto-applied" entries that were no-ops (same mechanism). None of the 16 were genuine new learnings — they were re-syntheses of existing knowledge.

### Fix

Before running distill for an agent, compute a **content hash** of the merged knowledge state and compare against a stored last-successful-distill hash.

**Content hash spec:**
- Input: all `vault/studio/memory/{agent}/knowledge/*.md` files, concatenated in filename-sorted order. Top-level only (no recursion) — matches the convention used by `distill-cache.ts` and `agent-lifecycle.ts`.
- Normalize before hashing:
  - Strip the YAML frontmatter block entirely (`/^---\r?\n[\s\S]*?\r?\n---\r?\n?/`).
  - Convert `\r\n` → `\n`.
  - Strip trailing whitespace from each line.
  - Collapse runs of whitespace (inside lines) to a single space.
  - Trim leading/trailing whitespace from the resulting file body.
- Between files, inject a single record separator (`\n---FILE:${filename}---\n`) so that moving content across files still changes the hash (filename boundaries are semantically meaningful).
- Hash: SHA-256 of the normalized concatenated bytes. Return first 16 hex chars.
- Shared knowledge is **NOT** included in this hash. This is the intentional difference from `computeDistillInputHash` in `agent-lifecycle.ts` (which is a daemon staleness signal, not a gate). Shared-knowledge changes are out of scope for ASL-0024's gate.

**Storage:**
- A new column `last_distilled_knowledge_hash TEXT` on `agent_lifecycle`. See the Migration section below for why we do NOT reuse `last_distill_input_hash`.
- On successful distill completion (after the LLM call returns, before the gate/judge/apply loop), write the current hash to this column.
- On distill entry, read the stored hash, compute the current hash, compare. If equal → skip with reason `"knowledge content unchanged since last distill"`.

**Behavior changes:**
| Scenario | Before | After |
|---|---|---|
| Empty inbox, consolidate skipped, distill runs on same knowledge | FIRES, regenerates | SKIPS |
| New inbox items consolidate into existing knowledge (dedupe hit), no content change | FIRES | SKIPS |
| New inbox items consolidate into existing knowledge with a real content update | FIRES | FIRES (correct) |
| New knowledge file appears | FIRES | FIRES (correct) |
| Whitespace-only touch on knowledge file (mtime change) | FIRES | SKIPS (hash identical) |
| Initial run for new agent, no stored hash | FIRES | FIRES, stores hash |

### What this does NOT fix

- The distilled proposals themselves may still be bad quality (ASL-0018 Penny evidence, ASL-0019 Sol redundancy).
- Rejected proposals don't feed back into distill (ASL-0023 bug 1).
- Refinement semantics in apply (ASL-0021).

But these issues no longer **fire continuously**. They fire only when knowledge actually changes, which is rare in steady state.

## Part B — Force-stage in degraded quorum (Principle 4)

### Problem

The judge panel currently has a "degraded quorum" code path that logs `DEGRADED QUORUM: running with 2 alive judges (skipped: minimax — api)` and then continues as if the panel had returned a valid unanimous verdict. If both alive judges vote approve, the proposal is auto-applied. This is a fail-open fallback.

**Concrete incident (2026-04-08):** During an asl-sync run with minimax dead, Penny's distill produced 2 proposals that gemini and grok both approved. The pipeline auto-applied them directly to `.claude/souls/penny.md`, bypassing human review. The content was functionally equivalent to 2 proposals that had been rejected earlier the same day. This incident is the primary impetus for ASL-0023 (which is broader) and this part of ASL-0024 (which is the minimum viable fix).

### Important semantic trap discovered in code review

**`JudgePanelResult.degradedMode` does NOT mean what the task title suggests.** In `src/libs/judge-panel.ts:364`:

```ts
const degradedMode = aliveCount < 2;            // true ONLY when 1 alive
const approved = unanimous && aliveCount >= 2;  // auto-apply requires >= 2 alive
```

So with 2 judges alive (minimax dead) and both approving, `degradedMode === false` and `approved === true`. The Penny incident happened through a code path where `degradedMode` was **never set true**. Checking `if (judgeResult.degradedMode)` in the orchestrator does **not** catch the 2-alive case.

**Do NOT redefine `degradedMode` in judge-panel.ts.** The ASL-0004 lifecycle tests and the judge-panel tests pin the 1-alive semantic; flipping it has unknown blast radius. Instead, the orchestrator must gate on `health.alive.length < 3` (the pipeline-level health check already computed at line 408 of `autonomous-distill.ts`) — this is the authoritative "all three judges ran" signal and is independent of the panel's own `degradedMode` flag.

### Fix

At the point in `autonomous-distill.ts` where the judges have returned `approved === true` and the orchestrator is about to call `applySoulChange`, add a guard:

```
if (health.alive.length < 3) {
    force-stage with reason "degraded quorum: only N of 3 judges alive (dead: ...)"
    continue;
}
```

The guard fires for both the 1-alive (where the panel already refused to approve) and the 2-alive (where it did). The 3-alive path is unchanged. Write the proposal to `staged/` via the existing `_stageProposal` helper (already in the file at line 702) and inject `degradedMode: true` into the staged frontmatter so `review-staged list` shows it in the queue.

**Staged-file frontmatter:** `staged.ts:322` currently writes a fixed frontmatter block that does NOT include a `degraded_mode` / `degradedMode` key. The body-level `## Judge Results` section already prints `degradedMode: true` for the 1-alive case (`formatJudgeResults` at staged.ts:165), and the parser `parseJudgeResultsTable` (staged.ts:259) reads it back. BUT that only works for the 1-alive case because the panel sets the flag. For the 2-alive case we need to:

1. Extend `StagedProposal` with an optional `degradedQuorum?: { alive: number; dead: string[] }` field.
2. Extend `writeStagedProposal` to emit a `degraded_quorum:` frontmatter key (JSON-escaped) when the field is present.
3. Extend `formatJudgeResults` (or a new section writer) to print the alive/dead breakdown in the body regardless of `judgeResults.degradedMode`.
4. Extend `review-staged.ts:413` to surface the degraded-quorum note from either source (panel `degradedMode` OR the orchestrator-supplied `degraded_quorum` frontmatter), so operators see "(degraded mode)" on every quorum-gated staging.

The `review-staged` CLI already surfaces `degradedMode` in its list output (ASL-0012), so once frontmatter is written the operator will see these clearly marked in their queue.

### What this does NOT fix

- The fact that `degradedMode` happens in the first place (ASL-0020 — minimax stability).
- The missing audit trail for past auto-applies (ASL-0023 bug 3).
- Cases where the panel runs at full strength but 2/3 judges approve and 1 dissents (that's still a legitimate 2-1 reject, not a degraded path).

But it does guarantee that **while any judge is down, no soul modification bypasses human review**, which is the urgent concern.

## Implementation plan

### Files to touch (verified against the actual codebase)

#### 1. `src/libs/memory-hash.ts` — NEW FILE

Pure helper. No imports from brain.db. Unit-testable with tmp dirs.

Exports:

```ts
/**
 * Compute a normalized content hash of an agent's knowledge dir.
 * Used by the ASL-0024 distill delta gate.
 *
 * Normalization rules (see task doc Part A):
 *   1. List vault/studio/memory/{agent}/knowledge/*.md (top level only), sort lex.
 *   2. For each file:
 *      a. Read UTF-8
 *      b. Strip YAML frontmatter block (/^---\r?\n[\s\S]*?\r?\n---\r?\n?/)
 *      c. Normalize line endings (\r\n → \n)
 *      d. Strip trailing whitespace per line
 *      e. Collapse inline whitespace runs to single space
 *      f. Trim leading/trailing whitespace
 *   3. Join files with "\n---FILE:${filename}---\n" separator.
 *   4. SHA-256, first 16 hex chars.
 *
 * Returns EMPTY_CONTENT_HASH ("empty0000000000000") if the dir is missing or
 * contains zero .md files — stable sentinel, same rationale as EMPTY_KNOWLEDGE_HASH
 * in agent-lifecycle.ts.
 *
 * Synchronous. No async I/O needed — we only touch a handful of small files.
 */
export function computeKnowledgeContentHash(agent: string): string;

/** Exported for tests. Takes a raw dir path instead of an agent name. */
export function _computeKnowledgeContentHashForDir(dir: string): string;

/** Exported for tests. Normalizes one file body the same way the hasher does. */
export function _normalizeKnowledgeFileBody(raw: string): string;

export const EMPTY_CONTENT_HASH: string; // "empty0000000000" (16 chars) or sha256("empty").slice(0,16)
```

Notes:
- Sync, not async. `Promise<string>` is unnecessary — all I/O is `readFileSync` on a tiny number of files, matching the existing `hashKnowledgeDir` in `agent-lifecycle.ts:128`.
- Keep this library pure: no brain.db imports, no logger, no paths beyond `fromRoot`. Ryan can import it from both `autonomous-distill.ts` and the daemon later without circular-dep risk.

#### 2. `src/libs/brain/schema.ts` — ADD ONE COLUMN

Current `agentLifecycle` table is at **lines 234–247**. Insert between `lastDistillInputHash` (line 240) and `soulVersionHash` (line 241):

```ts
lastDistilledKnowledgeHash: text("last_distilled_knowledge_hash"),
```

**Why a new column (not reuse of `last_distill_input_hash`):** The existing column is populated by `computeDistillInputHash` in `agent-lifecycle.ts:169`, which is a **composite of raw-bytes hashes of agent knowledge + shared knowledge**, with no normalization. Its semantics are "anything touched that would invalidate the daemon's staleness signal" — intentionally oversensitive because it drives the daemon's "should I even consider distilling?" check. ASL-0024's gate has opposite semantics: "did the *meaningful content* change?" Reusing the column would either (a) break the ASL-0004 staleness tests that pin the raw-bytes semantics, or (b) make the gate under-trigger on shared-knowledge changes it should honor. Two columns, two semantics, zero coupling.

#### 3. `src/libs/brain/index.ts` — MIGRATION

The `CREATE TABLE IF NOT EXISTS agent_lifecycle` block is at **lines 311–326**. Add `last_distilled_knowledge_hash TEXT,` between line 318 (`last_distill_input_hash TEXT,`) and line 319 (`soul_version_hash TEXT,`).

Then, because existing databases already have the table, add a migration block **immediately after line 326** (the closing `);` of the CREATE TABLE), following the pattern already used for `staged_proposals.title` at lines 330–339:

```ts
// ASL-0024: add last_distilled_knowledge_hash column if missing
try {
  const cols = raw.prepare("PRAGMA table_info(agent_lifecycle)").all() as { name: string }[];
  const hasCol = cols.some(c => c.name === "last_distilled_knowledge_hash");
  if (!hasCol) {
    raw.exec("ALTER TABLE agent_lifecycle ADD COLUMN last_distilled_knowledge_hash TEXT");
  }
} catch {
  // table doesn't exist yet — CREATE TABLE above handles fresh DBs
}
```

**Backfill:** None. The column is nullable. Existing agents will see `null` on first read, which the gate treats as "no prior distill hash → fire and store." That's the same first-run path as a brand-new agent.

#### 4. `src/libs/agent-lifecycle.ts` — ADD TYPED ACCESSORS

Current `AgentLifecycle` interface is at **lines 60–73**. Add one field (preserve existing field order — insert between `lastDistillInputHash` at line 66 and `soulVersionHash` at line 67):

```ts
lastDistilledKnowledgeHash: string | null;
```

Then add two new public functions after `recordDistilled` (current line 368). Do NOT modify `recordDistilled` itself — it's load-bearing for ASL-0004 tests and the daemon.

```ts
/**
 * Read the last-successful distill knowledge-content hash for an agent.
 * Returns null if no row exists OR the column is null (first distill).
 *
 * ASL-0024 gate reads this, compares against computeKnowledgeContentHash(),
 * and skips distill if they match.
 */
export function getLastDistilledKnowledgeHash(agent: string): string | null;

/**
 * Store the current normalized knowledge-content hash as the marker of the
 * last successful distill. Called by autonomous-distill AFTER the per-agent
 * distill+gate+judge+apply loop completes without error — the marker is only
 * advanced when the current content has actually been fully processed.
 *
 * Does NOT touch last_distill_input_hash, soul_version_hash, or any other
 * lifecycle field. Single-column update.
 */
export function setLastDistilledKnowledgeHash(agent: string, hash: string): void;
```

Both functions are thin wrappers over `getAgentLifecycle` / `upsertAgentLifecycle` with a one-field payload. The upsert path already handles row creation.

#### 5. `src/libs/autonomous-distill.ts` — GATE + FORCE-STAGE

**Insertion point A — knowledge-delta gate (Part A):**

The agent loop starts at **line 438** (`for (const agent of targetAgents)`). The soul-existence check runs at 442. The first `await import("../tools/distill-soul.js")` dynamic import is inside the `try` block at 453–454. The actual LLM call is `callDistillGemini(agent)` at **line 466**.

Insert the gate **between line 450** (end of soul-existence check) **and line 452** (`let rawDistillOutput: string;`). Before any import or LLM call:

```ts
// ── ASL-0024 Part A: knowledge-content delta gate ─────────────────────────
// Skip distill entirely if the agent's normalized knowledge content hasn't
// changed since the last successful distill. This fires BEFORE the Gemini
// call so we save both quota and review-queue volume.
const currentKnowledgeHash = computeKnowledgeContentHash(agent);
const lastKnowledgeHash = getLastDistilledKnowledgeHash(agent);
if (lastKnowledgeHash !== null && lastKnowledgeHash === currentKnowledgeHash) {
  agentResults.push({
    agent,
    skipped: true,
    skipReason: "knowledge content unchanged since last distill",
    proposals: [],
  });
  continue;
}
```

**Insertion point B — hash commit (Part A follow-through):**

After the `for (const proposal of parsedProposals)` loop completes for this agent (i.e. **immediately before line 687**, `agentResults.push({ agent, skipped: false, proposals: outcomes })`), call:

```ts
// ASL-0024: mark this content state as distilled. Only runs on clean exit
// of the proposal loop — any early throw leaves the hash unchanged so the
// next run re-fires.
try {
  setLastDistilledKnowledgeHash(agent, currentKnowledgeHash);
} catch {
  // DB write failure is non-fatal — the next run will re-fire, which is
  // the safe default (re-distill vs. silently skip on stale hash).
}
```

**Insertion point C — degraded-quorum force-stage (Part B):**

The judge panel result is checked at **line 610** (`if (!judgeResults.approved)`) — rejection path. The auto-apply path starts at **line 638** (`// Judges approved — apply incrementally`) and calls `applySoulChange(agent, soulPath, proposal)` at **line 640**.

Insert the force-stage branch **between line 636** (`continue;` that closes the rejection `if` block) **and line 638** (the `// Judges approved` comment):

```ts
// ── ASL-0024 Part B: degraded quorum → force stage, never auto-apply ──────
// Even if the judge panel returned approved=true, if fewer than 3 judges
// were alive for this run the verdict is not authoritative enough for an
// unattended soul mutation. Gate on health.alive.length (the pipeline-level
// health check at line 408), NOT on judgeResults.degradedMode — the panel's
// flag is true ONLY for the 1-alive case (see judge-panel.ts:364), but the
// 2026-04-08 Penny incident happened in the 2-alive case.
if (health.alive.length < 3) {
  const deadList = health.dead.map(d => `${d.model} (${d.errorType})`).join(", ");
  const reason = `degraded quorum: only ${health.alive.length} of 3 judges alive (dead: ${deadList || "none"})`;
  await _stageProposal(agent, proposal, gateResults, judgeResults, reason);
  outcomes.push({
    agent,
    proposalTitle: proposal.title,
    proposedChange: gateInput.proposedDiff[0],
    gateResults,
    judgeResults,
    action: "staged",
    reason,
  });
  continue;
}
```

Ordering matters: this check fires AFTER the rejection path (so a 2-alive unanimous-reject still logs a normal "Judges rejected" outcome, not a "degraded quorum" outcome) and BEFORE the apply call (so no apply happens under degraded conditions).

#### 6. `src/libs/staged.ts` — FRONTMATTER DEGRADED FLAG

Current `StagedProposal` interface is at **lines 19–31**. Add one optional field (does not break existing callers):

```ts
degradedQuorum?: {
  alive: number;
  dead: string[];  // e.g. ["minimax"]
};
```

Current `writeStagedProposal` frontmatter block is at **lines 322–332**. When `proposal.degradedQuorum` is defined, inject one line just before `reason:` (line 330):

```ts
${proposal.degradedQuorum ? `degraded_quorum: ${fmEscape(JSON.stringify(proposal.degradedQuorum))}\n` : ""}
```

Then `_stageProposal` in `autonomous-distill.ts` (**lines 702–749**) must populate the field when called from the Part B branch:

```ts
const stagedProposal: StagedProposal = {
  // ... existing fields ...
  degradedQuorum: health.alive.length < 3
    ? { alive: health.alive.length, dead: health.dead.map(d => d.model) }
    : undefined,
};
```

This requires `_stageProposal` to see `health`. Either (a) make it a closure inside `autonomousDistill` (simplest — it's already local-state-heavy), or (b) add a new parameter `degraded?: { alive, dead }` and pass it explicitly from the force-stage branch only. Option (b) is cleaner because the other call sites (gate failure, judge rejection, apply failure) should NOT mark degraded even if the panel happened to be degraded.

**Ryan: prefer option (b).** Add `degraded?: StagedProposal["degradedQuorum"]` as the 6th parameter of `_stageProposal`. Pass `undefined` from the four existing call sites (gate-error, gate-fail, judge-throw, judge-reject, apply-fail) and pass the constructed `{alive, dead}` from the new ASL-0024 force-stage branch only.

#### 7. `src/tools/review-staged.ts` — SURFACE FRONTMATTER FLAG

**Line 413** currently shows `(degraded mode)` only when `s.judges.degradedMode` is true (i.e. 1-alive case). Extend it to also check a frontmatter-sourced `degraded_quorum` value on the staged file. This requires threading the frontmatter object through to the list renderer — check the existing `listStagedProposals` call site and whether it already loads frontmatter (it does: `parseStagedFrontmatter` at staged.ts:118, called from `listStagedProposals` at line 408).

Simplest approach: add `degradedQuorum?: { alive: number; dead: string[] }` to the `StagedFile.proposal` reconstruction path in `listStagedProposals` (staged.ts:432) and `readStagedFile` (staged.ts:561), parsing `fm.degraded_quorum` via `JSON.parse` with a try/catch. Then review-staged's line 413 becomes:

```ts
const degradedLabel =
  s.judges.degradedMode ? " (degraded mode — 1 alive)" :
  proposal.degradedQuorum ? ` (degraded quorum — ${proposal.degradedQuorum.alive}/3 alive, dead: ${proposal.degradedQuorum.dead.join(",")})` :
  "";
lines.push(`  Panel verdict: ${panelVerdict}${dim(degradedLabel)}`);
```

#### 8. Tests

**`src/libs/__tests__/memory-hash.test.ts` — NEW FILE**

Test shapes (one-line descriptions, no bodies):

- `_normalizeKnowledgeFileBody strips YAML frontmatter and returns the body`
- `_normalizeKnowledgeFileBody normalizes \r\n to \n`
- `_normalizeKnowledgeFileBody strips trailing whitespace per line`
- `_normalizeKnowledgeFileBody collapses inline whitespace runs to single space`
- `_normalizeKnowledgeFileBody is idempotent (normalize(normalize(x)) === normalize(x))`
- `_computeKnowledgeContentHashForDir returns EMPTY_CONTENT_HASH on missing dir`
- `_computeKnowledgeContentHashForDir returns EMPTY_CONTENT_HASH on dir with zero .md files`
- `_computeKnowledgeContentHashForDir is stable across two calls on the same fixture`
- `_computeKnowledgeContentHashForDir is stable when mtimes are touched but content unchanged`
- `_computeKnowledgeContentHashForDir differs when a new .md file is added`
- `_computeKnowledgeContentHashForDir differs when a line of content is appended to an existing file`
- `_computeKnowledgeContentHashForDir is unchanged when only the frontmatter updated: field moves forward`
- `_computeKnowledgeContentHashForDir is unchanged when a trailing whitespace is appended to a line`
- `_computeKnowledgeContentHashForDir differs when content moves between files (filename separator matters)`
- `_computeKnowledgeContentHashForDir respects filename lex order (same content in a.md vs b.md is deterministic)`
- `computeKnowledgeContentHash resolves the real vault path for a test agent fixture`

**`src/libs/__tests__/agent-lifecycle.test.ts` — EXTEND**

Add to the existing file (do not create a new one — it already initializes the test DB correctly):

- `getLastDistilledKnowledgeHash returns null when no row exists for the agent`
- `getLastDistilledKnowledgeHash returns null when row exists but column is null`
- `setLastDistilledKnowledgeHash creates a row if none exists and stores the hash`
- `setLastDistilledKnowledgeHash updates an existing row's column without touching other fields`
- `setLastDistilledKnowledgeHash is idempotent (set twice with same value is a no-op)`
- `setLastDistilledKnowledgeHash does not affect last_distill_input_hash` (guards the semantic separation)

**`src/libs/__tests__/autonomous-distill-gate.test.ts` — NEW FILE**

The existing test infra does not have an autonomous-distill integration test — we need a new file. Use module stubs (bun:test `mock.module` or direct dependency injection) for `callDistillGemini`, `runJudgePanel`, `checkJudgeHealth`, and `applySoulChange` so tests don't touch Gemini.

- `skips agent when knowledge hash matches stored hash (no Gemini call made)`
- `fires agent when no stored hash exists (first run) and stores the hash after`
- `fires agent when knowledge hash differs from stored hash`
- `stored hash is NOT advanced when distill throws mid-run (ensures next run re-fires)`
- `stored hash IS advanced when all proposals are processed successfully (gate-fail + judge-reject + auto-apply are all "processed")`
- `stored hash is advanced even when no proposals were produced (NO_CHANGES from Gemini)`
- `degraded quorum (2 alive, both approve) force-stages instead of applying — soul file unchanged`
- `degraded quorum (2 alive, both approve) writes staged file with degraded_quorum frontmatter`
- `non-degraded (3 alive, unanimous approve) still auto-applies exactly as before`
- `degraded quorum (1 alive) follows the existing panel.degradedMode path and stages` (regression guard)
- `health check with 0 alive judges still short-circuits the pipeline (unchanged ASL-0017 behavior)`

**`src/libs/__tests__/staged-roundtrip.test.ts` — EXTEND**

- `degraded_quorum frontmatter round-trips through writeStagedProposal → listStagedProposals → StagedFile.proposal.degradedQuorum`
- `writeStagedProposal with no degradedQuorum field produces the same frontmatter as before (back-compat)`

**`src/tools/__tests__/review-staged.test.ts` — EXTEND (if exists; otherwise new file)**

- `list renders "(degraded quorum — 2/3 alive, dead: minimax)" when frontmatter has degraded_quorum`
- `list renders "(degraded mode — 1 alive)" when panel.degradedMode is true`
- `list renders no degraded label when quorum was full`

### Ordering

1. Schema + migration in `brain/schema.ts` and `brain/index.ts`.
2. `memory-hash.ts` helper + tests. Ship and verify before touching agent-lifecycle.
3. `agent-lifecycle.ts` accessors + tests. Verify `last_distill_input_hash` behavior is untouched.
4. `staged.ts` frontmatter extension + roundtrip test. Must not regress AAR-0008 title handling.
5. `autonomous-distill.ts` integration — gate (Part A), force-stage (Part B), hash commit. Use module mocks in the test file; do NOT call real Gemini.
6. `review-staged.ts` surface update.
7. Manual smoke: run `bun run tool asl-sync` against the real vault. Confirm:
   - First run stores the hash, may or may not fire depending on current state.
   - Second run skips all agents with `"knowledge content unchanged since last distill"` in the summary.
   - Artificially kill minimax (unset `MINIMAX_API_KEY` or similar), touch a knowledge file with real content delta, re-run. Confirm the resulting proposal lands in `staged/` with the new `degraded_quorum` frontmatter and `.claude/souls/{agent}.md` is unchanged.

### Acceptance criteria

1. **Hash gate works positively:** running `bun run tool asl-sync` twice in a row against an unchanged vault skips distill the second time with the exact log reason `"knowledge content unchanged since last distill"`.
2. **Hash gate works negatively (mtime touch):** `touch vault/studio/memory/tala/knowledge/*.md` (no content change) still skips on the next run.
3. **Hash gate works negatively (frontmatter-only touch):** updating a `updated: 2026-04-08` field in a knowledge file's frontmatter still skips.
4. **Hash gate fires correctly:** appending a real content line to any knowledge file triggers distill on the next run.
5. **Hash is advanced only on successful completion:** if the Gemini call throws mid-run, the stored hash is unchanged and the next run re-fires.
6. **Schema migration is idempotent:** running `initBrain()` twice against a DB that already has the column is a no-op; running against an old DB adds the column without touching existing rows.
7. **Degraded force-stage works (2-alive path):** stub minimax dead, run distill with a knowledge delta that produces at least one proposal that would pass gates + get unanimous approve from gemini + grok. Confirm the proposal lands in `staged/` with `degraded_quorum: {"alive":2,"dead":["minimax"]}` in the frontmatter. Confirm `.claude/souls/{agent}.md` is byte-identical to its pre-run state.
8. **Degraded force-stage works (1-alive path):** stub 2 judges dead. Confirm the proposal stages with the existing `degradedMode` body marker AND the new `degraded_quorum` frontmatter.
9. **Non-degraded path is unaffected:** with all 3 judges alive and unanimous approve, the proposal auto-applies exactly as before. No `degraded_quorum` frontmatter on the approved soul change record. Existing ASL-0017 tests still green.
10. **ASL-0004 staleness is unaffected:** `isDistillStale`, `computeDistillInputHash`, `recordDistilled` behavior and test assertions are byte-identical before and after this task. The new column is decoupled.
11. **No regression in existing tests:** `bun test src/libs/__tests__/` and `bun test src/tools/__tests__/` all green. Specifically: `agent-lifecycle.test.ts`, `staged-roundtrip.test.ts`, `judge-panel.test.ts`, `schema-asl-0002.test.ts`.
12. **Manual verification on real vault:** operator runs `bun run tool asl-sync` after the fix ships. Confirms (a) distill is skipped for agents whose knowledge hasn't changed, (b) any distill that does fire under a degraded quorum (any judge dead) produces a staged file, not a soul modification.

## Principles being implemented

- **Principle 3:** Distill fires only on meaningful knowledge-content delta.
- **Principle 4:** Degraded judge quorum fails closed (stages) instead of open (auto-applies).

These are the two most actively-firing violations in the current pipeline. Fixing them here immediately stops the bleeding even though the deeper bugs (ASL-0018, 0019, 0020, 0021, 0023) remain.

## What's left for follow-up tasks

After ASL-0024 ships, the remaining ASL backlog is:

- **ASL-0023** (still P0) — rejection feedback loop + audit trail + proper degraded handling. Less acute now that degraded just stages and distill rarely fires, but the audit trail gap is still a real problem and rejection feedback is still needed for the case where knowledge does change and re-produces previously-rejected content.
- **ASL-0021** (P1) — refinement apply semantics. Still blocks meaningful refinement cycles.
- **ASL-0020** (P1) — minimax stability. Still needed because degraded mode should be rare, not routine.
- **ASL-0018** (P2) — Penny distill evidence quality.
- **ASL-0019** (P2) — Sol distill redundancy.
- **ASL-0022** (P3) — wick cleanup.

The daemon remains blocked on ASL-0024 + ASL-0023 minimum. After both ship, it should be safe to start.

## Not in scope

- Fixing the underlying bugs this task gates around. This is a circuit breaker, not a root-cause fix.
- Adding per-agent auto-apply feature flags (future refinement).
- Changing the consolidation trigger (it's less broken than distill's, and Principle 3 applies to it equally — follow-up task if it proves noisy).
- Redesigning the `agent_lifecycle` schema beyond adding one column.
- Changing `judge-panel.ts` `degradedMode` semantics. The 1-alive meaning stays.
- Merging `last_distill_input_hash` and `last_distilled_knowledge_hash`. They are intentionally separate per-semantic columns.
- Touching `distill-cache.ts`. Its raw-bytes cache key is fine — the new gate runs BEFORE the cache lookup, so a stale cache entry won't be consulted on an unchanged-content run.
- Any UX change to `agent-state`.

## Context

- Surfaced during the 2026-04-08 review queue cleanup session.
- User named the framing in conversation: "maybe what we are distilling is not the inbox but the knowledge entry itself... consolidate kinda do a mini-distill... maybe instead of we distill everything, we only distill the consolidated knowledge."
- That insight is captured in PRINCIPLES.md Principle 2 (consolidation is a mini-distill) and operationalized here via Part A.
- The force-stage fix (Part B) is the narrowest possible version of ASL-0023's bug 2. ASL-0023 expands on it with a feature flag and full audit trail work.
- Code-review findings that changed the spec (see Part B "semantic trap" section): `judgeResult.degradedMode` is `aliveCount < 2`, NOT `aliveCount < 3`. The 2026-04-08 Penny auto-apply happened through the 2-alive code path where `degradedMode === false`. The fix gates on `health.alive.length < 3` at the orchestrator level instead of relying on the panel flag.
