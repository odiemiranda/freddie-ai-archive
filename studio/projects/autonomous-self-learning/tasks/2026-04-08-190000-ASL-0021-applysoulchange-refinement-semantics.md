---
id: ASL-0021
title: applySoulChange treats 'modify' proposals as appends, creates stacked duplicates
project-code: ASL
status: shipped
shipped: 2026-04-09
commits: [8d24a33]
priority: P1
created: 2026-04-08
author: freddie
depends-on: []
blocks: []
related: [ASL-0019]
tags: [task, asl, apply-logic, soul-updates, redundancy]
---

# ASL-0021 — applySoulChange doesn't reconcile refinements, only appends

## Observed problem

On 2026-04-08, two Echo proposals were approved and applied to `.claude/souls/echo.md`:

1. **"Expanded Suno Critique Filter Rules"** — proposal type `modify`, intended to refine an existing rule about Gemini's structural observations (adding "I mine individual fixes from these rather than applying them wholesale").
2. **"Prioritized Suno Structural Critique"** — proposal type `add`, adding a new seven-factor critique rule.

After `applySoulChange` ran, Echo's soul contained **three stacked versions** of the Gemini structural observations rule:
- The original short version: `"...missing tags, redundancy) are high-confidence. Its per-instrument mixing or production effects suggestions..."`
- A medium version that was already present (possibly from an earlier distill run): `"...missing tags, redundancy, tag counts)...wholesale rewrites, or atmospheric stripping..."`
- The newly approved refinement: `"...tag counts)...wholesale rewrites; I mine individual fixes..."`

The user hand-edited the soul to collapse all three into the newest refinement (commit at `HEAD`, see git log). But the underlying bug is that **`applySoulChange` has no refinement semantics** — every change, regardless of `Type` field (`add` / `modify`), is implemented as an append below the target section heading.

## Why the regression gate didn't catch it

The regression gate at `src/libs/gates/regression.ts` ran and reported `"1 high-similarity pair(s) checked, no contradictions"` — it detected the high similarity between the old line and the proposed new line, but its contract is to check for **contradictions**, not **duplications**. Two lines can say the same thing differently without contradicting, and the gate correctly passes those.

## Architectural gap

The pipeline today:
1. `distill-soul.ts` produces proposals tagged with `changeType: "add" | "modify" | "delete"`.
2. `staged.ts` writes them to `staged/` with the type preserved.
3. Judges evaluate content quality.
4. `applySoulChange` at approval time... always appends.

The `Type: modify` tag is carried through the whole pipeline but is never honored at the apply step. A modify-type proposal needs to:
- Identify the existing line(s) in the soul that correspond to the "before" state.
- Replace them with the "after" state.
- OR record a no-op if the existing line can't be confidently matched (and fall back to staging-for-manual).

This is not trivial — matching "before" lines requires either:
- (a) The distill step captures the `beforeText` explicitly and passes it through staged files, OR
- (b) Apply step runs a similarity search against the soul at apply time.

Option (a) is cleaner — single source of truth, explicit contract.

## Relationship to ASL-0019

ASL-0019 is about **Sol's distill producing redundant proposals**. ASL-0021 is about **apply logic creating stacked duplicates even when distill was clean**. They're different bugs that produce similar-looking symptoms (redundant soul content), and fixing one doesn't fix the other.

The combined effect: **the self-learning loop has no mechanism to meaningfully refine existing rules**. It can only accrete. Every soul file grows monotonically, and the regression gate's "no contradictions" PASS is a false reassurance.

## Goals

1. Proposals with `Type: modify` should *replace* target lines in the soul, not append.
2. The distill step must emit a `beforeText` field (exact string to match) alongside the `afterText` for every modify-type proposal.
3. If `applySoulChange` can't find a confident match for `beforeText`, the proposal is rejected with a clear error and archived separately — not silently appended.
4. Add a test covering the refinement case: stage a modify proposal, apply it, assert the soul has the new line and NOT the old line.

## Investigation path

