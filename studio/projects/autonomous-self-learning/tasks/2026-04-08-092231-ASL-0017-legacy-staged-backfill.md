---
id: ASL-0017
title: Backfill legacy 3-col staged files to 4-col judge format
project-code: ASL
status: in-progress-needs-fix
priority: P3
created: 2026-04-08
author: mccall
implementer: ryan
reviewer: mccall
depends-on: [ASL-0012, ASL-0016]
blocks: []
files:
  - src/tools/migrate-staged-format.ts
  - src/tools/__tests__/migrate-staged-format.test.ts
  - src/tools/review-staged.ts
  - vault/studio/projects/autonomous-self-learning/tasks/TASKS.md
estimated-complexity: small (1-2 hours if kept; 0 hours if deferred)
tags: [task, tech-debt, migration, staged-proposals, asl-phase-2]
---

# ASL-0017 — Backfill legacy 3-col staged files to 4-col judge format

## TL;DR

ASL-0012 added a fourth column (`ErrorType`) to the judge results table in staged proposal files. The writer at `src/libs/staged.ts:119-124` emits 4-column format. `parseJudgeResultsTable` in `src/tools/review-staged.ts:175-193` handles **both** formats for backward compatibility. Today ~12 staged files on disk under `vault/studio/memory/echo/staged/` use the legacy 3-column format; new files use 4-column. Without a migration, this drift grows every time the pipeline stages a new proposal.

**The question on the table is not "how do we backfill" — it's "is the backfill worth doing at all?"** The answer in this doc is **yes, with narrow scope**. The rationale and the reversal clause are below.

---

## Decision: backfill, one-shot, P3

### Options considered

| Option | Description | Pros | Cons |
|---|---|---|---|
| **A: Do nothing** | Leave the dual-format parser in place indefinitely. | Zero risk. | Debt compounds. Every future parser change has to touch both branches. Debugging "why does this file parse differently?" stays on the future-me menu. |
| **B: One-shot backfill script** (CHOSEN) | Write `src/tools/migrate-staged-format.ts`. Rewrites all legacy files in place to 4-col format with `errorType: "substantive"` for every existing row. Simplifies parser afterwards. | Single source of truth on disk. Parser branch can eventually delete (see §Parser cleanup). | `errorType` values are synthesized — they reflect "no judge ever labeled this row with an error type," not a real classification. |
| **C: Delete legacy files** | Move old staged files to archive and start fresh. | Cleanest. | Loses real human-review queue. The 12 files on disk are actual pending decisions waiting on the user. |

### Why B

The parser branch (`hasFourCols` logic at `review-staged.ts:175`) is 20 lines of real complexity that we keep paying for forever. The backfill is a one-time cost that deletes that complexity permanently. The synthesized `errorType: "substantive"` is defensible because:

1. **It matches what `formatJudgeResults` already defaults to** (`src/libs/staged.ts:122`: `const et = v.errorType ?? "substantive"`). The default is already "substantive" anywhere the upstream judge failed to provide one. Backfilling to the same default is internally consistent.
2. **It doesn't misclassify anything.** "substantive" means "this was a normal judge verdict about the content of the proposal," which is exactly what every legacy row is. The only rows that shouldn't be labeled "substantive" are API-error or infrastructure-error rows — and the evidence shows none of the 12 legacy files contain those (verified against `20260407-164120-expanded-suno-critique-filter-rules.md` and its siblings during scoping).
3. **The user can always rewrite a specific file's errorType by hand** if they disagree with the backfill on one row. The migration is not irreversible at the per-row level — the data before migration is preserved in a sibling `archive-pre-migrate/` dir.

### Reversal clause

**If Ryan's implementation reveals that any legacy file contains a judge row that cannot honestly be labeled `errorType: "substantive"`** (e.g., a row whose reason is literally `"API error: Failed to parse JSON"` — which would be `"api_error"`), stop, post a TECH BLOCKER, and I will either:
- narrow the migration to skip those rows, or
- extend the migration with a pattern-based classifier (regex `/API error/` → `"api_error"`), or
- defer the task back to Option A.

The smoke check before starting the migration is: grep every legacy file's judge rows for `"API error"`, `"timeout"`, `"parse"`. If any hits come back, halt and escalate.

