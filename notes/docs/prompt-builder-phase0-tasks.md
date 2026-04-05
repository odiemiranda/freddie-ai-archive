# Phase 0: Shared Data Extraction — Task Slices for Wick

**Architect:** McCall
**Target:** `src/libs/prompt-data/`
**Constraint:** All files must compile with `tsc --noEmit`. No `any`, no untyped arrays. All exports are typed constants.

---

## Architecture Overview

```
src/libs/prompt-data/
  index.ts              <-- barrel export (all named exports)
  genre-normalization.ts
  vibe-map.ts
  conflict-rules.ts
  realism-tiers.ts
  homographs.ts
  artist-blocklist.ts
  cliches.ts
  density-rules.ts
  section-priority.ts
  syllable-heuristic.ts
```

Every file exports typed `as const` arrays/records. No classes, no runtime logic, no imports from outside the project (pure data). The barrel `index.ts` re-exports everything.

---

## Shared Type Definitions

Wick: place this at the top of each file as needed, OR create a `src/libs/prompt-data/types.ts` if types are shared across 3+ files. Use your judgment.

```typescript
// -- Genre Normalization --
export interface GenreAlias {
  readonly input: string;       // what the user types
  readonly canonical: string;   // normalized form
  readonly family: string;      // family group
}

export interface GenreFamily {
  readonly family: string;
  readonly canonical: string;   // primary canonical name
  readonly aliases: readonly string[];
}

// -- Vibe Map --
export interface VibeEntry {
  readonly vibe: string;          // dangerous user word
  readonly problem: string;       // why it's dangerous
  readonly styleTranslation: string;   // safe Style block word
  readonly lyricsTranslation: string;  // safe Lyrics block word
}

// -- Conflict Rules --
export interface ConflictPair {
  readonly a: string;
  readonly b: string;
}

export interface TempoRange {
  readonly min: number;
  readonly max: number;
}

export interface VocalGenreWarning {
  readonly vocalStyle: string;
  readonly unusualGenres: readonly string[];
}

export interface SectionFollower {
  readonly section: string;
  readonly typicalFollowers: readonly string[];
}

// -- Realism Tiers --
export interface RealismTier {
  readonly tier: number;
  readonly label: string;
  readonly descriptors: readonly string[];
  readonly slots: readonly string[];        // which Style slots they go in
  readonly counterproductive: readonly string[]; // genres where this hurts
}

// -- Homographs --
export interface HomographPronunciation {
  readonly pronunciation: string;
  readonly meaning: string;
}

export interface Homograph {
  readonly word: string;
  readonly pronunciations: readonly HomographPronunciation[];
}

// -- Artist Blocklist --
export interface BlockedArtist {
  readonly name: string;
  readonly alternative: string;
  readonly genre: string;
}

// -- Cliches --
export type ClicheSource = "ai-slop" | "ubiquitous" | "both";
export type ClicheGenre = "universal" | "pop" | "rock" | "country" | "indie"
  | "rap" | "r&b" | "electronic" | "jazz" | "metal" | "folk" | "ambient"
  | "romantic" | "emotional-flattener";

export interface Cliche {
  readonly pattern: string;
  readonly source: ClicheSource;
  readonly genre: ClicheGenre;
  readonly betterDirection?: string;  // from ai-slop only
}

// -- Density Rules --
export interface GenreDensity {
  readonly genre: string;
  readonly linesPerVerse: readonly [number, number]; // [min, max]
  readonly linesPerChorus: readonly [number, number];
  readonly syllablesPerLine: readonly [number, number];
  readonly density: string;           // "Sparse" | "Low" | "Low-Mod" | "Moderate" | "Mod-High" | "High" | "Very High"
  readonly rhymeApproach: string;
}

// -- Section Priority --
export interface SectionPriority {
  readonly section: string;
  readonly weight: number;  // 1=low, 2=medium, 3=high
}

// -- Syllable Heuristic --
// (exported as a function, not data — see Task C)
```

---

## Task A: Critical Data Files

**Files to create:**
- `src/libs/prompt-data/genre-normalization.ts`
- `src/libs/prompt-data/vibe-map.ts`
- `src/libs/prompt-data/conflict-rules.ts`

### A1: Genre Normalization (`genre-normalization.ts`)

