---
id: ASL-0016
title: Fix multi-line gate reason truncation in staged proposal writer
project-code: ASL
status: planned
priority: P1
created: 2026-04-08
author: mccall
implementer: ryan
reviewer: mccall
depends-on: [ASL-0012]
blocks: []
files:
  - src/libs/staged.ts
  - src/tools/review-staged.ts
  - src/libs/__tests__/staged-roundtrip.test.ts
  - src/tools/__tests__/review-staged.test.ts
  - vault/studio/projects/autonomous-self-learning/tasks/TASKS.md
estimated-complexity: small (1-2 hours)
tags: [task, bug, data-loss, staged-proposals, asl-phase-2]
---

# ASL-0016 — Fix multi-line gate reason truncation in staged proposal writer

## TL;DR

`formatGateResults()` in `src/libs/staged.ts:96-106` serializes `GateResult.reason` directly into a markdown table cell with no escaping. When a gate produces a reason containing `\n` or a literal `|`, the table row terminates at the first newline and everything after is silently truncated on disk. This is **not** a display bug — the data is lost at write time and cannot be recovered by any reader.

Evidence: `vault/studio/memory/echo/staged/20260407-164110-refined-multi-stage-critique.md:33`:

```
| constitution | false | FAIL. The proposed change introduces a contradiction regarding |
```

The cell ends mid-sentence. Everything the LLM said after "regarding" is gone from disk.

**Impact.** The staged proposal is the human's only window into why a gate rejected a soul change. Losing the back half of the reason defeats the human-in-the-loop contract that ASL-0012 exists to enforce. For a system that routes gate failures to human review, a silent-truncation bug in the staged record is a P1 data-loss defect.

**Fix.** Escape `\n`, `|`, and `\` in both gate and judge reason cells at write time. Decode symmetrically at parse time. Add a `formatCellForDisplay()` helper to `review-staged show` so humans see properly-wrapped text while the on-disk format remains a single-line table cell.

---

## Goals

1. **Stop the bleeding.** New staged files written after this ships must round-trip any `GateResult.reason` through write → read → parse with zero information loss, including multi-line reasons and reasons containing `|`.
2. **Symmetric writer + parser change.** The escape scheme is a tight contract between two functions: `formatGateResults()` (producer) and `parseGateResultsTable()` (consumer). Both land in the same commit.
3. **Pin the fix with a round-trip regression test.** A multi-line reason fixture flows through `writeStagedProposal()` → `listStagedProposals()` → field equality. If anyone ever reverts the escape, the test fails loudly.
4. **Preserve human readability in the reviewer UI.** `review-staged show` must decode escapes before displaying gate reasons so the reviewer sees the actual multi-line reason, not `\n` literals.
5. **Close the same gap for judge reasons.** `formatJudgeResults()` has the same bug shape — a judge `reason` containing `\n` or `|` will corrupt the row. Apply the same escape to both sides in one pass.

## Non-Goals

- **Backfilling legacy staged files.** Tracked separately as **ASL-0017** (§Scope split). Data already lost on disk cannot be recovered by a migration script — only the writer fix prevents future loss.
- **Recovering the truncated content in `20260407-164110-refined-multi-stage-critique.md`.** The bytes are gone. The smoke test will confirm the file is unrecoverable and document it as a known dead record.
- **Changing the `## Gate Results` section to a non-table format** (Option D — see §Design decision). Rejected. Forces a parser rewrite, breaks format symmetry with 12 existing files, larger surface area than the fix requires.
- **Rewriting `GateResult.reason` on the gates side** to enforce single-line reasons. Rejected. Constitution gate deliberately pipes LLM verdicts that are naturally multi-line; truncating upstream would lose information at a different layer.
- **Schema changes to `StagedProposal`.** The type is fine; the serializer is broken.

---

## Hard locks

- **Same escape scheme for gates and judges.** Do not introduce two different encodings. One `escapeTableCell()` + one `unescapeTableCell()` helper pair used by both `formatGateResults` + `formatJudgeResults` writers and both `parseGateResultsTable` + `parseJudgeResultsTable` parsers.
- **No new dependencies.** Pure string transforms. No markdown parser library, no YAML, no anything.
- **Do not touch `writeStagedProposal()` frontmatter rendering.** That code already uses `fmEscape()` (JSON.stringify) and handles multi-line values correctly. This task is scoped to the body sections only.
- **Do not touch the legacy 3-column judge results path in `parseJudgeResultsTable()`.** The dual-format branch at `review-staged.ts:175-193` stays exactly as-is. Its cleanup is ASL-0017's concern.
- **Do not modify `autonomous-distill.ts`.** The gate runner is correct. The bug is entirely in the serialization layer.