**Audit:** During scoping I confirmed `20260407-164120-expanded-suno-critique-filter-rules.md` contains exactly this case at line 44 (`| minimax | reject | API error: Failed to parse JSON |`). So the reversal clause is **guaranteed to trigger on at least one file**. Ryan will hit this and must decide: either classify by pattern, or skip the row. See §Implementation path for the branching logic.

---

## Goals

1. **One-shot migration.** Rewrite all legacy 3-col staged files in `vault/studio/memory/*/staged/` to 4-col format.
2. **Reversible.** Every migrated file is copied to `vault/studio/memory/{agent}/staged/archive-pre-migrate/` before being rewritten. If something goes wrong, the archive is the rollback.
3. **Deletable parser branch.** Once migration runs clean, delete the 3-col branch in `parseJudgeResultsTable` and the `hasFourCols` detection logic. The parser becomes a single path.
4. **No runtime cost.** The migration tool is a one-shot CLI. It is not wired into the daemon, not run on hooks, not auto-invoked anywhere. The user runs it manually, once, after ASL-0016 ships.

## Non-Goals

- Migrating anything other than the `## Judge Results` table. The `## Gate Results` table is already 3-column by design and has no legacy format drift.
- Backfilling `errorType` from log files or external data sources. The synthesized value is fine.
- Re-running judges on legacy proposals. Out of scope.
- Any UX change to `review-staged`. The parser simplification is the only downstream effect.
- Running the migration automatically on any schedule.

---

## Hard locks

- **Archive before modify.** Every legacy file that gets touched has a copy placed at `vault/studio/memory/{agent}/staged/archive-pre-migrate/{filename}` first. No in-place edit without a pre-copy.
- **Dry-run default.** `migrate-staged-format` runs in dry-run mode by default. It prints what it would change. It only writes with `--apply`.
- **Do not touch anything other than judge result rows.** Frontmatter, body, gate results, proposed change, evidence — all bytes other than the judge rows must survive byte-for-byte.
- **Do not modify files outside `staged/`.** No touching `approved/`, `rejected/`, or the agent's knowledge dirs. The scope is pending staged files only.
- **Do not couple to ASL-0016.** The migration runs independently. It uses the `escapeTableCell` helper from ASL-0016 if the writer needs it, but it does not modify gate result tables.
- **No touching archived raw files.** Do not walk `vault/studio/memory/raw/` or anything under `archive/`.

---

## Producer-side audit

Already done in §Decision above. Key facts:

- Current legacy files: 11 files under `vault/studio/memory/echo/staged/` (counted during scoping on 2026-04-08 at `vault/studio/memory/echo/staged/` — excludes `archive-pre-fix`).
- Only one agent (`echo`) has staged files today. Other agents have empty `staged/` dirs or don't exist.
- New files written since ASL-0012's 4-col change will be skipped (the migration detects 4-col header and no-ops).
- One file (`20260407-164120-expanded-suno-critique-filter-rules.md`) contains an API-error row — **this triggers the reversal clause above.**

---

## Implementation path

### 1. `src/tools/migrate-staged-format.ts`

New CLI tool. Wired into the tool runner auto-discovery.

**Behavior:**

```
bun run tool migrate-staged-format                     # dry run, all agents
bun run tool migrate-staged-format --agent echo        # dry run, single agent
bun run tool migrate-staged-format --apply             # write changes
bun run tool migrate-staged-format --apply --agent echo
```

**Algorithm:**

```
for each file in memory/*/staged/*.md:
  parse the ## Judge Results section header row
  if header already has 4 columns (Model | Vote | ErrorType | Reason):
    skip (already migrated)
  if no ## Judge Results section or "No judge panel run.":
    skip
  for each judge row in the section:
    classify errorType:
      if reason matches /API error|timeout|parse/i → "api_error"   (see §Classification)
      else → "substantive"
  rebuild the section with the 4-col header and escaped cells (reusing escapeTableCell from ASL-0016)
  in --apply mode:
    copy original to archive-pre-migrate/
    write new content back
    print: MIGRATED <path>
  in dry-run mode:
    print: WOULD MIGRATE <path> with N rows
```

**Classification rules for `errorType` backfill:**

| Pattern (case-insensitive) on the reason field | errorType |
|---|---|
| `/api error/` | `api_error` |
| `/timeout/` | `api_error` |
| `/failed to parse/` | `api_error` |
| `/rate limit/` | `api_error` |
| (anything else) | `substantive` |