**Sources:**
- `vault/studio/references/suno/craft-rules.md` lines 24-49 (normalization table + culture rules)
- `C:\Users\odiem\Downloads\claude-ai-music-skills-main\reference\suno\genre-list.md` (full file, 500+ genres)

**Process:**
1. Extract the ~10 rows from `craft-rules.md` lines 28-39 into canonical families
2. Expand each family by cross-referencing the 500+ genre list. For each genre in the external list, if it clearly maps to an existing family, add it as an alias. If it does not map, skip it entirely. Do NOT invent new families.
3. The culture-to-instrument rules (lines 41-49) go into a separate export: `CULTURE_INSTRUMENT_MAP`

**Expected families (minimum):** lo-fi, hip-hop, jazz, reggae, rock, electronic, pop. You will discover more from the external list: metal, folk/country, classical, ambient, blues, punk, r&b/soul, world. Total should be ~15-20 families with 3-30 aliases each.

**Exports:**
```typescript
export const GENRE_FAMILIES: readonly GenreFamily[] = [...] as const;

// Flat lookup: input string -> { canonical, family }
// Built from GENRE_FAMILIES at module level
export const GENRE_LOOKUP: ReadonlyMap<string, { canonical: string; family: string }>;

// "Don't say X, say Y" cultural instrument rules
export const CULTURE_INSTRUMENT_MAP: readonly {
  readonly dontSay: string;
  readonly sayInstead: string;
  readonly why: string;
}[] = [...] as const;
```

**Acceptance criteria:**
- Every entry from `craft-rules.md` lines 28-39 is present
- 500+ genre list entries that map to existing families are included as aliases
- `GENRE_LOOKUP` is a Map keyed by lowercased input string
- No family has zero aliases
- Culture instrument map has all 5 entries from lines 43-49

### A2: Vibe Map (`vibe-map.ts`)

**Source:** `vault/studio/references/suno/craft-rules.md` lines 72-93

**Extract the 8 dangerous vibes** from lines 77-86 and the vibe translation process from lines 89-93.

**Exports:**
```typescript
export const DANGEROUS_VIBES: readonly VibeEntry[] = [...] as const;

// Fast lookup by lowercased vibe word
export const VIBE_LOOKUP: ReadonlyMap<string, VibeEntry>;

// The 5 vibe categories for classification
export const VIBE_CATEGORIES = ["Texture", "Space", "Era", "Energy", "Atmosphere"] as const;
export type VibeCategory = typeof VIBE_CATEGORIES[number];
```

**Acceptance criteria:**
- All 8 entries from the table (lazy, cold, muted, distant, dark, floating, dreamy, raw)
- Each entry has all 4 fields populated
- VIBE_LOOKUP keyed by lowercase

### A3: Conflict Rules (`conflict-rules.ts`)

**Source:** `src/tools/validate-prompt.ts` lines 19-102

**Extract verbatim** from the existing validate-prompt.ts. This is a data-only extraction — the arrays already exist, just move them to the data layer with proper types.

**Exports:**
```typescript
export const MOOD_CONFLICTS: readonly ConflictPair[] = [...] as const;
export const GENRE_CONFLICTS: readonly ConflictPair[] = [...] as const;
export const PRODUCTION_CONFLICTS: readonly ConflictPair[] = [...] as const;
export const GENRE_TEMPO_RANGES: Readonly<Record<string, TempoRange>> = {...} as const;
export const VOCAL_GENRE_UNUSUAL: readonly VocalGenreWarning[] = [...] as const;
export const TYPICAL_SECTION_FOLLOWERS: readonly SectionFollower[] = [...] as const;
```

**Transformation notes:**
- Current `MOOD_CONFLICTS` is `[string, string][]` — convert to `ConflictPair[]` with `{ a, b }` shape
- Current `VOCAL_GENRE_UNUSUAL` is `Record<string, string[]>` — convert to `VocalGenreWarning[]`
- Current `TYPICAL_FOLLOWERS` is `Record<string, string[]>` — convert to `SectionFollower[]`
- Do NOT modify validate-prompt.ts in this phase. That refactor happens in Phase 1.

**Acceptance criteria:**
- MOOD_CONFLICTS has exactly 35 pairs (count from source)
- GENRE_CONFLICTS has exactly 13 pairs
- PRODUCTION_CONFLICTS has exactly 19 pairs
- GENRE_TEMPO_RANGES has exactly 25 entries
- VOCAL_GENRE_UNUSUAL has exactly 6 vocal styles
- TYPICAL_SECTION_FOLLOWERS has exactly 12 sections
- All data matches source exactly — no additions, no removals

