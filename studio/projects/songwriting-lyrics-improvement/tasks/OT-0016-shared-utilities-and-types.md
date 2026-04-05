---
title: "Shared Utilities and Types"
type: task
project: songwriting-lyrics-improvement
status: done
created: 2026-04-03
author: mccall
architect: "vault/studio/projects/songwriting-lyrics-improvement/ARCHITECT.md"
group: 1
parallel: false
depends_on: []
---

# OT-0016: Shared Utilities and Types

## What to Build

Create the `src/libs/lyrics-analyzer/` directory structure and implement all shared
utilities that the four T1 tools will depend on. This is the foundation -- every
subsequent task imports from these files.

**Files to create:**

1. **`src/libs/lyrics-analyzer/types.ts`** -- All shared types and interfaces used
   across the library. This is the single source of truth for types.

2. **`src/libs/lyrics-analyzer/shared/cmu-dict.ts`** -- Lazy-loaded CMU dictionary
   singleton. Port the existing pattern from `src/libs/phonetics.ts` lines 57-65
   (the `_dict` / `getDict()` pattern). Use `fromRoot()` from `src/libs/paths.ts`
   for the path to `src/data/cmudict.json`.

3. **`src/libs/lyrics-analyzer/shared/phoneme-utils.ts`** -- Core phoneme functions.
   Port `lookupPhonemes()` and `getStresses()` from `src/libs/phonetics.ts` lines
   68-79. Add new `getPhonemeSequence()` (split phoneme string into individual
   phonemes) and `classifyPhoneme()` (map ARPAbet phoneme to feature category).
   Imports `getDict()` from `cmu-dict.ts`.

4. **`src/libs/lyrics-analyzer/shared/section-parser.ts`** -- Parse lyrics into
   sections from bracket tags. Must handle: `[Verse 1]`, `[Chorus]`, `[Bridge]`,
   `[Pre-Chorus]`, `[Outro]`, `[Intro]`, `[Hook]`, and aliased names (e.g.,
   `[Refrain]` = chorus). Return `ParsedSection[]`. If no section headers found,
   wrap entire text as single "unknown" section. Parse typed bracket metadata
   (e.g., `[Energy: High]`) into `metadata` field without treating them as sections.

5. **`src/libs/lyrics-analyzer/shared/language-detector.ts`** -- Per-word language
   detection. Chain: (1) CMU dict present AND not in Tagalog list -> en (high),
   (2) In Tagalog list AND not in CMU dict -> tl (high), (3) In both -> en (medium),
   (4) Neither -> unknown (low). Export `detectLanguage()` and `profileLanguage()`.

6. **`src/libs/lyrics-analyzer/shared/contraction-handler.ts`** -- Normalize
   contractions for dictionary lookup while preserving originals. "don't" -> lookup
   "don't" first, then "dont". Export `normalizeForLookup()`, `isContraction()`,
   `contractionSyllables()`. Key rule: contractions are SINGLE tokens. "don't" = 1
   syllable, not "do" + "not" = 2.

7. **`src/libs/lyrics-analyzer/shared/input-validator.ts`** -- Sanitize input, reject
   malformed, length guard. Max input: 10,000 characters (roughly 200 lines).
   Strip BOM, normalize line endings, reject empty input. Return
   `ValidationResult` with sanitized text and warnings.

8. **`src/libs/prompt-data/tagalog-common-words.ts`** -- Static list of 500-1000
   common Tagalog words for language detection. Export as `TAGALOG_COMMON_WORDS: Set<string>`.
   Source from publicly available Tagalog frequency lists. Include articles (ang, ng,
   sa, mga), pronouns (ako, ikaw, siya, kami, tayo, sila), common verbs, nouns.

9. **`src/libs/prompt-data/tagalog-stress-exceptions.ts`** -- Static exception list
   for Tagalog words that don't follow penultimate stress. Export as
   `TAGALOG_STRESS_EXCEPTIONS: Map<string, number[]>` where value is stress pattern
   array. Start with ~100 common exceptions. Include oxytone words (stressed on last
   syllable) like common -in, -an suffixes.

