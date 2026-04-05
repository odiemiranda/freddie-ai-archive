---
title: "Rhyme Detector Tool"
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

# OT-0019: Rhyme Detector Tool

## What to Build

Implement the T1 rhyme detector at `src/libs/lyrics-analyzer/tools/rhyme-detector.ts`.
This tool detects end rhymes and internal rhymes, classifies type, and scores quality
using weighted ARPAbet phoneme comparison.

**Rhyme scoring (ADR-6):**

| Component | Weight | Measures |
|-----------|--------|----------|
| Stressed vowel match | 65% | Same stressed vowel = highest weight |
| Tail consonant similarity | 25% | Consonants after stressed vowel |
| Onset similarity | 15% | Consonants before stressed vowel. For perfect/slant: scored as onset DISSIMILARITY (different onsets = better rhyme). For assonance/alliteration: scored as onset SIMILARITY |

**Classification thresholds:**
- `score >= 0.85` -> perfect
- `score >= 0.6` -> slant
- `score >= 0.4` (vowel match, consonant miss) -> assonance
- `score >= 0.4` (consonant match, vowel miss) -> consonance
- Same word or homophone -> identity (flagged "lazy rhyme", severity medium)
- Multisyllabic with 2+ syllable match -> mosaic
- `score < 0.3` -> flagged "weak rhyme", severity low

**End rhyme detection:**
- Compare last stressed vowel + tail of each line against other lines in same section
- Lines within same section compared pairwise

**Internal rhyme detection:**
- For each line, compare mid-line words (not first, not last) against each other
- Only flag if score >= 0.4 (skip noise)

**Tagalog rhyme:**
- Tagalog rhyme is primarily vowel-based
- For Tagalog words with no phonemes (heuristic source), compare last 2 characters as a rough vowel check
- Score Tagalog-Tagalog pairs separately, with vowel match weighted at 80% (no consonant tail data)

**Export:**
```typescript
export function detectRhymes(input: ToolInput): RhymeReport;

interface RhymeReport extends ToolReport {
  tool: "rhyme-detector";
  endRhymes: Array<{
    lineA: number;
    lineB: number;
    wordA: string;
    wordB: string;
    type: "perfect" | "slant" | "assonance" | "consonance" | "mosaic" | "identity";
    score: number;
    section: string;
  }>;
  internalRhymes: Array<{
    line: number;
    wordA: string;
    wordB: string;
    positionA: number;
    positionB: number;
    type: "perfect" | "slant" | "assonance" | "consonance" | "mosaic";
    score: number;
  }>;
  perSection: Array<{
    section: string;
    endRhymeCount: number;
    internalRhymeCount: number;
    internalRhymeDensity: number;
  }>;
}
```

## Acceptance Criteria

- [ ] AC-1: "cat" / "hat" scores >= 0.85, classified as perfect rhyme
- [ ] AC-2: "cat" / "bed" scores < 0.3, flagged as weak rhyme
- [ ] AC-3: "cat" / "cat" classified as identity, flagged "lazy rhyme" severity medium
- [ ] AC-4: "time" / "mine" detected as end rhyme between consecutive lines
- [ ] AC-5: Internal rhyme detected within a single line (e.g., "I find the light inside my mind")
- [ ] AC-6: Per-section internal rhyme density calculated (count / total word pairs)
- [ ] AC-7: Tagalog word pairs compared via vowel-based scoring when no phonemes available
- [ ] AC-8: Mosaic rhyme detected for multisyllabic matches (e.g., "master" / "disaster")
- [ ] AC-9: Scores are 0.0-1.0 range, never outside

Maps to PRD: US-3 (AC-3.1 through AC-3.7)

## Files in Scope

- `src/libs/lyrics-analyzer/tools/rhyme-detector.ts` -- **create**
- `src/libs/lyrics-analyzer/shared/phoneme-utils.ts` -- **read-only** (import `getPhonemeSequence`, `classifyPhoneme`)

## Architectural Constraints

- Rhyme scoring is pure computation. No external API calls
- All thresholds should be constants at top of file for easy tuning (R14)
- The onset dissimilarity refinement from Gemini council: when classifying perfect/slant, different onsets are BETTER (indicates true rhyme, not identity). When classifying assonance/alliteration, similar onsets are better

## Out of Scope

- Rhyme scheme labeling (AABB/ABAB -- that's OT-0025)
- CLI entry point (OT-0021)
- Integration with workflow critics

## Test Plan

Test with known rhyme pairs:
- Perfect: cat/hat, light/night, moon/soon
- Slant: cat/bed (should be below slant), time/line (should be slant or perfect)
- Identity: love/love
- Internal: "I find the light inside my mind" -- light/mind detected

```bash
bun -e "
import { detectRhymes } from './src/libs/lyrics-analyzer/tools/rhyme-detector.js';
// Build ToolInput with pre-parsed section containing rhyming couplets
"
```

## Notes

- The 65/25/15 weighting comes from RHYME-CTRL research. It's a starting point. R14 acknowledges tuning may be needed.
- Internal rhyme detection is computationally more expensive (O(n^2) per line). For a typical song line of 8-12 words, this is ~36-66 comparisons per line. Acceptable.
- Homophone detection for identity rhymes: compare normalized spellings. "there"/"their" = identity.