---

## Task B: External Extractions

**Files to create:**
- `src/libs/prompt-data/homographs.ts`
- `src/libs/prompt-data/artist-blocklist.ts`
- `src/libs/prompt-data/cliches.ts`

### B1: Homographs (`homographs.ts`)

**Source:** `C:\Users\odiem\Downloads\claude-ai-music-skills-main\servers\bitwize-music-server\handlers\text_analysis.py` lines 34-95

**Extract all 20 homograph entries** (the Python source shows 20, not 18 — the spec was approximate). Convert from Python dict format to TypeScript.

**Python source structure:**
```python
_HIGH_RISK_HOMOGRAPHS = {
    "live": [{"pron_a": "LIV (live performance)", "pron_b": "LYVE (alive, living)"}],
    ...
}
```

**Convert to:**
```typescript
export const HOMOGRAPHS: readonly Homograph[] = [
  {
    word: "live",
    pronunciations: [
      { pronunciation: "LIV", meaning: "live performance" },
      { pronunciation: "LYVE", meaning: "alive, living" },
    ],
  },
  // ... all entries
] as const;

// Fast lookup by lowercased word
export const HOMOGRAPH_LOOKUP: ReadonlyMap<string, Homograph>;
```

**Acceptance criteria:**
- All entries from the Python `_HIGH_RISK_HOMOGRAPHS` dict are present (count them — should be 20: live, read, lead, wind, close, tear, bow, bass, row, sow, wound, minute, resume, object, project, record, present, content, desert, refuse)
- Each entry has exactly 2 pronunciations
- Pronunciation and meaning are split from the Python `"PRON (meaning)"` format

### B2: Artist Blocklist (`artist-blocklist.ts`)

**Source:** `C:\Users\odiem\Downloads\claude-ai-music-skills-main\reference\suno\artist-blocklist.md` (full file)

**Extract all artist entries** organized by genre heading.

```typescript
export const ARTIST_BLOCKLIST: readonly BlockedArtist[] = [...] as const;

// Fast lookup by lowercased artist name
export const ARTIST_LOOKUP: ReadonlyMap<string, BlockedArtist>;
```

**Acceptance criteria:**
- Count all table rows across all genre sections. The source has entries under: Electronic & Dance (8), Hip-Hop & Rap (8), Jazz & Blues (6), Rock & Metal (8), Punk (8), Pop & Contemporary (8), R&B & Soul (6), Country & Folk (6), World & Cultural (4), Classical & Orchestral (4), Soundtrack & Theme (5), Vocal & Choral (4), K-Pop & Korean (24), Industrial & Experimental (5). Total: ~104 entries.
- Every entry has name, alternative, and genre
- ARTIST_LOOKUP keyed by lowercased name

### B3: Cliches (`cliches.ts`)

**Sources:**
- `vault/studio/references/lyrics/ai-slop-patterns.md` (full file — universal + genre-specific + romantic + emotional flatteners)
- `C:\Users\odiem\Downloads\claude-ai-music-skills-main\servers\bitwize-music-server\handlers\lyrics_analysis.py` lines 20-75 (`_COMMON_SONG_PHRASES` — the "ubiquitous" set)