This is a deliberately narrow classifier — any borderline row should end up as `substantive` (the safe default). If Ryan encounters a legacy row whose reason doesn't fit either bucket cleanly, the default is safe.

**Output contract:** CLI mirrors ASL-0009 / ASL-0012 conventions — exit code 0 on success, 1 on error, `--json` flag for structured output.

### 2. `src/tools/__tests__/migrate-staged-format.test.ts`

Tests use fixtures written to a temp dir under `out/migrate-test/`. Do not touch real vault paths.

- **Legacy 3-col file with all-substantive rows** → migrates to 4-col with all `substantive`.
- **Legacy 3-col file with one `"API error"` row** → migrates with that row labeled `api_error`, others `substantive`.
- **Already 4-col file** → no-op, file content byte-identical.
- **"No judge panel run."** → no-op.
- **Dry-run mode** → file content byte-identical to input, prints plan.
- **`--apply` creates archive copy** → archive file exists at `archive-pre-migrate/` with original content.

### 3. `src/tools/review-staged.ts` — parser cleanup

**Only after** Ryan has run the migration on the real vault and all tests pass and `review-staged list` renders correctly against migrated files, delete the 3-col branch:

**Before** (current, lines 171-193):

```ts
const headerLineMatch = section.match(/^\|\s*Model\s*\|([^|\n]+\|)+\s*$/m);
const headerCols = headerLineMatch ? ... : 3;
const hasFourCols = headerCols >= 4;
if (hasFourCols) { /* 4-col parse */ } else { /* 3-col parse */ }
```

**After:**

```ts
// All staged files use the 4-col format: Model | Vote | ErrorType | Reason
// Legacy 3-col files were migrated by src/tools/migrate-staged-format.ts on 2026-04-08.
const rowRe = /^\|\s*(\w+)\s*\|\s*(approve|reject|abstain)\s*\|\s*(\S+)\s*\|\s*(.*?)\s*\|?\s*$/gm;
let m: RegExpExecArray | null;
while ((m = rowRe.exec(section)) !== null) {
  votes.push({
    model: unescapeTableCell(m[1]),
    vote: m[2] as "approve" | "reject" | "abstain",
    errorType: unescapeTableCell(m[3]),
    reason: unescapeTableCell(m[4].trim()),
  });
}
```

**This deletion is the payoff for the whole task.** The parser goes from 20 lines of dual-path logic to 10 lines of single-path logic.

### 4. `vault/studio/projects/autonomous-self-learning/tasks/TASKS.md`

Already updated as part of ASL-0016's TASKS.md change. No additional edit here.

---

## Acceptance criteria

1. **Migration tool exists and is tool-runner-discoverable.** `bun run tool --list` shows `migrate-staged-format`.
2. **Dry-run output is complete and accurate.** Running in dry-run mode on the real vault prints every legacy file that would be migrated, the number of rows per file, and the errorType classification for each row. No file contents change.
3. **`--apply` migrates all legacy files.** After running with `--apply`:
   - Every file formerly in 3-col format is now in 4-col format.
   - Every migrated file has an archive copy in `vault/studio/memory/{agent}/staged/archive-pre-migrate/{filename}` with the pre-migration content, byte-identical to the original.
   - No non-judge content in any file has changed. A diff between archive and current should show only the judge table region.
4. **Parser cleanup runs green.** After deletion of the 3-col branch in `parseJudgeResultsTable`, `bun test` is green, `review-staged list` shows all proposals correctly, `review-staged show` on every migrated file renders without error.
5. **Unit tests pass.** All new tests in `migrate-staged-format.test.ts` pass. All existing `review-staged.test.ts` tests pass (the 3-col tests must be updated or replaced — they now test migrated content).
6. **Manual verification.** Ryan pastes the output of `bun run tool review-staged list` before and after the migration. They must show the same proposal count and the same titles. The recommendation column may change because `errorType` is now populated — document any such change in the PR.
7. **Reversibility demonstrated.** Ryan shows (via a single-file rollback test) that copying a file from `archive-pre-migrate/` back over the migrated version restores the original bytes exactly.
8. **API-error row handled.** `20260407-164120-expanded-suno-critique-filter-rules.md` migrates with the `minimax` row labeled `api_error`, not `substantive`. This is verified in the PR body.