10. **`src/libs/prompt-data/phoneme-features.ts`** -- Map every ARPAbet phoneme to
    its feature category. Export `PHONEME_FEATURES: Record<string, PhonemeFeature>`.
    Categories: `open-vowel`, `closed-vowel`, `plosive`, `fricative`, `nasal`,
    `liquid`, `glide`, `affricate`. ~80 entries covering all ARPAbet symbols.
    Handle "NG" (ng) as `nasal` -- this is critical for Tagalog (R11).

## Types to Implement

```typescript
// types.ts -- implement ALL of these

type Severity = "high" | "medium" | "low";
type SectionType = "verse" | "chorus" | "bridge" | "pre-chorus"
                 | "outro" | "intro" | "hook" | "unknown";

interface Flag {
  line: number;
  text: string;
  issue: string;
  severity: Severity;
  suggestion?: string;
  tool: string;
}

interface ToolReport {
  tool: string;
  status: "pass" | "warn" | "fail";
  flags: Flag[];
  summary: string;
  executionMs: number;
}

interface ToolInput {
  sections: ParsedSection[];
  languageProfile: LanguageProfile;
  options?: Record<string, unknown>;
}

interface ParsedSection {
  type: SectionType;
  label: string;
  index: number;
  lines: ParsedLine[];
  metadata: Record<string, string>;
  startLine: number;
}

interface ParsedLine {
  text: string;
  lineNumber: number;
  words: WordToken[];
}

interface WordToken {
  original: string;
  normalized: string;
  language: "en" | "tl" | "unknown";
  phonemes: string | null;
  syllables: number;
  stresses: number[] | null;
  source: "cmu" | "phonemize" | "heuristic";
}

interface LanguageProfile {
  totalWords: number;
  english: number;
  tagalog: number;
  unknown: number;
  dominantLanguage: "en" | "tl" | "mixed";
  perWordAssignments: Array<{
    word: string;
    language: "en" | "tl" | "unknown";
    confidence: "high" | "medium" | "low";
  }>;
}

interface ValidationResult {
  valid: boolean;
  sanitized: string;
  warnings: string[];
}

type PhonemeFeature = "open-vowel" | "closed-vowel" | "plosive"
                    | "fricative" | "nasal" | "liquid" | "glide" | "affricate";
```

## Acceptance Criteria

- [ ] AC-1: Directory `src/libs/lyrics-analyzer/` exists with `types.ts` and `shared/` subdirectory
- [ ] AC-2: `getDict()` lazy-loads CMU dict and caches. Second call returns cached instance (no re-read)
- [ ] AC-3: `lookupPhonemes("walking")` returns ARPAbet string. `lookupPhonemes("xyznonword")` returns null
- [ ] AC-4: `parseSections()` correctly splits lyrics with `[Verse 1]`, `[Chorus]`, `[Bridge]` into typed sections. Lyrics with no headers return single "unknown" section
- [ ] AC-5: `detectLanguage("ang")` returns `{ language: "tl", confidence: "high" }`. `detectLanguage("walking")` returns `{ language: "en", confidence: "high" }`
- [ ] AC-6: `normalizeForLookup("don't")` enables dictionary lookup. `contractionSyllables("don't")` returns 1
- [ ] AC-7: `validateInput("")` returns `{ valid: false, ... }`. Input over 10,000 chars returns warning
- [ ] AC-8: `TAGALOG_COMMON_WORDS` has 500+ entries. `TAGALOG_STRESS_EXCEPTIONS` has 100+ entries
- [ ] AC-9: `PHONEME_FEATURES["NG"]` returns `"nasal"` (not treated as n+g)
- [ ] AC-10: `classifyPhoneme("AE1")` returns `"open-vowel"`. `classifyPhoneme("P")` returns `"plosive"`
- [ ] AC-11: All exports compile with `bun build src/libs/lyrics-analyzer/shared/cmu-dict.ts` (and each file individually)

Maps to PRD: US-1 (AC-1.2, AC-1.3, AC-1.5, AC-1.6), US-2 (AC-2.3), US-4 (AC-4.1), US-16 (AC-16.1, AC-16.2, AC-16.6)

