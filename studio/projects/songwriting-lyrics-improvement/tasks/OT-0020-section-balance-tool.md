---
title: "Section Balance Tool"
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

# OT-0020: Section Balance Tool

## What to Build

Implement the T1 section balance analyzer at
`src/libs/lyrics-analyzer/tools/section-balance.ts`. This tool reports structural
metrics per section and flags imbalances.

**Per-section metrics:**
- Word count
- Line count
- Unique word count
- TTR (type-token ratio): unique words / total words
- Average syllables per line (from `WordToken.syllables`)
- Total syllables

**Cross-section comparison:**
- Compare each section against the average of its section type (e.g., Verse 1 vs Verse 2)
- Flag sections deviating >50% from section-type average on any metric

**Genre target comparison:**
- Genre density targets come from `ToolInput.options.genreTargets` (optional)
- Default targets if not provided:

| Section Type | Lines | Syllables/Line |
|-------------|-------|----------------|
| Verse | 4-8 | 7-12 |
| Chorus | 2-6 | 6-10 |
| Bridge | 2-4 | 6-10 |
| Pre-Chorus | 2-4 | 6-10 |

- Chorus compression: choruses exceeding 6 lines flagged (research: 2-4 optimal for Suno)

**Flagging:**
- Section deviates >50% from type average -> severity: medium
- Chorus > 6 lines -> severity: medium
- Section deviates >75% -> severity: high
- Genre target out of range -> severity: low (advisory)

**Export:**
```typescript
export function analyzeBalance(input: ToolInput): SectionBalanceReport;

interface SectionBalanceReport extends ToolReport {
  tool: "section-balance";
  perSection: Array<{
    section: string;
    type: SectionType;
    wordCount: number;
    lineCount: number;
    uniqueWords: number;
    ttr: number;
    avgSyllablesPerLine: number;
    totalSyllables: number;
  }>;
  crossSectionFlags: Array<{
    sections: [string, string];
    metric: string;
    deviation: number;
  }>;
  genreTargetFlags: Array<{
    section: string;
    metric: string;
    actual: number;
    target: [number, number];
  }>;
}
```

## Acceptance Criteria

- [ ] AC-1: Parses section headers including aliased names (`[Refrain]` = chorus)
- [ ] AC-2: Per-section metrics computed: word count, line count, unique words, TTR, avg syllables/line, total syllables
- [ ] AC-3: Cross-section comparison: Verse 2 twice as long as Verse 1 flagged
- [ ] AC-4: Genre target comparison works with default targets
- [ ] AC-5: Chorus with 7+ lines flagged severity: medium
- [ ] AC-6: Language-agnostic: works identically for English, Tagalog, and Taglish
- [ ] AC-7: TTR calculated correctly: unique(words) / total(words)
- [ ] AC-8: Section with 0 lyric lines (metadata-only) handled gracefully

Maps to PRD: US-4 (AC-4.1 through AC-4.6)

## Files in Scope

- `src/libs/lyrics-analyzer/tools/section-balance.ts` -- **create**
- `src/libs/lyrics-analyzer/shared/section-parser.ts` -- **read-only** (uses parsed sections from input)
- `src/libs/lyrics-analyzer/types.ts` -- **read-only**

## Architectural Constraints

- This tool operates on tokenized text, NOT phonetics. It is the most language-agnostic tool
- Genre targets are optional input, not hardcoded. Defaults are fallbacks
- No imports from `phoneme-utils.ts` -- this tool doesn't need phonemes

## Out of Scope

- Genre target loading from brain.db (workflow responsibility)
- Repetition analysis (OT-0024)
- CLI entry point (OT-0021)

## Test Plan

```bash
bun -e "
import { analyzeBalance } from './src/libs/lyrics-analyzer/tools/section-balance.js';
// Test with a song that has:
// - Verse 1 (4 lines), Verse 2 (8 lines) -- should flag cross-section deviation
// - Chorus (7 lines) -- should flag compression warning
"
```

## Notes

- This is the simplest of the T1 tools. No phoneme lookups, no stress analysis. Pure counting and comparison.
- TTR (type-token ratio) is a standard linguistic metric. Values close to 1.0 = highly varied vocabulary. Values close to 0.0 = heavy repetition. For lyrics, chorus TTR is naturally low (intentional repetition).