---

## Risks

| Risk | Likelihood | Blast radius | Mitigation |
|---|---|---|---|
| Migration corrupts a file body outside the judge section | Low | One file mangled, archive copy exists for rollback | Archive-before-modify is mandatory. Byte-diff assertion in the test suite. |
| Classifier mislabels a borderline row | Medium | One or two rows get `substantive` when they should be `api_error` or vice versa | Narrow classifier, safe default. User can hand-edit if wrong. |
| Parser cleanup breaks a legacy file that slipped through the migration | Low | `review-staged show` errors on that file | Acceptance criterion #4 requires showing every staged file after cleanup. If any fail, revert the parser cleanup, re-run migration, retry. |
| Archive dir gets indexed as a real staged directory by `listStagedProposals` | Medium | Phantom entries in `review-staged list` | `listStagedProposals` already scans `staged/` recursively (`readdirSync` on `stagedDir`). **Verify during implementation** whether it descends into subdirs. If yes, either (a) name the archive dir with a leading dot (`.archive-pre-migrate/`) so it's hidden, or (b) explicitly filter it out in `listStagedProposals`. The dot-prefix is simpler. |
| Someone runs the migration twice | Low | Idempotent no-op on second run (4-col detection) | The header-count check already covers this. |
| ASL-0016 hasn't shipped yet and the escape helpers don't exist | High if tasks run out of order | Migration tool can't compile | Hard dependency on ASL-0016 declared in frontmatter. Ryan does not start ASL-0017 until ASL-0016 is merged. |

---

## Dependencies

- **ASL-0012** — source of the 4-col writer and the dual-format parser this task cleans up.
- **ASL-0016** — source of the `escapeTableCell` / `unescapeTableCell` helpers that the migration tool imports. **Must be merged before ASL-0017 starts.**

---

## Scope boundary with ASL-0016

ASL-0016 ships the writer + parser fix for gate reason truncation. It does NOT touch any file on disk. Zero data migration.

ASL-0017 is the data migration. It touches files on disk. It does NOT modify the writer or the gate reason parser.

**These are orthogonal changes and must not be bundled.** If during ASL-0017 implementation you find yourself editing `formatGateResults`, `formatJudgeResults`, `escapeTableCell`, `unescapeTableCell`, `parseGateResultsTable`, or any writer-side code, stop — that work belongs in ASL-0016 or a follow-up.

---

## Out of scope (explicit)

- Re-running judges on legacy proposals.
- Inferring real errorType values from the original judge log (the logs may not exist).
- Migrating anything in `approved/` or `rejected/` — only pending files in `staged/`.
- Any change to `recordDecision`, `applySoulChange`, or the human-in-the-loop flow.
- Any change to `autonomous-distill.ts` or the gate/judge runners.
- Running the migration automatically on a hook, cron, or the daemon startup.

---

## Reviewer checklist (McCall)

1. Dry-run output is attached in the PR body.
2. `archive-pre-migrate/` is visible in the vault after `--apply`, and its contents byte-match the original files.
3. The parser cleanup commit is separate from (or at least clearly demarcated within) the migration commit, so the cleanup can be reverted alone if needed.
4. `listStagedProposals` does not pick up archive entries (confirmed via list output).
5. The `minimax` API-error row in the suno-critique file is labeled `api_error`, not `substantive`.
6. All acceptance criteria have logs or screenshots in the PR body.
7. `bun test` output is green and pasted.
8. TASKS.md move of ASL-0016 and ASL-0017 from Immediate to Shipped is handled by Freddie at commit time, not by Ryan.

---

## Review Findings — 2026-04-08 (paused mid-fix)

Ryan implemented the migration tool (`src/tools/migrate-staged-format.ts`, 638 lines + 639 lines of tests). 30/30 tests pass. Dry-run smoke verified — correctly identifies 2 files to migrate, correctly classifies the minimax `"API error: Failed to parse JSON"` row as `api_error`, vault untouched. McCall reviewed and gave **PASS with 2 NEEDS-FIX items + 1 DISCUSS**, then a planned Ryan dispatch hit consistent Anthropic API 529 overload errors and we paused for the day.