## Files in Scope

- `src/libs/lyrics-analyzer/types.ts` -- **create**
- `src/libs/lyrics-analyzer/shared/cmu-dict.ts` -- **create**
- `src/libs/lyrics-analyzer/shared/phoneme-utils.ts` -- **create**
- `src/libs/lyrics-analyzer/shared/section-parser.ts` -- **create**
- `src/libs/lyrics-analyzer/shared/language-detector.ts` -- **create**
- `src/libs/lyrics-analyzer/shared/contraction-handler.ts` -- **create**
- `src/libs/lyrics-analyzer/shared/input-validator.ts` -- **create**
- `src/libs/prompt-data/tagalog-common-words.ts` -- **create**
- `src/libs/prompt-data/tagalog-stress-exceptions.ts` -- **create**
- `src/libs/prompt-data/phoneme-features.ts` -- **create**
- `src/libs/phonetics.ts` -- **read-only reference** (port patterns, do not modify)
- `src/libs/prompt-data/homographs.ts` -- **read-only reference** (import from here)
- `src/libs/paths.ts` -- **read-only reference** (use `fromRoot()`)

## Architectural Constraints

- Import CMU dict path via `fromRoot("src", "data", "cmudict.json")` from `src/libs/paths.ts`
- All imports use `.js` extension per Bun ESM convention
- Language detector imports `TAGALOG_COMMON_WORDS` from `src/libs/prompt-data/tagalog-common-words.js`
- Language detector imports `getDict` from `./cmu-dict.js` to check CMU presence
- `phoneme-utils.ts` imports `HOMOGRAPH_LOOKUP` from `src/libs/prompt-data/homographs.js` -- the homographs file is NOT moved, only imported
- No npm dependencies in this task. `phonemize` and `syllables` are optional deps handled in OT-0017
- Per-run word cache NOT in this task -- that is in individual tool tasks

## Out of Scope

- Tool implementations (OT-0017 through OT-0020)
- `index.ts` barrel export (OT-0021)
- `report.ts` aggregator (OT-0021)
- CLI entry point (OT-0021)
- npm dependency spikes (handled per-tool)
- Modification of any existing files

## Test Plan

```bash
# Each shared utility compiles independently
bun build src/libs/lyrics-analyzer/types.ts --no-bundle
bun build src/libs/lyrics-analyzer/shared/cmu-dict.ts --no-bundle
bun build src/libs/lyrics-analyzer/shared/phoneme-utils.ts --no-bundle

# Quick smoke test via bun eval
bun -e "import { getDict } from './src/libs/lyrics-analyzer/shared/cmu-dict.js'; const d = getDict(); console.log('entries:', Object.keys(d).length)"
bun -e "import { lookupPhonemes } from './src/libs/lyrics-analyzer/shared/phoneme-utils.js'; console.log(lookupPhonemes('walking'))"
bun -e "import { parseSections } from './src/libs/lyrics-analyzer/shared/section-parser.js'; console.log(JSON.stringify(parseSections('[Verse 1]\nHello world\n[Chorus]\nLa la la'), null, 2))"
bun -e "import { detectLanguage } from './src/libs/lyrics-analyzer/shared/language-detector.js'; console.log(detectLanguage('ang'), detectLanguage('walking'))"
bun -e "import { PHONEME_FEATURES } from './src/libs/prompt-data/phoneme-features.js'; console.log(PHONEME_FEATURES['NG'], PHONEME_FEATURES['P'], PHONEME_FEATURES['AE1'])"
```

## Notes

- This is the foundation task. Nothing else in Phase 1 can start until this is complete.
- The `getDict()` pattern already works in the existing codebase. Port it, don't reinvent.
- Tagalog word list: 500 is the floor. Source from Filipino linguistics frequency corpora. Quality matters more than quantity -- common spoken words, not literary Tagalog.
- Section parser must handle messy real-world input: inconsistent casing, missing closing brackets, duplicate section names, metadata brackets mixed with section brackets.