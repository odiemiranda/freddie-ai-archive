---
title: "Singability, Prosody, and Phase 2 CLI Update"
type: task
project: songwriting-lyrics-improvement
status: done
created: 2026-04-03
author: mccall
architect: "vault/studio/projects/songwriting-lyrics-improvement/ARCHITECT.md"
group: 7
parallel: false
depends_on: ["OT-0025"]
---

# OT-0026: Singability, Prosody, and Phase 2 CLI Update

## What to Build

Three deliverables in one task: two T3 advisory tools and the CLI/report/index updates
to wire all Phase 2 tools into the system.

### Part A: Singability Score (`src/libs/lyrics-analyzer/tools/singability-score.ts`)

Composite 0-100 advisory score predicting vocal singability:

```typescript
export function scoreSingability(
  input: ToolInput,
  syllableReport: SyllableReport,
  stressReport: StressReport,
  densityReport?: VowelConsonantDensityReport
): SingabilityReport;

interface SingabilityReport extends ToolReport {
  tool: "singability-score";
  composite: number;
  subScores: {
    lineLengthAlignment: number;   // how well line syllable counts match genre target
    vowelPlacement: number;        // open vowels at line endings (sustain points)
    breathPoints: number;          // natural pause points (line breaks, punctuation, commas)
    consonantDensity: number;      // inverse of harsh consonant cluster density
    stressRegularity: number;      // regularity of S/W pattern
  };
  weights: Record<string, number>;
  calibrated: boolean;             // false until empirical calibration
}
```

**Sub-score calculations:**
- lineLengthAlignment: 100 if all lines within genre target range, proportionally less for deviations
- vowelPlacement: % of line-ending words that end in open vowels
- breathPoints: presence of natural pauses every 8-12 syllables
- consonantDensity: inverse of harsh cluster frequency (from density report if available)
- stressRegularity: % of lines with consistent metrical pattern

**Default weights (pre-calibration):**
- lineLengthAlignment: 0.25
- vowelPlacement: 0.20
- breathPoints: 0.20
- consonantDensity: 0.15
- stressRegularity: 0.20

**Composite:** Weighted sum of sub-scores, scaled to 0-100.

ADVISORY ONLY. `calibrated: false` until empirical data from reflect workflow.

### Part B: Prosody Alignment (`src/libs/lyrics-analyzer/tools/prosody-alignment.ts`)

```typescript
export function checkProsody(
  input: ToolInput,
  stressReport: StressReport
): ProsodyReport;

interface ProsodyReport extends ToolReport {
  tool: "prosody-alignment";
  perLine: Array<{
    lineNumber: number;
    text: string;
    metricalGrid: string;
    violations: Array<{
      position: number;
      word: string;
      wordType: "content" | "function";
      gridPosition: "strong" | "weak";
      issue: string;
    }>;
  }>;
  targetMeter: "iambic" | "trochaic";
  strictness: number;
}
```

**Algorithm:**
1. Build alternating S/W metrical grid (iambic default: W S W S W S...)
2. Map each syllable to grid position
3. Classify words: content (nouns, verbs, adjectives, adverbs) vs function (articles, prepositions, pronouns, conjunctions)
4. Flag: content word syllable on weak grid position, function word syllable on strong grid position
5. Only flag lines with >= `strictness` violations (default: 2)

**POS tagging:** Simple heuristic, not full NLP. Maintain a function-word list. Everything not in the list is "content". This will have false positives but keeps dependencies at zero.

ADVISORY ONLY. High false-positive rate without melody knowledge (R3).

### Part C: Update report.ts, index.ts, CLI

1. **`report.ts`** -- add T2/T3 tool reports to the aggregator. No logic change, just handle the new tool names in status rollup.

2. **`index.ts`** -- add orchestration for T2/T3 tools:
   - After T1 tools run, run T2 tools: density, repetition, rhyme-scheme (passes rhyme-detector output)
   - After T2 tools, run T3 tools: singability (passes syllable + stress + density reports), prosody (passes stress report)
   - All new tools gated by `options.tools` filter (if specified, only run requested tools)
   - Re-export new tool functions

3. **`lyrics-analyze.ts` CLI** -- add sub-commands:
   ```
   bun run tool lyrics-analyze density --file path.md
   bun run tool lyrics-analyze repetition --file path.md
   bun run tool lyrics-analyze scheme --file path.md
   bun run tool lyrics-analyze singability --file path.md
   bun run tool lyrics-analyze prosody --file path.md
   ```

## Acceptance Criteria

- [ ] AC-1: Singability score returns 0-100 composite with 5 sub-scores
- [ ] AC-2: `calibrated: false` in singability report (pre-calibration)
- [ ] AC-3: Prosody alignment flags content word on weak beat position
- [ ] AC-4: Prosody strictness configurable. Default 2 = only flag lines with 2+ violations
- [ ] AC-5: Both tools are ADVISORY ONLY -- status never "fail", maximum "warn"
- [ ] AC-6: `bun run tool lyrics-analyze --file path.md` now runs all 9 tools
- [ ] AC-7: `bun run tool lyrics-analyze density --file path.md` runs only density
- [ ] AC-8: `bun run tool lyrics-analyze --file path.md --tools syllable,singability` runs only those two
- [ ] AC-9: JSON output includes all 9 tool reports
- [ ] AC-10: Execution time still < 500ms for ~50 line song with all 9 tools

Maps to PRD: US-8 (AC-8.1 through AC-8.6), US-9 (AC-9.1 through AC-9.6), US-10 (updated)

## Files in Scope

- `src/libs/lyrics-analyzer/tools/singability-score.ts` -- **create**
- `src/libs/lyrics-analyzer/tools/prosody-alignment.ts` -- **create**
- `src/libs/lyrics-analyzer/report.ts` -- **modify** (handle new tools)
- `src/libs/lyrics-analyzer/index.ts` -- **modify** (add T2/T3 orchestration + re-exports)
- `src/tools/lyrics-analyze.ts` -- **modify** (add sub-commands)

## Architectural Constraints

- T3 tools are advisory: their status can be "pass" or "warn" but NEVER "fail"
- Enforce in the tool: cap `status` at "warn" regardless of flag severity
- Singability weights are constants at top of file, not hardcoded in formula
- Prosody function-word list: reuse the list from stress-mapper (OT-0018)
- Tool execution order in `index.ts` must respect dependencies: T1 first, then T2, then T3

## Out of Scope

- Empirical calibration of singability weights (ongoing, via reflect workflow)
- Workflow integration (Phase 3)
- Melody-aware prosody (out of scope entirely)

## Test Plan

```bash
# Full 9-tool run
bun run tool lyrics-analyze --file out/test-lyrics.md --json

# Singability sub-command
bun run tool lyrics-analyze singability --text "[Verse 1]\nWalking through the morning light\nShadows dancing left and right"

# Prosody sub-command
bun run tool lyrics-analyze prosody --text "The cat sat on the mat" --json

# Performance with all 9 tools
time bun run tool lyrics-analyze --file <50-line-file>
```

## Notes

- Singability is the highest-risk tool (R2). Ship it as advisory, collect data, iterate. The `calibrated: false` flag is a signal to consumers that the score is not yet validated.
- Prosody without melody is inherently imprecise. The strictness threshold (default 2) reduces false positives. Users who find it noisy can set strictness higher.
- This is the last Phase 2 task. After this, all 9 tools are operational.