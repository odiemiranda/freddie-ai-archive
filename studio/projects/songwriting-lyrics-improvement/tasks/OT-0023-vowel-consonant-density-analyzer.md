---
title: "Vowel/Consonant Density Analyzer"
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

# OT-0023: Vowel/Consonant Density Analyzer

## What to Build

Implement the T2 vowel/consonant density analyzer at
`src/libs/lyrics-analyzer/tools/vowel-consonant-density.ts`. This tool breaks down the
phonetic texture of each section and flags texture-energy mismatches.

**Per-section phoneme breakdown:**
- Use `WordToken.phonemes` (ARPAbet string from ToolInput)
- Split phonemes, classify each via `PHONEME_FEATURES` from `src/libs/prompt-data/phoneme-features.ts`
- Count: open vowels, closed vowels, plosives, fricatives, nasals, liquids, glides
- Calculate totalPhonemes per section

**Texture profile classification:**
- "open": open vowels > 40% of phonemes
- "percussive": plosives + affricates > 30%
- "liquid": liquids + nasals > 25%
- "balanced": no category dominant

**Texture-energy alignment flags:**
- Open-vowel-dominated lines in `[Energy: High]` sections -> "May lack impact", severity: low
- Plosive-dominated lines in `[Energy: Low]` sections -> "May feel harsh", severity: low
- These are ADVISORY -- texture is stylistic choice

**This tool absorbs and replaces the LY5 consonant texture check from `phonetics.ts`:**
- Port the harsh cluster detection (lines 93-126 of `phonetics.ts`)
- Port the soft pattern detection
- Port the energy-level parsing from typed brackets
- All current harsh/soft cluster detection preserved as a subset of this richer analysis

**Export:**
```typescript
export function analyzeDensity(input: ToolInput): VowelConsonantDensityReport;

interface VowelConsonantDensityReport extends ToolReport {
  tool: "vowel-consonant-density";
  perSection: Array<{
    section: string;
    openVowels: number;
    closedVowels: number;
    plosives: number;
    fricatives: number;
    nasals: number;
    liquids: number;
    glides: number;
    totalPhonemes: number;
    textureProfile: "open" | "percussive" | "liquid" | "balanced";
  }>;
  textureEnergyFlags: Flag[];
}
```

## Acceptance Criteria

- [ ] AC-1: Per-section phoneme counts computed from ARPAbet phonemes
- [ ] AC-2: "NG" classified as nasal (Tagalog "ng" digraph, R11)
- [ ] AC-3: Texture profile correctly classified for an open-vowel-heavy section
- [ ] AC-4: Texture-energy flag generated for harsh clusters in low-energy section (LY5 preserved)
- [ ] AC-5: Soft-in-high-energy flag generated for breathy words in high-energy section (LY5 preserved)
- [ ] AC-6: Words with no phonemes (heuristic source) gracefully skipped in density analysis
- [ ] AC-7: Advisory only -- no blocking flags

Maps to PRD: US-6 (AC-6.1 through AC-6.5), S2, S8

## Files in Scope

- `src/libs/lyrics-analyzer/tools/vowel-consonant-density.ts` -- **create**
- `src/libs/prompt-data/phoneme-features.ts` -- **read-only** (import from OT-0016)
- `src/libs/phonetics.ts` -- **read-only reference** (already deleted in OT-0022, use git history for reference of LY5 logic)

## Architectural Constraints

- Import `PHONEME_FEATURES` from `src/libs/prompt-data/phoneme-features.js`
- Words with `source: "heuristic"` have no phonemes. Skip them in density calculation but count them in totalPhonemes estimate (use syllable count * 2.5 as rough phoneme estimate)
- Energy level comes from section metadata (parsed by section-parser in OT-0016)
- This tool MUST preserve all existing LY5 detection behavior. The old harsh/soft cluster logic is a subset. If anything the old tool caught, this tool should also catch

## Out of Scope

- Singability scoring (OT-0026)
- CLI/report updates for this tool (OT-0026)
- Workflow integration

## Test Plan

Port the existing consonant texture test cases from the current system:
- Harsh cluster "strength" in low-energy section -> flagged
- Soft word "shimmer" in high-energy section -> flagged
- "ng" in Tagalog word -> counted as nasal, not n+g

## Notes

- This tool replaces LY5 from `phonetics.ts`. The existing behavior must be preserved as a baseline. The new analysis (full phoneme feature breakdown) is additive on top.
- The `phonetics.ts` file is already deleted by OT-0022. Reference the logic from git history or from the copy you read during this session (lines 89-340 of the old file).