**State on disk (untracked, not committed):**
- `src/tools/migrate-staged-format.ts`
- `src/tools/__tests__/migrate-staged-format.test.ts`

**Vault state:** clean, no `--apply` has run.

### NEEDS-FIX #1 — Architectural: third parser created instead of reusing the canonical one

**Locations:** `src/tools/migrate-staged-format.ts:99-183` (`parse3ColJudgeSection`) and `:246-285` (`verifyRoundTrip` inline regex).

**Problem:** The canonical `parseJudgeResultsTable` in `src/libs/staged.ts:236` already handles BOTH 3-col and 4-col formats. Ryan reimplemented both branches in the migration tool. The whole point of ASL-0017 is parser consolidation — adding a third parser defeats the task's payoff.

**Fix (Option A — smaller diff):**
- Keep `parse3ColJudgeSection` ONLY for offset/preamble extraction (which character positions bound the section).
- Call `parseJudgeResultsTable(content)` for the actual data extraction.
- Override `errorType` post-parse: `parseJudgeResultsTable` hardcodes `"substantive"` for 3-col rows at `staged.ts:295` because it has no other information at parse time. Apply `classifyErrorType(reason)` on each vote AFTER parsing to upgrade matches.
- Same fix for `verifyRoundTrip`: replace the inline 4-col row regex at lines 260-269 with `parseJudgeResultsTable(newContent)` and compare `.votes`.

### NEEDS-FIX #2 — `formatJudgeResults` not exported, so writer was reimplemented

**Location:** `src/tools/migrate-staged-format.ts:201-228` (`rebuild4ColJudgeSection`).

**Problem:** `formatJudgeResults` exists at `src/libs/staged.ts:160` but is NOT exported, so Ryan reimplemented the table writer. If anyone changes column widths, alignment, or escape behavior in `formatJudgeResults`, migrated files will diverge from newly-written files.

**Fix:**
- Add `export` to `formatJudgeResults` in `staged.ts:160` (one-line change).
- Replace `rebuild4ColJudgeSection` body with a call to `formatJudgeResults(judgePanelResult)` where `judgePanelResult` is constructed from the parsed data + classified errorTypes. The `JudgePanelResult` shape is in `staged.ts` — read it for the field list.

NF-1 and NF-2 are paired — fix together.

### DISCUSS-1 (recommend FIX) — Archive dir collides with `listStagedProposals`

**Location:** `src/tools/migrate-staged-format.ts:48` (or wherever the constant lives).

**Problem:** Archive dir is currently `archive-pre-migrate/` (no leading dot). McCall flagged that `listStagedProposals` in `staged.ts:390` may pick up the archive directory as if it contained staged proposals. The migration scanner itself filters to top-level `.md` files only so it won't recurse, but the consumer-side `listStagedProposals` was not verified.

**Fix:** Rename to `.archive-pre-migrate/` (Unix-conventional hidden prefix that scanner code typically skips). Update test fixtures.

### Resume sequence

1. Dispatch Ryan with the three fixes above.
2. Ryan: refactor + rerun tests (expect 30/30) + rerun dry-run smoke (expect identical output, api_error still classified).
3. McCall: re-review (focus on NF-1/NF-2 fix + nothing else regressed).
4. Freddie: commit the migration tool.
5. Freddie: run `--apply` against the real vault (this is the actual data migration — verify with `git diff vault/studio/memory/` and `bun run tool review-staged list` before/after).
6. Freddie: commit the migrated staged files.
7. Ryan: parser cleanup commit per §3 — remove the dual-path branch from `parseJudgeResultsTable` in `staged.ts:268-299` now that all files are 4-col. This is the final payoff commit.
8. McCall: review the cleanup.
9. Freddie: commit the cleanup, move ASL-0017 from Immediate to Shipped.

### Other notes from McCall's review

- **AC #4 (parser cleanup) is deferred to step 7.** Don't try to do it inside the migration tool commit.
- **Pre-existing unrelated test failure:** `normalizeGenre 'jazzhop'` in `shared-modules.test.ts` — completely unrelated to ASL-0017 (genre normalizer in prompt-builder). Track separately, do not let it block.
- **Daemon liveness check:** NOT required by the task doc. Ryan correctly omitted it. Stop the ASL daemon manually before running `--apply` if it's running (it's not, per /morning).
