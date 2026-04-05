---
title: "Repetition Analyzer"
type: task
project: songwriting-lyrics-improvement
status: done
created: 2026-04-03
author: mccall
architect: "vault/studio/projects/songwriting-lyrics-improvement/ARCHITECT.md"
group: 5
parallel: true
depends_on: ["OT-0022"]
---

# OT-0024: Repetition Analyzer

## What to Build

Implement the T2 repetition analyzer at
`src/libs/lyrics-analyzer/tools/repetition-analyzer.ts`. This tool detects intentional
repetition (hooks, refrains, callbacks, anaphora) and unintentional repetition
(overused words/phrases).

**Exact-match repeated phrases:**
- Sliding window of 2-6 word n-grams across all lines
- Normalize: lowercase, strip punctuation
- Track locations (section + line number)
- Mark as "structural" if they appear in chorus/hook sections

**Near-match phrases:**
- Compare all 3+ word phrases using Levenshtein word-level similarity
- Threshold: similarity > 0.7 (70% of words match)
- These are potential callbacks/variations

**Anaphora detection:**
- Group lines by their opening 1-3 words
- 3+ lines starting with same pattern = anaphora
- Label as intentional technique

**Repetition ratio per section:**
- (repeated word occurrences / total words)
- Flag sections with ratio > 0.4 that are NOT chorus/hook type
- Chorus/hook repetition is expected and not flagged

**Export:**
```typescript
export function analyzeRepetition(input: ToolInput): RepetitionReport;

interface RepetitionReport extends ToolReport {
  tool: "repetition-analyzer";
  exactRepeats: Array<{
    phrase: string;
    occurrences: number;
    locations: Array<{ section: string; line: number }>;
    structural: boolean;
  }>;
  nearMatches: Array<{
    phraseA: string;
    phraseB: string;
    similarity: number;
    locations: Array<{ section: string; line: number }>;
  }>;
  anaphora: Array<{
    pattern: string;
    lines: number[];
    section: string;
  }>;
  perSection: Array<{
    section: string;
    repetitionRatio: number;
    isStructural: boolean;
  }>;
}
```

## Acceptance Criteria

- [ ] AC-1: Exact phrase "walking in the rain" appearing 3 times detected with all locations
- [ ] AC-2: Chorus repetition marked as `structural: true`
- [ ] AC-3: Near-match "walking in the rain" / "walking through the rain" detected (similarity > 0.7)
- [ ] AC-4: Anaphora: 3 lines starting with "Every time I" detected and labeled
- [ ] AC-5: Verse with repetition ratio > 0.4 flagged (non-chorus)
- [ ] AC-6: Chorus with repetition ratio > 0.4 NOT flagged (expected structural repetition)
- [ ] AC-7: Advisory only -- all flags are informational, never blocking

Maps to PRD: US-7 (AC-7.1 through AC-7.6)

## Files in Scope

- `src/libs/lyrics-analyzer/tools/repetition-analyzer.ts` -- **create**
- `src/libs/lyrics-analyzer/types.ts` -- **read-only**

## Architectural Constraints

- Levenshtein similarity is word-level, not character-level. Split phrases into word arrays, compare
- No external NLP dependencies. Simple string comparison
- N-gram window: 2-6 words. Below 2 = too noisy (common word pairs). Above 6 = unlikely to repeat exactly
- This tool does NOT use phoneme data. It operates on normalized text only

## Out of Scope

- Rhyme scheme (OT-0025)
- CLI/report update (OT-0026)
- Sensory anchor detection (US-15 AC-15.5, Could Have, deferred)

## Test Plan

```bash
bun -e "
import { analyzeRepetition } from './src/libs/lyrics-analyzer/tools/repetition-analyzer.js';
// Test with lyrics that have:
// - Repeated chorus phrase
// - Near-match callback in verse 2
// - Anaphora in bridge
// - Overused phrase in a verse (non-structural)
"
```

## Notes

- Repetition in lyrics is complex: choruses SHOULD repeat, hooks SHOULD repeat, anaphora is a technique. The tool must distinguish structural from unintentional. The `isStructural` flag on sections and the chorus/hook type detection are key.
- Word-level Levenshtein: implement inline. It's a simple dynamic programming algorithm on word arrays. No npm package needed.