1. Read `src/libs/apply-soul-change.ts` (or wherever the apply logic lives). Confirm the current "always append" behavior.
2. Read `src/tools/distill-soul.ts` — does it already generate a `beforeText`? If yes, apply just needs to honor it. If no, distill needs a schema update.
3. Read `src/libs/staged.ts` — does the staged proposal format have a slot for `beforeText`? If not, ASL-0012's file format needs a non-breaking extension.
4. Check Echo's `.approved/` directory for the two proposals applied today — confirm they don't contain a `beforeText` field, which would explain why apply had nothing to match against.
5. Write a failing test first: stage a modify proposal with a beforeText, apply it, assert the old line is gone and the new line is in.

## Not in scope

- Fixing past accumulated duplication in existing soul files (manual cleanup).
- Changing the `Type` field values or semantics.
- Adding `delete` proposal support (separate concern).

## Why P1

Every modify-type proposal accepted today stacks a duplicate into a soul file. The self-learning loop's whole value depends on souls actually converging toward truth, not accreting every phrasing variant. This bug defeats the loop's core purpose. The longer we let it run, the more cosmetic cleanup accumulates, and the more gemini rejects future proposals as "duplicates existing traits" (see ASL-0019 — Sol's entire review batch rejected on this ground, likely at least partly caused by this same apply bug earlier in her pipeline).

## Context

- User manually collapsed Echo's 3 stacked "Gemini structural observations" lines into 1 during the 2026-04-08 review queue commit. See `git log -p .claude/souls/echo.md` for the cleanup diff.
- Two approved Echo proposals archived at `vault/studio/memory/echo/approved/` with the raw (stacked) application history.
- ASL-0019 may partially resolve itself once ASL-0021 ships — if refinements actually refine, Sol's distill redundancy will have a path to converge.

---

## Refined spec (mccall, 2026-04-09)

### 0. Discovery notes — what changed since the task was filed

- **There is no `src/libs/apply-soul-change.ts`.** `applySoulChange` lives in `src/libs/autonomous-distill.ts`. Current location: **line 301** (signature), **lines 312-334** (body). `appendToSection` helper at **line 188**.
- The `ParsedProposal` type uses the variant `"add" | "remove" | "modify"` — the third value is `"remove"`, NOT `"delete"`. `applySoulChange` already has a working `"remove"` branch (lines 321-334) that does literal substring removal. **This bug only affects the `"modify"` branch.** The current code explicitly groups `add` and `modify` together: `if (proposal.type === "add" || proposal.type === "modify")` then `appendToSection(...)`. That line IS the bug.
- `StagedProposal.changeType` in `src/libs/staged.ts:25` matches: `"add" | "remove" | "modify"`. Same values carry from `ParsedProposal.type` through `StagedProposal.changeType` and back (via `toParsedProposal` in `src/tools/review-staged.ts:260`).
- **Two call sites** of `applySoulChange` today — both must keep working:
  1. `src/libs/autonomous-distill.ts:799` — auto-apply path after judges unanimously approve.
  2. `src/tools/review-staged.ts:620` — human-in-the-loop approve path (reconstructs a `ParsedProposal` from a `StagedFile` via `toParsedProposal`).
- Staged file format: frontmatter carries `change_type: {add|remove|modify}` plus `section` and free-form `reason`. Body carries `## Proposed Change`, `## Evidence`, `## Gate Results`, `## Judge Results`, `## Reason for Staging`, `## Decision`. **There is no slot for `beforeText` today.** `parseStagedFrontmatter` (staged.ts:123) is a flat key-value parser and supports JSON-encoded string values — adding a new key is non-breaking.
- **The distill LLM prompt explicitly tells Gemini the opposite of what we now want.** `src/tools/distill-soul.ts:219` contains the rule: "CONTENT should be what you are ADDING to a section (even for modify-type proposals — the auto-apply system appends, never replaces)." This line is the documented policy enshrined in the prompt — fixing the apply code without fixing the prompt will produce `modify` proposals whose content is not a replacement candidate at all. **Prompt + code must ship together.**
- **`distill-cache.ts` versions the prompt.** Every prompt change requires bumping `DISTILL_PROMPT_VERSION` in `src/libs/distill-cache.ts` or stale cached results will reintroduce the old behavior. This is called out in a comment at `distill-soul.ts:147-148`. Ryan MUST bump the version.
- There is currently no standalone unit test for `applySoulChange`. Existing tests in `auto-apply-audit.test.ts`, `autonomous-distill-gate.test.ts`, and `rejection-feedback.test.ts` all inject a noop `applySoulChangeFn`. **A new test file dedicated to `applySoulChange` is the right home for the refinement tests.**
- **ASL-0024 cross-cutting concern.** The knowledge-content delta gate (`_computeKnowledgeContentHashFn` + `_getLastDistilledKnowledgeHashFn`) fires BEFORE the distill call. It will prevent re-distills while knowledge is unchanged — meaning Ryan's manual smoke test needs to either (a) touch a knowledge file, (b) run with `noCache: true` AND clear the stored hash, or (c) test through the injected-deps path. Document this in the smoke plan.
- **ASL-0023 cross-cutting concern.** The post-LLM rejection hash filter (`_computeRejectionHash` + `_findSimilarRejections`) runs against the proposal's `content` field. When a `modify` proposal's `content` changes shape (after the prompt update, it will be the replacement text, not the addition text), rejection matching will still work because it normalizes both sides through `_normalizeForRejectionHash`. No change needed, but note it so Ryan isn't surprised when existing rejection hashes still match.
- **ASL-0025 has no known interaction** — verify with a quick `git log` on `autonomous-distill.ts` before starting.

### 1. File inventory

| File | Current LOC | Change | Why |
|------|------|--------|-----|
| `src/libs/autonomous-distill.ts` | 960 | Modify `applySoulChange` (line 301). Add new helper `replaceInSoul`. Extend `ParsedProposal` interface (line 76) with optional `beforeText?: string`. Update `parseDistillOutput` (line 250) to extract `BEFORE_TEXT:`. Copy `beforeText` into `auditProposal` at line 819. | Core fix: honor `modify` semantics via find-and-replace. |
| `src/libs/staged.ts` | 702 | Extend `StagedProposal` interface (line 19) with optional `beforeText?: string`. Write it as JSON-encoded `before_text:` frontmatter key in `writeStagedProposal` (line 394) and `writeApprovedArchive` (line 321). Read it in `listStagedProposals` (line 459) and `readStagedFile` (line 632). | Plumb `beforeText` through staged storage round-trip. |
| `src/tools/distill-soul.ts` | 424 | Update distill prompt (lines 153-220) — remove the "appends, never replaces" line, add the `BEFORE_TEXT:` contract for modify proposals. | LLM must emit the replacement target. |
| `src/tools/review-staged.ts` | ~1100 | Update `toParsedProposal` (line 260) to copy `sf.proposal.beforeText` through. | Human-approve path preserves the contract. |
| `src/libs/distill-cache.ts` | small | Bump `DISTILL_PROMPT_VERSION`. | Prompt changed → cache must invalidate. |
| `src/libs/__tests__/apply-soul-change.test.ts` | NEW | Add 6 tests (see §6). | Regression coverage. |

**Do NOT touch:**
- `src/libs/gates/*` — gates run on the `proposedChange` string; they don't care about `beforeText`.
- `src/libs/judge-panel.ts` — judges receive `formatProposalForJudge` which already shows `Type` and `Content`. Including `beforeText` in the judge prompt is a nice-to-have, explicitly OUT of scope for this task.
- The `"remove"` branch. It already works.
- `consolidate-memory.ts`, brain.db queries, the ASL daemon. None read `beforeText`.

### 2. Function signatures

**`ParsedProposal` (autonomous-distill.ts:76) — extend:**

```ts
export interface ParsedProposal {
  title: string;
  type: "add" | "remove" | "modify";
  section: string;
  confidence: "high" | "medium";
  evidence: string;
  content: string;
  /** For type="modify": exact substring of the current soul to replace with `content`.
   *  Required for modify. Ignored for add/remove. */
  beforeText?: string;
}
```

**`StagedProposal` (staged.ts:19) — extend:**

```ts
export interface StagedProposal {
  // ... existing fields unchanged ...
  /** Only populated for changeType="modify". Exact current-soul substring that
   *  `proposedChange` should replace. Empty/undefined for add/remove. */
  beforeText?: string;
}
```

**`applySoulChange` (autonomous-distill.ts:301) — rewrite body with three branches:**

```ts
export function applySoulChange(
  agent: string,
  soulPath: string,
  proposal: ParsedProposal,
): void {
  if (!existsSync(soulPath)) {
    throw new Error(`Soul file not found: ${soulPath}`);
  }
  const current = readFileSync(soulPath, "utf-8");

  if (proposal.type === "add") {
    if (!proposal.content) {
      throw new Error(`add proposal "${proposal.title}" has no content`);
    }
    const updated = appendToSection(current, proposal.section, proposal.content);
    writeFileSync(soulPath, updated, "utf-8");
    return;
  }

  if (proposal.type === "modify") {
    if (!proposal.content) {
      throw new Error(`modify proposal "${proposal.title}" has no content`);
    }
    if (!proposal.beforeText || !proposal.beforeText.trim()) {
      throw new Error(
        `modify proposal "${proposal.title}" has no beforeText — cannot locate ` +
        `replacement target. Reject the proposal or re-issue as an add.`
      );
    }
    const updated = replaceInSoul(current, proposal.beforeText, proposal.content, proposal.title);
    writeFileSync(soulPath, updated, "utf-8");
    return;
  }

  if (proposal.type === "remove") {
    // UNCHANGED — existing branch body at lines 321-334 of current file.
    // ...
  }
}
```

**New helper `replaceInSoul` (exported, colocated with `applySoulChange`):**

```ts
/**
 * Replace `beforeText` with `afterText` in `content`.
 * Normalizes CRLF → LF on both sides for matching.
 * Throws on zero matches or ambiguous (>1) matches.
 */
export function replaceInSoul(
  content: string,
  beforeText: string,
  afterText: string,
  proposalTitle: string,
): string {
  const normalized = content.replace(/\r\n/g, "\n");
  const target = beforeText.replace(/\r\n/g, "\n").trim();
  if (!target) {
    throw new Error(`modify proposal "${proposalTitle}": beforeText is empty after trim`);
  }
  const firstIdx = normalized.indexOf(target);
  if (firstIdx === -1) {
    throw new Error(
      `modify proposal "${proposalTitle}": beforeText not found in soul file. ` +
      `Soul may have drifted since distillation. First 80 chars: ` +
      `"${target.slice(0, 80).replace(/\n/g, "\\n")}"`
    );
  }
  const secondIdx = normalized.indexOf(target, firstIdx + target.length);
  if (secondIdx !== -1) {
    // Count all occurrences for a clear error message
    let count = 0;
    let i = 0;
    while ((i = normalized.indexOf(target, i)) !== -1) { count++; i += target.length; }
    throw new Error(
      `modify proposal "${proposalTitle}": beforeText matches ${count} locations in soul file — ambiguous, refusing to guess`
    );
  }
  return normalized.slice(0, firstIdx) + afterText.trim() + normalized.slice(firstIdx + target.length);
}
```

**`parseDistillOutput` (autonomous-distill.ts:250) — extend body parser.** After the existing `contentMatch` extraction (around line 269), add:

```ts
const beforeTextMatch = body.match(/^BEFORE_TEXT:\s*\n([\s\S]*?)(?=\n\s*CONTENT:)/m);
const beforeText = beforeTextMatch?.[1]?.trim() || undefined;
```

And push it onto the returned proposal object. Parser MUST still accept proposals without `BEFORE_TEXT` (legacy, add, remove).

### 3. Distill prompt diff

In `src/tools/distill-soul.ts` (around lines 196-220), apply this surgical edit.

**REMOVE** (currently at line 219):

> `- **No full section rewrites.** Propose surgical refinements, not replacements. CONTENT should be what you are ADDING to a section (even for modify-type proposals — the auto-apply system appends, never replaces).`

**REPLACE** the proposal block template (currently lines 196-204):

Old template:

```
=== PROPOSAL N: {short title} ===
TYPE: add | modify | remove
SECTION: {which soul file section this belongs in}
CONFIDENCE: high | medium
EVIDENCE: {brief — what knowledge entries support this}

CONTENT:
{The exact text to add to the soul file (max 6 lines), written in the agent's voice}
=== END PROPOSAL ===
```

New template:

```
=== PROPOSAL N: {short title} ===
TYPE: add | modify | remove
SECTION: {which soul file section this belongs in}
CONFIDENCE: high | medium
EVIDENCE: {brief — what knowledge entries support this}

BEFORE_TEXT:
{ONLY for TYPE: modify — the exact current text in the soul file that will be replaced. Must be copied verbatim from the soul shown above, character-for-character including punctuation and whitespace. Omit this field entirely for add and remove.}

CONTENT:
{The replacement text for modify, OR the new text to append for add, OR the substring to remove for remove. Max 6 lines. Written in the agent's voice.}
=== END PROPOSAL ===
```

**ADD** to the Rules block (replacing the removed line):

```
- **modify vs add — when in doubt, use add.** A `modify` proposal REPLACES existing soul text. You MUST provide a BEFORE_TEXT block containing the exact current line(s) to replace, copied verbatim from the "Current Soul File" block above. If you cannot find the exact existing text, use TYPE: add instead — it is always safe. An add is never wrong; a modify with a mismatched BEFORE_TEXT will be rejected by the apply step.
- **BEFORE_TEXT must be unique in the current soul.** If the text you want to replace appears more than once, expand the BEFORE_TEXT to include enough surrounding context to make it unique, or use TYPE: add.
- **No full section rewrites.** Propose surgical refinements, not replacements of entire sections.
```

**BUMP** `DISTILL_PROMPT_VERSION` in `src/libs/distill-cache.ts` (increment by 1).

### 4. Staged file format extension

**Frontmatter** (non-breaking — add a new key). In `writeStagedProposal` and `writeApprovedArchive`, when `proposal.beforeText` is non-empty, emit:

```
before_text: ${fmEscape(proposal.beforeText)}
```

Only write the line when `beforeText` is set. Keep files clean for add/remove. `fmEscape` already JSON-encodes the value, which handles newlines correctly.

**Read path.** `listStagedProposals` and `readStagedFile` both call `parseStagedFrontmatter`. Add to both construction sites:

```ts
beforeText: fm.before_text || undefined,
```

**No body-section change.** `beforeText` lives in frontmatter only. Do NOT add a `## Before Text` section — it would change the body parser contract unnecessarily and create ambiguity with `## Proposed Change`.

### 5. Error handling matrix

| Failure case | `applySoulChange` behavior | Pipeline behavior | Staged outcome |
|---|---|---|---|
| `modify` with missing/empty `beforeText` | Throws `"… has no beforeText — cannot locate replacement target."` | Auto-apply loop's `try/catch` at line 800 catches, calls `_stageProposal` with reason `"Apply failed: <message>"` | Lands in `{agent}/staged/` with `status: pending`, reason explains missing contract. |
| `modify` with `beforeText` matching zero substrings | Throws `"… beforeText not found in soul file. Soul may have drifted …"` | Same path → staged | Flagged for human review. |
| `modify` with `beforeText` matching 2+ substrings | Throws `"… beforeText matches N locations … ambiguous, refusing to guess"` | Same path → staged | Next distill run (per prompt update) is instructed to expand the beforeText for uniqueness. |
| `modify` with matching unique `beforeText` | Writes replaced soul via existing `writeFileSync` | Continues to audit-write + `logSoulChange` | `auto-applied` |
| `add` (any shape) | Unchanged — `appendToSection` | Unchanged | Unchanged |
| `remove` (any shape) | Unchanged | Unchanged | Unchanged |
| Legacy staged file (pre-ship, `modify` with no `before_text`) + human approve | `toParsedProposal` yields `beforeText=undefined` → `applySoulChange` throws missing-contract error | `applyApprove` in `review-staged.ts` surfaces the error; file is NOT moved to `approved/`, nothing written to soul | Human sees a clear error. See §8. |

**No silent fallback to append.** Defining principle: `modify` without a resolvable `beforeText` never mutates the soul. Ever.

### 6. Test shapes

**New file:** `src/libs/__tests__/apply-soul-change.test.ts`. All tests use a temp-dir soul file via `mkdtempSync(join(tmpdir(), "asl-0021-"))`. Each creates a minimal soul with known content, calls `applySoulChange`, and asserts outcome.

1. **`modify with matching unique beforeText → replaces cleanly`** — Soul contains `"- Original line here."`. Proposal `{type:"modify", content:"- Refined line here.", beforeText:"- Original line here."}`. Assert soul now contains refined line and NOT original.

2. **`modify with missing beforeText → throws, soul unchanged`** — Proposal `{type:"modify", content:"whatever", beforeText: undefined}`. Assert throws with message containing `"has no beforeText"`. Assert soul content identical to pre-call.

3. **`modify with non-matching beforeText → throws, soul unchanged`** — Soul contains `"- A line."`. Proposal's beforeText is `"- A totally different line."`. Assert throws with message containing `"not found in soul file"`. Assert soul unchanged.

4. **`modify with ambiguous beforeText (2 matches) → throws, soul unchanged`** — Soul contains two identical lines `"- Repeated rule."` in different sections. Proposal beforeText is `"- Repeated rule."`. Assert throws with message containing `"ambiguous"` and shows count. Assert soul unchanged.

5. **`add proposal still appends (regression guard)`** — Proposal `{type:"add", section:"How I Think", content:"- New belief."}`. Assert soul grew by the new line, inserted in the correct section per `appendToSection` semantics. No `beforeText` field on the proposal.

6. **`modify with CRLF-normalized beforeText matches LF soul`** — Soul uses `\n`. Proposal's `beforeText` contains `\r\n` (simulating LLM output from a Windows-style prompt render). Assert match succeeds — `replaceInSoul` normalizes both sides.

**Existing test updates:**

- Extend `toParsedProposal` test in `src/tools/__tests__/review-staged.test.ts:460` to assert a `beforeText` round-trip when the staged fixture includes `before_text` frontmatter.
- `auto-apply-audit`, `autonomous-distill-gate`, `rejection-feedback` — should compile without changes since `beforeText` is optional. Run them to confirm.

### 7. Acceptance criteria

- [ ] `ParsedProposal.beforeText` exists as an optional field.
- [ ] `StagedProposal.beforeText` exists as an optional field.
- [ ] `applySoulChange` has three separate branches (`add` / `modify` / `remove`) — `modify` is NOT grouped with `add`.
- [ ] `applySoulChange`'s `modify` branch calls `replaceInSoul` and throws on missing/not-found/ambiguous `beforeText`.
- [ ] `replaceInSoul` exists, is exported (for testability), and CRLF-normalizes both sides.
- [ ] `parseDistillOutput` extracts `BEFORE_TEXT:` from the Gemini output into `ParsedProposal.beforeText`, and still parses legacy proposals without that field.
- [ ] Distill prompt in `distill-soul.ts` includes the new `BEFORE_TEXT` contract and the "when in doubt, use add" rule. The old "appends, never replaces" line is gone.
- [ ] `DISTILL_PROMPT_VERSION` in `distill-cache.ts` is incremented.
- [ ] `writeStagedProposal` and `writeApprovedArchive` emit `before_text:` frontmatter when non-empty; omit the line when empty/undefined.
- [ ] `listStagedProposals` and `readStagedFile` read `before_text` from frontmatter into `StagedProposal.beforeText`.
- [ ] `toParsedProposal` in `review-staged.ts` copies `beforeText` through.
- [ ] Auto-apply path at `autonomous-distill.ts:819` includes `beforeText` in the constructed `auditProposal` passed to `writeApprovedArchive`.
- [ ] New test file `src/libs/__tests__/apply-soul-change.test.ts` with the 6 tests from §6, all passing.
- [ ] Existing tests (`auto-apply-audit`, `autonomous-distill-gate`, `rejection-feedback`, `review-staged`) still pass.
- [ ] `bun run tool brain --check` still works. No DB schema change.
- [ ] Manual smoke: hand-craft a staged file with `change_type: modify` + `before_text:` matching a real soul line, run `bun run tool review-staged approve <id> --yes`, verify soul has the new line and NOT the old one. Revert after.

### 8. Backward compatibility plan

**Scenario A — legacy staged files on disk (pre-ship).** Any pending `change_type: modify` staged files without `before_text` frontmatter. After ship:
- The human-approve path will throw a clear missing-contract error. The staged file is NOT moved, the soul is NOT touched.
- **McCall decision: manual reject.** Ryan lists pending modifies via `bun run tool review-staged list --json`, pipes through `jq` to filter `changeType=modify`, the user reviews and rejects each via `review-staged reject <id>`. Expected volume is < 10 files (per the ASL-0017 audit). Writing a migration tool is overkill.
- Document this in the PR description as a post-ship manual step.

**Scenario B — legacy approved-archive files.** Read-only historical records, never re-applied. No change needed. The new frontmatter key is simply absent on old files; `readStagedFile` returns `beforeText: undefined`.

**Scenario C — mid-pipeline distill cache holds pre-prompt-update results.** The `DISTILL_PROMPT_VERSION` bump invalidates all cached results. First post-ship run re-calls Gemini. No stale-prompt proposals leak through.

### 9. Scope boundaries (NOT in this task)

- **No `delete` type.** Not a valid value anywhere. `"remove"` already exists and works.
- **No atomic soul writes.** Existing non-atomic `writeFileSync` risk pre-dates this task (documented in ASL-0012).
- **No judge prompt update to include `beforeText`.** Judges evaluate content quality, not matching correctness. Follow-up task if needed.
- **No `BEFORE_TEXT` fuzzy matching.** Strict exact-match only (after CRLF normalization). If the LLM can't produce an exact match, it should fall back to `TYPE: add`.
- **No LLM self-repair on ambiguous match.** Failed modifies stage for human review — no second Gemini round.
- **No refactor of `appendToSection`.** Left as-is.
- **No changes to gates.** Gate 2 (size) and regression gate work on `content` alone.
- **No changes to reflection / consolidation.** They never touch `applySoulChange`.
- **No cleanup of the already-stacked Echo duplicates.** Already manually collapsed by the user.

### 10. Cross-cutting risks Ryan must be aware of

- **ASL-0024 knowledge-delta gate.** If Ryan tests end-to-end by running `autonomousDistill` on a real agent, the gate may skip the run because the knowledge hash hasn't moved. For manual smoke, either (a) touch a knowledge file, (b) clear the stored hash via `setLastDistilledKnowledgeHash(agent, "")`, or (c) test through the injected-deps path. Unit tests on `applySoulChange` directly don't hit this.
- **ASL-0023 rejection hash filter.** Runs on `proposal.content`, not `proposal.beforeText`. After this ship, a `modify` proposal's content is the *replacement text*. Rejection matching still works via normalization — this is a neutral observation, not a bug.
- **ASL-0025.** `git log --oneline src/libs/autonomous-distill.ts` before starting to confirm no concurrent edits around `applySoulChange`. Resolve any merge conflict by preserving ASL-0021's signature change.
- **Prompt-cache invalidation.** If Ryan forgets to bump `DISTILL_PROMPT_VERSION`, the first run after ship will serve a cached response from BEFORE the prompt update — no `BEFORE_TEXT` field — every modify proposal will fail the contract check and stage. Not dangerous, but noisy. **Bump the version.**

### 11. Ship order (suggested)

1. Extend `ParsedProposal` + `StagedProposal` interfaces (type plumbing).
2. Write the 6 new tests — they should FAIL against current `applySoulChange`.
3. Implement `replaceInSoul` + rewrite `applySoulChange`. Tests should PASS.
4. Update `parseDistillOutput` for `BEFORE_TEXT` extraction.
5. Update `writeStagedProposal`, `writeApprovedArchive`, `listStagedProposals`, `readStagedFile` for the `beforeText` round-trip.
6. Update `toParsedProposal` in `review-staged.ts`. Extend the existing test.
7. Update auto-apply path at `autonomous-distill.ts:819` to copy `beforeText` into the audit proposal.
8. Update distill prompt in `distill-soul.ts` + bump `DISTILL_PROMPT_VERSION`.
9. Run full test suite. Expect green across `apply-soul-change`, `review-staged`, `auto-apply-audit`, `autonomous-distill-gate`, `rejection-feedback`.
10. Manual smoke: hand-craft a staged modify with matching `before_text`, approve via CLI, verify soul, revert.
11. Commit referencing ASL-0021.