**Process:**
1. Extract all patterns from `ai-slop-patterns.md` — tag each with `source: "ai-slop"` and its genre category
2. Extract all phrases from `_COMMON_SONG_PHRASES` — tag each with `source: "ubiquitous"` and `genre: "universal"`
3. Cross-reference: if a phrase appears in BOTH sources, tag it `source: "both"` and keep only one entry
4. For ai-slop entries, include the `betterDirection` field from the "Better direction" column
5. Ubiquitous entries have no betterDirection (they're simple phrases, not tables with advice)

```typescript
export const CLICHES: readonly Cliche[] = [...] as const;

// Lookup sets for fast checking
export const CLICHE_PATTERNS: ReadonlySet<string>;  // all patterns, lowercased
export const UBIQUITOUS_PHRASES: ReadonlySet<string>;  // just the ubiquitous ones
```

**ai-slop counts (approximate):**
- Universal: 15
- Pop: 6, Rock: 6, Country: 6, Indie: 6, Rap: 6, R&B: 5, Electronic: 5, Jazz: 4, Metal: 4, Folk: 4, Ambient: 4
- Romantic: 9
- Emotional flatteners: 11
- Total ai-slop: ~91

**Ubiquitous count:** 75 phrases in `_COMMON_SONG_PHRASES`

**Acceptance criteria:**
- De-duplicated: no pattern string appears twice in the final array
- Every entry has source and genre
- betterDirection populated for all ai-slop entries (from the table column)
- Total entries: ~140-160 after dedup
- `CLICHE_PATTERNS` set contains all lowercased pattern strings
- `UBIQUITOUS_PHRASES` set contains only the ubiquitous-sourced entries

---

## Task C: Density, Priority, Heuristic, Tiers, and Barrel

**Files to create:**
- `src/libs/prompt-data/density-rules.ts`
- `src/libs/prompt-data/section-priority.ts`
- `src/libs/prompt-data/syllable-heuristic.ts`
- `src/libs/prompt-data/realism-tiers.ts`
- `src/libs/prompt-data/index.ts`

### C1: Density Rules (`density-rules.ts`)

**Sources:**
- `vault/studio/references/lyrics/genre-structure.md` lines 34-48 (Genre Quick Reference table)
- `vault/studio/references/lyrics/genre-structure.md` lines 196-228 (Fusion rules + Suno limits)

**Extract the genre density table** from lines 34-48 and the density spectrum from line 199.

```typescript
export const GENRE_DENSITY: readonly GenreDensity[] = [...] as const;

// Density spectrum for fusion resolution (least dense -> most dense)
export const DENSITY_SPECTRUM: readonly string[] = [
  "Ambient", "Folk", "Reggae", "Lo-fi", "Jazz", "Indie", "Country",
  "Pop", "Rock", "R&B", "Melodic Hip-Hop", "Rap", "Metal"
] as const;

// Suno syllable ceiling per genre (from lines 222-228)
export const SUNO_SYLLABLE_LIMITS: Readonly<Record<string, number>> = {
  "rap": 12,
  "hip-hop": 12,
  "jazz": 10,
  "soul": 10,
  // default for all other genres: not specified (no ceiling)
} as const;
```

**Acceptance criteria:**
- All 13 genres from the quick reference table are present
- Each has linesPerVerse, linesPerChorus, syllablesPerLine as [min, max] tuples
- DENSITY_SPECTRUM has exactly 13 entries in the correct order
- SUNO_SYLLABLE_LIMITS covers the genres mentioned in lines 222-228

### C2: Section Priority (`section-priority.ts`)

**Source:** `C:\Users\odiem\Downloads\claude-ai-music-skills-main\servers\bitwize-music-server\handlers\lyrics_analysis.py` lines 79-88

```typescript
export const SECTION_PRIORITY: readonly SectionPriority[] = [
  { section: "chorus", weight: 3 },
  { section: "hook", weight: 3 },
  { section: "pre-chorus", weight: 2 },
  { section: "bridge", weight: 2 },
  { section: "outro", weight: 2 },
  { section: "verse", weight: 1 },
  { section: "intro", weight: 1 },
  { section: "end", weight: 1 },
] as const;

export const SECTION_PRIORITY_LOOKUP: ReadonlyMap<string, number>;
```

**Acceptance criteria:**
- All 8 entries from the Python source
- Weights match exactly (chorus/hook=3, pre-chorus/bridge/outro=2, verse/intro/end=1)

### C3: Syllable Heuristic (`syllable-heuristic.ts`)

**Source:** `C:\Users\odiem\Downloads\claude-ai-music-skills-main\servers\bitwize-music-server\handlers\lyrics_analysis.py` lines 266-301 (`_count_syllables_word`)

This is the one file that exports a **function**, not just data. Port the Python heuristic to TypeScript.

```typescript
/**
 * Count syllables in a single word using vowel cluster heuristic.
 *
 * Rules:
 * 1. Count vowel groups (aeiouy consecutive = 1 group)
 * 2. Subtract trailing silent 'e' if count > 1
 * 3. Handle consonant-'le' endings (bottle, apple — add 1)
 * 4. Floor at 1
 */
export function countSyllables(word: string): number {
  // ... port from Python
}

/**
 * Count syllables for a full line of text.
 * Splits on whitespace, sums per-word counts.
 */
export function countLineSyllables(line: string): number {
  // ...
}
```

**Acceptance criteria:**
- Logic matches the Python source exactly (same vowel set including 'y', same silent-e rule, same consonant-le rule, same floor-at-1)
- Test cases that must pass:
  - `countSyllables("bottle")` === 2
  - `countSyllables("apple")` === 2
  - `countSyllables("the")` === 1
  - `countSyllables("beautiful")` === 3
  - `countSyllables("a")` === 1
  - `countSyllables("")` === 0

### C4: Realism Tiers (`realism-tiers.ts`)

**Source:** `vault/studio/references/suno/craft-rules.md` lines 196-204

```typescript
export const REALISM_TIERS: readonly RealismTier[] = [
  {
    tier: 1,
    label: "subtle",
    descriptors: ["close mic", "natural dynamics", "room tone"],
    slots: ["8", "10"],  // textures + production
    counterproductive: ["electronic", "hip-hop", "synthwave", "edm"],
  },
  {
    tier: 2,
    label: "recorded feel",
    descriptors: ["breath detail", "pick noise", "short room reverb", "limited stereo"],
    slots: ["8", "10"],
    counterproductive: ["electronic", "hip-hop", "synthwave", "edm"],
  },
  {
    tier: 3,
    label: "full bedroom/live",
    descriptors: ["fret squeak", "tape saturation", "wow and flutter", "one-take", "imperfections kept"],
    slots: ["8", "10"],
    counterproductive: ["electronic", "hip-hop", "synthwave", "edm"],
  },
] as const;
```

**Acceptance criteria:**
- All 3 tiers present with exact descriptors from source
- Counterproductive genres listed (from line 204: "electronic/hip-hop/synthwave")
- Slots reference the Style block formula (slots 8 and 10)

### C5: Barrel Export (`index.ts`)

```typescript
// src/libs/prompt-data/index.ts
export * from "./genre-normalization.js";
export * from "./vibe-map.js";
export * from "./conflict-rules.js";
export * from "./realism-tiers.js";
export * from "./homographs.js";
export * from "./artist-blocklist.js";
export * from "./cliches.js";
export * from "./density-rules.js";
export * from "./section-priority.js";
export * from "./syllable-heuristic.js";
```

**Note:** Use `.js` extensions in imports (Bun ESM resolution).

**Acceptance criteria:**
- All 10 modules re-exported
- `bun build src/libs/prompt-data/index.ts --outdir /dev/null` succeeds (or `tsc --noEmit`)
- No circular dependencies

---

## File Overlap Analysis

```
Task A                    Task B                    Task C
-----------               -----------               -----------
genre-normalization.ts    homographs.ts             density-rules.ts
vibe-map.ts               artist-blocklist.ts       section-priority.ts
conflict-rules.ts         cliches.ts                syllable-heuristic.ts
                                                    realism-tiers.ts
                                                    index.ts (barrel)
```

**Overlap: NONE.** Each task touches completely independent files. All three tasks can run in parallel worktrees.

**Dependency:** Task C's barrel (`index.ts`) imports from A and B files, but only needs the filenames — not their contents. Write the barrel with all imports; it will compile once all tasks merge.

---

## Parallel Dispatch Configuration

```
Task A: isolation: "worktree", model: "sonnet", branch: "phase0-task-a"
Task B: isolation: "worktree", model: "sonnet", branch: "phase0-task-b"
Task C: isolation: "worktree", model: "sonnet", branch: "phase0-task-c"
```

**Merge order:** A and B first (parallel), then C (needs A+B for barrel compile check). Or merge all three and fix barrel if needed — it is a one-line-per-module file.

---

## Post-Merge Validation

After all three tasks merge:

```bash
# Must pass
bun build src/libs/prompt-data/index.ts --outdir /dev/null
# OR
npx tsc --noEmit src/libs/prompt-data/index.ts
```

Then spot-check:
1. `GENRE_FAMILIES` length >= 15
2. `GENRE_LOOKUP` size >= 200 (many aliases)
3. `DANGEROUS_VIBES` length === 8
4. `MOOD_CONFLICTS` length === 35
5. `HOMOGRAPHS` length === 20
6. `ARTIST_BLOCKLIST` length >= 100
7. `CLICHES` length >= 130
8. `GENRE_DENSITY` length === 13
9. `SECTION_PRIORITY` length === 8
10. `REALISM_TIERS` length === 3