---

## Producer-side audit

### Where the bug lives

`src/libs/staged.ts:96-106`:

```ts
function formatGateResults(gates: AllGatesResult): string {
  const lines: string[] = [];
  lines.push(`Passed: ${gates.passed}`);
  lines.push("");
  lines.push("| Gate | Pass | Reason |");
  lines.push("|------|------|--------|");
  for (const g of gates.results) {
    lines.push(`| ${g.gate} | ${g.pass} | ${g.reason} |`);  // ← BUG
  }
  return lines.join("\n");
}
```

`${g.reason}` is interpolated directly. Any `\n` in the reason breaks the row; any `|` breaks the column count.

`src/libs/staged.ts:119-124` has the same bug for judges:

```ts
lines.push("| Model | Vote | ErrorType | Reason |");
lines.push("|-------|------|-----------|--------|");
for (const v of judges.votes) {
  const et = v.errorType ?? "substantive";
  lines.push(`| ${v.model} | ${v.vote} | ${et} | ${v.reason} |`);  // ← same bug
}
```

### Why the data shape matters

`src/libs/gates.ts` defines `GateResult = { gate, pass, reason }` where `reason` is an unconstrained `string`. Grep confirms most built-in gates produce short single-line reasons (e.g., `"No changes target protected sections"`, `"30-day drift 3/5 (46-line soul, 10% threshold)"`). The constitution gate is the outlier — it passes an LLM verdict through, which can contain full paragraphs with newlines. That's the path that produced the evidence file.

**Assumption audit:** The task doc for ASL-0005-era staging assumed gate reasons would be short single-line strings. That assumption was never validated against the LLM-backed constitution gate. Knight Capital lesson: untested assumption, silent failure. Fix the serializer, not the gates.

### Downstream consumer

`src/tools/review-staged.ts:107-126` parses the table with a per-line regex:

```ts
const rowRe = /^\|\s*(\w+)\s*\|\s*(true|false)\s*\|\s*(.*?)\s*\|?\s*$/gm;
```

This is a correct parser **for a correctly-written table**. When the writer corrupts the table by inserting a literal `\n` inside a cell, the regex matches only the first fragment and the rest becomes orphaned text between the row and the `## Judge Results` header. The parser has no way to reassemble it.

---

## Design decision — escape strategy

### Options considered

| Option | Mechanism | Reversible? | Handles `\n`? | Handles `\|`? | Readable in raw MD? | Renders in Obsidian? | Parser change |
|---|---|---|---|---|---|---|---|
| **A: `<br>` tags** | `reason.replace(/\n/g, "<br>")` | Fuzzy | Yes | **No** | Fair | Yes | Light |
| **B: backslash escapes** (CHOSEN) | `replace \ → \\`, `\n → \n literal`, `\| → \|` | **Yes** | Yes | Yes | Acceptable | Raw text shows `\n` | Light (symmetric) |
| **C: space collapse** | `reason.replace(/\s+/g, " ")` | **No — lossy** | No (flattens) | Needs separate handling | Best | Best | None |
| **D: definition-list rewrite** | Drop the table; write `### {gate}\n**Reason:**\n{reason}` per result | Yes | Native | Native | Best | Best | **Full parser rewrite** |

### Decision: Option B — backslash escapes

**Reasoning chain:**

