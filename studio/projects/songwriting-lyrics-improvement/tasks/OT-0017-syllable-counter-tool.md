---
title: "Syllable Counter Tool"
type: task
project: songwriting-lyrics-improvement
status: done
created: 2026-04-03
author: mccall
architect: "vault/studio/projects/songwriting-lyrics-improvement/ARCHITECT.md"
group: 2
parallel: true
depends_on: ["OT-0016"]
---

# OT-0017: Syllable Counter Tool

## What to Build

Implement the T1 syllable counter at `src/libs/lyrics-analyzer/tools/syllable-counter.ts`.
This tool returns exact syllable counts per word and per line, with language-aware
resolution: English via CMU dict (and optionally `syllables` npm), Tagalog via
vowel-group heuristic, unknown via heuristic fallback.

**Phoneme resolution chain per word:**
1. Is it a contraction? Normalize via `contraction-handler.ts`, then proceed
2. In CMU dict? -> source: "cmu", extract syllable count from stress markers
3. `phonemize` G2P available? -> source: "phonemize", extract from G2P output
4. Neither -> vowel-group heuristic, source: "heuristic", flagged `[approx]`

For Tagalog-detected words: skip CMU/phonemize, go directly to heuristic.

**npm dependency handling (ADR-4):**
- Try dynamic import of `phonemize` and/or `syllables` packages
- Wrap in try/catch. If import fails, fall back to heuristic
- Run a quick spike first: `bun add phonemize syllables`, test 100 words
- If packages don't work under Bun, proceed with CMU + heuristic only

**Per-run word cache:**
- Cache phoneme lookups within a single `countSyllables()` call scope
- Same word appears multiple times in a song -- don't re-lookup
- Cache is a plain `Map<string, { syllables: number; source: string }>` passed through

**Export function signature:**
```typescript
export function countSyllables(input: ToolInput): SyllableReport;
```

**Report type:**
```typescript
interface SyllableReport extends ToolReport {
  tool: "syllable-counter";
  perLine: Array<{
    lineNumber: number;
    text: string;
    syllableCount: number;
    perWord: Array<{
      word: string;
      syllables: number;
      source: "cmu" | "phonemize" | "heuristic";
    }>;
  }>;
  perSection: Array<{
    section: string;
    avgSyllablesPerLine: number;
    totalSyllables: number;
  }>;
}
```

**Status rollup:**
- No flags needed from syllable counter itself (it reports counts, not issues)
- Status always "pass" unless tool errors
- Downstream tools (stress-mapper, section-balance) consume these counts to generate flags

## Acceptance Criteria

- [ ] AC-1: `countSyllables()` accepts `ToolInput` and returns `SyllableReport`
- [ ] AC-2: English word "walking" returns syllables: 2, source: "cmu"
- [ ] AC-3: English word "xylophone" returns syllables from CMU or phonemize, not heuristic
- [ ] AC-4: Contraction "don't" returns syllables: 1 (single token, never expanded)
- [ ] AC-5: Tagalog word "maganda" returns syllables: 3 via heuristic (ma-gan-da)
- [ ] AC-6: Unknown word "asdfghjkl" returns syllables via heuristic with source: "heuristic"
- [ ] AC-7: Per-line output includes all words with individual counts
- [ ] AC-8: Per-section aggregation computes avgSyllablesPerLine and totalSyllables
- [ ] AC-9: Same word appearing 5 times only triggers 1 phoneme lookup (cache hit for 4)
- [ ] AC-10: Existing `syllable-heuristic.ts` test cases all pass through this tool (no regression in heuristic path)

Maps to PRD: US-1 (AC-1.1 through AC-1.8), US-16 (AC-16.3, AC-16.4)

## Files in Scope

- `src/libs/lyrics-analyzer/tools/syllable-counter.ts` -- **create**
- `src/libs/lyrics-analyzer/shared/*` -- **read-only** (import from OT-0016)
- `src/libs/prompt-data/syllable-heuristic.ts` -- **read-only reference** (port heuristic logic, do not modify)
- `package.json` -- **modify** (add `phonemize` and `syllables` if spike passes)

## Architectural Constraints

- Import all shared types from `../types.js`
- Import shared utils from `../shared/*.js`
- Port the vowel-group heuristic from `src/libs/prompt-data/syllable-heuristic.ts` as an internal fallback function. Do NOT import from the old file -- copy the logic so the old file can be safely deleted later
- Phonemize/syllables are optional: `try { const pkg = await import("phonemize"); } catch { /* fallback */ }`
- Function must be synchronous in the hot path. If phonemize requires async, resolve during init

## Out of Scope

- Stress pattern analysis (OT-0018)
- Rhyme detection (OT-0019)
- CLI entry point (OT-0021)
- Deletion of `syllable-heuristic.ts` (OT-0022)
- Flagging/severity logic for syllable count deviations (that's section-balance, OT-0020)

## Test Plan

```bash
# Spike: test npm packages under Bun
bun add phonemize syllables
bun -e "import { syllable } from 'syllables'; console.log(syllable('walking'))"

# Smoke test the tool
bun -e "
import { countSyllables } from './src/libs/lyrics-analyzer/tools/syllable-counter.js';
// Build a minimal ToolInput with parsed sections
// Test: English, Tagalog, contraction, unknown word
"
```

Verify against known values:
- "walking" = 2, "beautiful" = 3, "don't" = 1, "maganda" = 3, "a" = 1

## Notes

- The heuristic from `syllable-heuristic.ts` is a port from Python. It works for English. For Tagalog, it should work even better because Tagalog has 5 vowels, no silent letters, and regular syllable structure.
- If both `phonemize` AND `syllables` npm packages fail under Bun, that's fine. CMU dict covers 134k words. Heuristic covers the rest. Document the spike result.