1. The hard constraint is lossless round-trip through a parser. That disqualifies **C** (lossy by design) and weakens **A** (doesn't handle `|`, meaning a gate reason like `"expected a|b got c"` still corrupts the row).
2. Between **B** and **D**, both are correct. **D** is architecturally cleaner — a definition list natively holds multi-line content and eliminates the escape dance entirely. But **D** requires:
   - A new section parser in `review-staged.ts` (larger blast radius).
   - Coexistence with 12+ legacy files that use the table format, which either forces a dual-format parser (more branches, same debt we're trying to escape) or forces ASL-0017 to backfill the old files (coupling that we explicitly want to avoid — see §Scope split).
   - A display change in `review-staged show`, which currently renders the section as-is.
3. **B** is the minimum viable fix:
   - One escape helper + one unescape helper, ~6 lines each.
   - Writer change is one character per cell: `escapeTableCell(g.reason)`.
   - Parser change is one call at the tail of each regex match: `unescapeTableCell(m[3].trim())`.
   - Legacy files are not affected — the unescape is a no-op on text that contains no `\n` / `\|` / `\\` sequences, which is exactly the shape of every pre-fix file.
   - Raw-text readability cost: a reviewer reading the staged file in Obsidian raw view sees `\n` instead of a line break inside the cell. Acceptable — `review-staged show` decodes on display, and Obsidian-rendered-view users rarely read staged proposals directly anyway (they use the CLI).
4. Knight Capital rule: when two correct options exist, prefer the one with the smaller blast radius. **B** touches 4 functions and 0 existing files on disk. **D** touches 4 functions and requires a migration path for 12 files.

**Decision:** **Option B.** Revisit only if a future requirement forces structured multi-line gate output (unlikely).

### Escape specification

Canonical helpers live in `src/libs/staged.ts` and are exported for the parser side to import.

```ts
/**
 * Escape a string for safe insertion into a single markdown table cell.
 * Order matters: escape the escape character first, then the metacharacters,
 * so that unescapeTableCell() can reverse the operation unambiguously.
 *
 * Transforms:
 *   \  → \\
 *   |  → \|
 *   \n → \n (literal two characters)
 *   \r → (dropped — we normalize line endings to \n on the way in)
 */
export function escapeTableCell(s: string): string {
  return s
    .replace(/\\/g, "\\\\")
    .replace(/\|/g, "\\|")
    .replace(/\r\n/g, "\n")
    .replace(/\r/g, "\n")
    .replace(/\n/g, "\\n");
}

/**
 * Reverse of escapeTableCell. Applied to the raw cell text captured by
 * parseGateResultsTable / parseJudgeResultsTable.
 *
 * Transforms (in reverse order):
 *   \n (literal) → \n (newline)
 *   \|           → |
 *   \\           → \
 *
 * Note: must process in a single pass to avoid \\n being decoded as \<newline>.
 */
export function unescapeTableCell(s: string): string {
  let out = "";
  for (let i = 0; i < s.length; i++) {
    if (s[i] === "\\" && i + 1 < s.length) {
      const next = s[i + 1];
      if (next === "\\") { out += "\\"; i++; continue; }
      if (next === "|")  { out += "|";  i++; continue; }
      if (next === "n")  { out += "\n"; i++; continue; }
      // Unknown escape — pass through literally (forward compatibility)
      out += s[i];
      continue;
    }
    out += s[i];
  }
  return out;
}
```

**Why the char-by-char unescape and not three chained `.replace()` calls?** Chaining `replace(/\\\\/g, X).replace(/\\n/g, "\n").replace(/\\\|/g, "|")` with a sentinel is fragile. The single-pass loop is 15 lines, obviously correct, and handles the `\\n` ambiguity (escaped backslash followed by the letter n) without placeholder gymnastics.

### Legacy compatibility

Any file written before this fix contains no `\\`, no `\|`, and no `\n` (literal two-char sequence) inside its cells — because those sequences simply didn't exist in the old format. `unescapeTableCell()` on an old cell is a no-op. No version flag needed. No parser branching needed.

**Edge case:** If a legacy gate reason happened to contain a literal `|` or the two-character sequence `\n`, the new parser would misread it. In practice: checked every file under `vault/studio/memory/*/staged/` — none contain these sequences. Even if a borderline case existed, the pre-fix row was already corrupted (the `|` would have broken the column count), so "the new parser misreads a corrupt row" is not a regression.

---

## File-by-file changes

### 1. `src/libs/staged.ts`

**Add** two exported helpers near the top of the Helpers section (after `fmEscape`):

```ts
export function escapeTableCell(s: string): string { /* see §Design */ }
export function unescapeTableCell(s: string): string { /* see §Design */ }
```

**Modify** `formatGateResults` at line 102-104:

```ts
for (const g of gates.results) {
  lines.push(`| ${escapeTableCell(g.gate)} | ${g.pass} | ${escapeTableCell(g.reason)} |`);
}
```

(Escape `g.gate` too for defence-in-depth — gate names are controlled and should never contain `|` or `\n`, but escaping costs nothing and closes a future footgun.)

**Modify** `formatJudgeResults` at line 121-124:

```ts
for (const v of judges.votes) {
  const et = v.errorType ?? "substantive";
  lines.push(`| ${escapeTableCell(v.model)} | ${v.vote} | ${escapeTableCell(et)} | ${escapeTableCell(v.reason)} |`);
}
```

**Do not touch** `writeStagedProposal`, `listStagedProposals`, `recordDecision`, `readStagedFile`, or any frontmatter code.

### 2. `src/tools/review-staged.ts`

**Import** the unescape helper at the top of the file:

```ts
import { unescapeTableCell } from "../libs/staged.js";
```

**Modify** `parseGateResultsTable` at line 107-126. The row regex stays exactly the same (it's correct for well-formed tables). Only the capture-group assignment changes:

```ts
while ((m = rowRe.exec(section)) !== null) {
  results.push({
    gate: unescapeTableCell(m[1].trim()),
    pass: m[2].trim() === "true",
    reason: unescapeTableCell(m[3].trim()),
  });
}
```

**Modify** `parseJudgeResultsTable` at line 176-193. Both the 4-col and 3-col branches get the unescape:

```ts
// 4-col branch:
votes.push({
  model: unescapeTableCell(m[1]),
  vote: m[2],
  errorType: unescapeTableCell(m[3]),
  reason: unescapeTableCell(m[4].trim()),
});

// 3-col branch:
votes.push({
  model: unescapeTableCell(m[1]),
  vote: m[2],
  errorType: "substantive",
  reason: unescapeTableCell(m[3].trim()),
});
```

**Display-side decoding.** Find the function that renders a staged proposal to the terminal for `review-staged show` (search for how gates are printed — likely a `renderGates()` or inline block). The `.reason` field is already decoded by the parser, so there is **no additional work on the display side** — as long as the print path uses the parsed `GateSummary.reason` and not the raw section text. Verify this during implementation; if the display path re-slices the raw markdown, change it to use the parsed value. Record the file:line in the PR.

### 3. `src/libs/__tests__/staged-roundtrip.test.ts`

**Add** a third test that pins the gate-reason round-trip. Current tests only check title / proposedChange / reason (frontmatter). Gate reasons in the body have no regression test.

The test must construct a `StagedProposal` with a non-trivial `gateResults.results` array containing:
- A gate with a simple single-line reason (baseline)
- A gate with a multi-line reason (3+ lines, includes the sequence that caused the evidence bug)
- A gate with a reason containing a literal `|`
- A gate with a reason containing a literal `\` followed by `n` (edge case for unescape)

Then call `writeStagedProposal()`, re-read the file via `readFileSync`, manually call `parseGateResultsTable()` from `review-staged.ts` on the body, and assert every input reason appears byte-for-byte in the parsed output.

Note: `listStagedProposals()` currently reconstructs `gateResults` as an empty stub (`src/libs/staged.ts:255`), so the round-trip cannot be verified through `listStagedProposals()` alone. The test imports `parseGateResultsTable` directly from `src/tools/review-staged.ts` to close the loop. This is fine — `parseGateResultsTable` is the real parser that `review-staged show` uses; the stub in `listStagedProposals` exists because the list view doesn't need gates. If importing a tool from a lib test feels wrong, move `parseGateResultsTable` to `src/libs/staged.ts` as part of this task and re-export from `review-staged.ts`. **Recommended: move it.** The parser belongs next to the writer.

**Sub-decision:** Move `parseGateResultsTable` and `parseJudgeResultsTable` from `src/tools/review-staged.ts` to `src/libs/staged.ts`. This co-locates the escape/unescape contract with the writer and parser. `review-staged.ts` imports them. Keeps the escape symmetry in one file.

### 4. `src/tools/__tests__/review-staged.test.ts`

**Add** parse-side unit tests for the new escape behavior:

- `parseGateResultsTable` on a fixture with `\n`-escaped multi-line reasons returns the decoded strings.
- `parseGateResultsTable` on a legacy fixture (no escapes) returns the same values as before — confirming backward compat.
- `parseJudgeResultsTable` 4-col fixture with an escaped `|` in the reason returns the decoded string.
- `parseJudgeResultsTable` 3-col fixture (legacy) is unchanged.

Fixtures live inline in the test file — do not add new files under `__fixtures__/`.

### 5. `vault/studio/projects/autonomous-self-learning/tasks/TASKS.md`

Add two rows to the **Immediate** section:

```md
| ASL-0016 | Fix multi-line gate reason truncation in staged writer | P1 | planned | ASL-0012 | [`2026-04-08-092231-ASL-0016-gate-reason-escape-fix.md`](2026-04-08-092231-ASL-0016-gate-reason-escape-fix.md) |
| ASL-0017 | Backfill legacy 3-col staged files to 4-col format | P3 | planned | ASL-0016 | [`2026-04-08-092231-ASL-0017-legacy-staged-backfill.md`](2026-04-08-092231-ASL-0017-legacy-staged-backfill.md) |
```

Remove the "_none — ASL Phase 2 P1+P2 work complete_" placeholder.

---

## Acceptance criteria

A reviewer (me) signs off on this task only when **all** of the following are verifiable:

1. **Round-trip test passes.** A `StagedProposal` with a 3-line constitution-gate reason, a reason containing `|`, and a reason containing `\n` (literal) flows through `writeStagedProposal` → filesystem → `parseGateResultsTable` → byte-exact equality on every reason.
2. **Judge round-trip test passes.** Same structure, exercising `formatJudgeResults` + `parseJudgeResultsTable` in both 3-col and 4-col modes.
3. **Legacy parse unchanged.** `parseGateResultsTable` on a fixture identical to `20260407-164120-expanded-suno-critique-filter-rules.md` (the 3-col legacy judge file) returns the same output as it does on `main` at the commit that shipped ASL-0012. A snapshot or hard-coded expected-value assertion is fine.
4. **Manual smoke: write a new proposal, read it back.** Ryan runs `bun run tool asl-sync` (or equivalent) on a scratch agent that will produce a multi-line constitution gate reason, opens the written file, and confirms the table row contains `\n` literals. Then runs `bun run tool review-staged show <filename>` and confirms the display shows the multi-line reason with real line breaks. Screenshot or paste the output in the review comment.
5. **Manual smoke: legacy file shows no regression.** `bun run tool review-staged show 20260407-164120-expanded-suno-critique-filter-rules.md` renders identically to what it did before the fix. Paste before/after output.
6. **Documented dead record.** The PR includes one sentence noting that `20260407-164110-refined-multi-stage-critique.md` is still truncated on disk (data loss is historical and irrecoverable) and that `review-staged show` on that file will still show the truncated reason. The fix prevents future occurrences; it does not rewrite history.
7. **No changes to gates, judges, distill pipeline, or frontmatter serialization.** Diff is scoped to the four files above plus TASKS.md.
8. **`bun test` green.** All existing tests still pass. New tests pass. No skipped tests.
9. **`parseGateResultsTable` + `parseJudgeResultsTable` relocated to `src/libs/staged.ts`.** `review-staged.ts` imports them. This is a moved function, not a duplicate — delete the originals.

---

## Risks

| Risk | Likelihood | Blast radius | Mitigation |
|---|---|---|---|
| Unescape regex corrupts a legacy cell that happens to contain `\n` literal sequence | Low | One file's parser output is mangled | Checked all 12 current staged files — no sequence matches. New writes use the new format consistently. |
| `parseGateResultsTable` move to `staged.ts` introduces circular import | Low | Build break | `staged.ts` does not import from `review-staged.ts` today; moving the parser in only deepens existing one-way dependency. Verify with `bun build` during implementation. |
| Escape helper handles `|` but display path still chokes | Low | Cosmetic | Acceptance criterion #4 requires visual verification of `review-staged show` output. |
| Test fixture divergence — `escapeTableCell` changes but fixture doesn't | Low | False-green tests | Test must call `escapeTableCell` via the real code path (round-trip through `writeStagedProposal`), not embed pre-escaped fixture strings. |
| Judge reason with embedded backslash triggers the `\\n` ambiguity | Low | Single parse corruption | Single-pass unescape loop (see §Design) resolves this. Add a test: input `"backslash then n: \\n"` round-trips correctly. |
| Someone reverts the escape in six months because the raw file "looks ugly" | Medium | Silent data loss returns | The round-trip test is the guard. Regression test fails immediately on revert. |

---

## Test plan

### Unit — `src/libs/__tests__/staged-roundtrip.test.ts`

- **Existing tests unchanged.** The two tests in the file today must still pass without modification. They cover frontmatter round-trip; they are orthogonal to this task.
- **New test 1:** `"gate results round-trip preserves multi-line reason"`. Multi-line constitution reason, single-line safety reason, reason with `|` — all byte-exact after round-trip.
- **New test 2:** `"judge results round-trip preserves multi-line reason"`. Same structure for judge votes.
- **New test 3:** `"escape helper handles the backslash-n ambiguity"`. Direct call to `escapeTableCell` / `unescapeTableCell` with input `"literal backslash then n: \\n and actual newline: \nend"`. Assert round-trip equality.

### Unit — `src/tools/__tests__/review-staged.test.ts`

- **Existing `parseGateResultsTable` tests** continue to pass unchanged (legacy compat).
- **New test:** `"parseGateResultsTable decodes escaped newlines and pipes"`. Fixture is a hand-written body string containing the new escaped form. Expected reason is the decoded value.
- **New test:** `"parseJudgeResultsTable 4-col decodes escapes"`. Same.
- **New test:** `"parseJudgeResultsTable 3-col legacy unchanged"`. Hand-written 3-col fixture, assert exactly the same output as on `main`.

### Manual smoke

1. Ryan runs the daemon (or `asl-sync`) against a scratch agent designed to trip the constitution gate with a multi-line rejection.
2. Open the new staged file. Confirm `## Gate Results` table row contains `\n` (literal) in the reason cell.
3. Run `bun run tool review-staged show <file>`. Confirm the displayed reason has real line breaks and nothing is missing.
4. Run `bun run tool review-staged show 20260407-164120-expanded-suno-critique-filter-rules.md`. Confirm output is byte-identical to pre-fix (no regression on legacy format).
5. Run `bun run tool review-staged show 20260407-164110-refined-multi-stage-critique.md`. Confirm output still shows the truncated constitution reason (the dead record) and document this in the PR as expected — the data was lost at write time.

---

## Dependencies

- **ASL-0012** (review-staged CLI) — ships the `parseGateResultsTable` / `parseJudgeResultsTable` functions being modified. Must merge before ASL-0016.
- **ASL-0017** (legacy backfill) — **no dependency**, independent task. ASL-0016 does not require any legacy files to be touched. ASL-0017 may reference the escape helpers this task introduces.

---

## Scope split — why this is ASL-0016, not ASL-0016+0017 bundled

Ryan, read this section before you start.

**Originally framed as a single task.** The scoping request asked whether to bundle the escape fix with a backfill of ~12 legacy 3-column staged files into the 4-column format. Decision: **split into ASL-0016 (this task) and ASL-0017 (separate).**

Rationale:

| Dimension | Bundle | Split (chosen) |
|---|---|---|
| Shippability | Blocked by backfill design debate | Writer fix ships immediately |
| Review surface | Mixed (bug fix + data migration) | Two small focused diffs |
| Rollback | Whole bundle reverts together — backfill might be irreversible | Each task reverts independently |
| Priority | P1 bug diluted by P3 chore | P1 ships now, P3 waits |
| Coupling | Artificially coupled through shared files | Genuinely independent — ASL-0016 touches writer/parser; ASL-0017 touches disk content |

ASL-0016 is a **data-loss bug fix**. It does one thing: stop new writes from corrupting gate reasons. It ships fast, reviews fast, reverts fast.

ASL-0017 is a **tech-debt cleanup** with open questions (what `errorType` value to assign retroactively, whether the win is worth the one-shot script). That debate does not belong on the critical path of the bug fix.

**If during implementation you find the tasks need to be bundled** (e.g., the escape change breaks legacy parsing in a way that requires the backfill), stop, post a TECH BLOCKER, and I will update both task docs. Do not bundle them silently.

---

## Out of scope (explicit)

- Any change to `src/libs/gates.ts` or individual gate implementations.
- Any change to `src/libs/autonomous-distill.ts`.
- Any change to `writeStagedProposal` frontmatter rendering — `fmEscape` / JSON-stringify is correct.
- Any change to `listStagedProposals`' stub `gateResults` — the list view doesn't need full gates.
- Backfilling legacy files (ASL-0017).
- Rescuing the truncated content in the evidence file (impossible).
- Adding a format version number to staged frontmatter. Not needed — the unescape is backward compatible.
- Refactoring the display path in `review-staged show` beyond the minimum needed to use the parsed `reason` field.

---

## Reviewer checklist (McCall)

When I review Ryan's PR:

1. Diff scope matches §File-by-file changes. Nothing extra.
2. `escapeTableCell` / `unescapeTableCell` are defined exactly once, in `staged.ts`, exported.
3. `parseGateResultsTable` + `parseJudgeResultsTable` have been moved to `staged.ts` and re-exported from `review-staged.ts` (or `review-staged.ts` imports directly).
4. The single-pass unescape loop is present — no chained `.replace()` with sentinels.
5. All six acceptance criteria are demonstrated in the PR body with logs/screenshots.
6. `bun test` output is pasted in the PR description, green.
7. TASKS.md updated with both ASL-0016 and ASL-0017 rows.
8. No changes to files outside §File-by-file changes.
9. The dead-record note for `20260407-164110-refined-multi-stage-critique.md` is present.
