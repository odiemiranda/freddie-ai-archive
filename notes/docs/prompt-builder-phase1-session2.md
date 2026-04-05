# Phase 1 Session 2: Suno Strategy + Tests

**Project:** Prompt Builder Engine
**Phase:** 1 of 5 (Suno Style Builder)
**Session:** 2 of 3
**Estimated:** 3-4 hours
**Depends on:** Session 1 complete (`types.ts`, `shared.ts` exist and compile)

---

## What to Build

Three things:

1. **Suno strategy** -- the 11-slot assembly engine with auto-trim loop
2. **`buildStyle()` pipeline** -- the main entry point that validates, normalizes, checks conflicts, dispatches to strategy, and returns output
3. **Unit tests** -- comprehensive coverage of slot assembly, normalization, trim, conflicts, determinism, and BGM vs vocal divergence

---

## Files to Create

| File | Purpose |
|------|---------|
| `src/libs/prompt-builder/strategies/suno.ts` | Suno 11-slot strategy implementing `BuildStrategy` |
| `src/libs/prompt-builder/index.ts` | `buildStyle()` pipeline + barrel exports |
| `src/libs/prompt-builder/__tests__/suno-strategy.test.ts` | Slot assembly, trim, metatags, type divergence |
| `src/libs/prompt-builder/__tests__/shared-modules.test.ts` | Genre normalization, vibe map, conflicts, blocklist, tempo, realism |
| `src/libs/prompt-builder/__tests__/pipeline.test.ts` | End-to-end pipeline: input -> output, determinism, known-good tracks |

---

## TypeScript Implementation Contract

### strategies/suno.ts

```typescript
import type { BuildStrategy, StyleInput, StyleOutput, SharedModules, Diagnostic, NormalizedGenre } from "../types.js";

/**
 * Suno slot numbers and their content.
 * Cut order when over budget: 6 -> 8 -> 9 (keep 1) -> 4 (simplify)
 * Never cut: 2, 3, 7, 10
 */
const SLOT_NAMES: Record<number, string> = {
  1: "metatag/vocal",    // [Instrumental] or vocal style
  2: "genre",            // max 2, normalized
  3: "lead instrument",  // name + behavior
  4: "support instrument", // name + constraint
  5: "rhythm",           // name + constraint
  6: "chords",           // cut first
  7: "key/bpm",          // never cut
  8: "textures",         // cut second
  9: "moods",            // cut third (keep 1)
  10: "production",      // never cut
  11: "end metatags",    // [No Vocals] [Background Music] or empty
};

// Trim order: which slots to cut and in what order
const TRIM_ORDER: number[] = [6, 8, 9, 4];

export const sunoStrategy: BuildStrategy = {
  name: "suno",
  charTarget: [75, 120],
  charHardMax: 200,
  maxGenres: 2,
  maxInstruments: 3,
  maxMoods: 5,      // input cap; slot 9 targets max 2 after vibe mapping
  mandatoryTags: ["clean mix", "separated instruments"],

  bgmTags: {
    start: ["[Instrumental]"],
    end: ["[No Vocals]", "[Background Music]"],
  },

  vocalSlot1(vocal) {
    if (vocal.technique) {
      return `${vocal.style}, ${vocal.technique}`;
    }
    return vocal.style;
  },

  assembleStyle(input: StyleInput, shared: SharedModules): StyleOutput {
    // Implementation contract below
  },
};
```

**`assembleStyle` must implement these steps in order:**

1. **Pre-validate vocal requirement.** If `type === "vocal"` and `!input.vocal`, emit error diagnostic `VOCAL_REQUIRED` and return early with empty text.

2. **Normalize genres.** Map each `input.genres` entry through `shared.normalizeGenre()`. For each:
   - If `isUnknown` and the raw text matches a cultural label from `CULTURE_INSTRUMENT_MAP`, emit error diagnostic `CULTURAL_LABEL` with the suggestion.
   - If `isUnknown` but not a cultural label, emit info diagnostic `UNKNOWN_GENRE`.
   - If `wasNormalized`, emit info diagnostic `GENRE_NORMALIZED` showing raw -> canonical.

3. **Check BPM range.** For each normalized genre, call `shared.getTempoRange()`. If BPM is outside the range, emit warning diagnostic `BPM_RANGE`.

4. **Check conflicts.** Run mood, genre, and production conflict checks. Emit warning diagnostics for each.

5. **Check artist blocklist.** Scan `moods`, `textures`, and `production` fields. Emit warning diagnostic `ARTIST_BLOCKLIST` for each hit.

6. **Map vibes.** Run each mood through `shared.mapVibe()`. Emit info diagnostic `VIBE_TRANSLATED` for each translation.

7. **Check realism applicability.** If `input.realism > 0` and genre family is NOT acoustic, emit warning diagnostic `REALISM_COUNTERPRODUCTIVE`.

8. **Assemble slots 1-11.** Build each slot string:
   - **Slot 1:** `[Instrumental]` for BGM, `vocalSlot1(input.vocal)` for vocal
   - **Slot 2:** Normalized genre string. Two genres comma-joined. If same family with different canonical (e.g., "lo-fi" + "jazzhop" -> "lo-fi (jazz-colored)"), use the parenthetical form. Different families: comma-join.
   - **Slot 3:** `{lead.name} {lead.behavior}`
   - **Slot 4:** `{support.name} {support.constraint}` (empty string if no support)
   - **Slot 5:** `{rhythm.name} {rhythm.constraint}` (empty string if no rhythm)
   - **Slot 6:** `input.chords` or empty string
   - **Slot 7:** `{key}, {bpm} BPM`
   - **Slot 8:** Textures joined with comma. Plus realism descriptors if applicable (injected here). Empty if none.
   - **Slot 9:** Mapped moods joined with comma (max 2 after mapping; if more than 2 after vibe translation, keep first 2).
   - **Slot 10:** Mandatory tags + any custom `input.production` entries + realism production descriptors if applicable. Always includes "clean mix, separated instruments".
   - **Slot 11:** BGM end tags joined with space, or empty string for vocal

9. **Join slots.** Filter out empty slots. Join with `, ` (comma space). Exception: slot 1 and slot 11 are joined with space (no comma) relative to adjacent slots if they are metatags (brackets).

10. **Auto-trim loop.** Measure char count. If over `charTarget[1]` (120):
    - Walk TRIM_ORDER. For each slot:
      - Slot 6 (chords): remove entirely
      - Slot 8 (textures): remove entirely
      - Slot 9 (moods): reduce to 1 mood (keep the first)
      - Slot 4 (support): shorten constraint to just the instrument name
    - After each cut, re-measure. Stop when within target.
    - Record each cut in `trimmed[]` with slot name.
    - If still over `charHardMax` (200) after all trims: emit error diagnostic `CHAR_HARD_MAX`.

11. **Build output.** Return `StyleOutput` with text, charCount, elementCount, slotMap, trimmed, diagnostics.

### index.ts

```typescript
import { StyleInputSchema, type StyleInput, type StyleOutput } from "./types.js";
import { createSharedModules } from "./shared.js";
import { sunoStrategy } from "./strategies/suno.js";
// import { minimaxStrategy } from "./strategies/minimax.js"; // Phase 4

export type { StyleInput, StyleOutput } from "./types.js";
export { StyleInputSchema } from "./types.js";

const STRATEGIES = {
  suno: sunoStrategy,
  // minimax: minimaxStrategy, // Phase 4
} as const;

/**
 * Build a Style block from structured input.
 *
 * Pipeline: validate -> normalize -> conflict check -> blocklist -> assemble -> trim
 */
export function buildStyle(rawInput: unknown): StyleOutput {
  // 1. Zod validation
  const parsed = StyleInputSchema.safeParse(rawInput);
  if (!parsed.success) {
    // Convert Zod errors to Diagnostic[] with actionable messages
    // Each ZodIssue becomes a Diagnostic with:
    //   level: "error"
    //   code: "VALIDATION_ERROR"
    //   field: issue.path.join(".")
    //   message: issue.message
    //   suggestion: format-specific help text
    return {
      text: "",
      charCount: 0,
      elementCount: 0,
      slotMap: {},
      trimmed: [],
      diagnostics: /* mapped Zod errors */,
    };
  }

  const input = parsed.data;

  // 2. Get strategy
  const strategy = STRATEGIES[input.platform];
  if (!strategy) {
    return /* error: unsupported platform */;
  }

  // 3. Create shared modules and delegate to strategy
  const shared = createSharedModules();
  return strategy.assembleStyle(input, shared);
}
```

---

## Test Specifications

### suno-strategy.test.ts

```
describe("Suno Strategy - Slot Assembly")
  test("BGM: slot 1 is [Instrumental], slot 11 is [No Vocals] [Background Music]")
  test("Vocal: slot 1 is vocal style, slot 11 is empty")
  test("Vocal with technique: slot 1 combines style + technique")
  test("Single genre: slot 2 is just the canonical name")
  test("Two genres same family: parenthetical form (e.g. lo-fi (jazz-colored))")
  test("Two genres different families: comma-joined")
  test("Lead instrument: slot 3 is name + behavior")
  test("Support instrument: slot 4 is name + constraint")
  test("No support instrument: slot 4 omitted")
  test("Rhythm: slot 5 is name + constraint")
  test("Key/BPM: slot 7 is always present")
  test("Mandatory tags: 'clean mix, separated instruments' always in slot 10")
  test("Realism tier 2 with jazz: descriptors in slots 8 and 10")
  test("Realism tier 2 with electronic: no descriptors (counterproductive)")

describe("Suno Strategy - Trim Loop")
  test("Under 120 chars: no trimming")
  test("Over 120: chords (slot 6) cut first")
  test("Still over: textures (slot 8) cut second")
  test("Still over: moods reduced to 1")
  test("Still over: support constraint simplified")
  test("Over 200 after all trims: error diagnostic emitted")
  test("Trimmed slots recorded in output.trimmed")

describe("Suno Strategy - Type Divergence")
  test("Same input with type:bgm vs type:vocal produces different slot 1 and 11")
  test("Vocal type without vocal field: error diagnostic VOCAL_REQUIRED")
  test("BGM type with vocal field: info warning VOCAL_IGNORED")

describe("Suno Strategy - Diagnostics")
  test("Genre normalization: GENRE_NORMALIZED info for 'lofi' -> 'lo-fi'")
  test("Unknown genre: UNKNOWN_GENRE info for 'vapor-trap'")
  test("Cultural label: CULTURAL_LABEL error for 'Japanese folk fusion'")
  test("BPM out of range: BPM_RANGE warning for lo-fi at 150 BPM")
  test("Mood conflict: MOOD_CONFLICT warning for calm + aggressive")
  test("Vibe translated: VIBE_TRANSLATED info for 'lazy' -> 'laid-back'")
  test("Artist in moods: ARTIST_BLOCKLIST warning for 'Drake'")
```

### shared-modules.test.ts

```
describe("normalizeGenre")
  test("'lofi hip hop' -> canonical 'lo-fi', family 'lo-fi'")
  test("'jazzhop' -> canonical 'lo-fi', family 'lo-fi' (maps to family)")
  test("'boom bap' -> canonical 'hip-hop', family 'hip-hop'")
  test("'vapor-trap' (unknown) -> passthrough with isUnknown: true")
  test("'Japanese folk fusion' -> isUnknown: true (cultural label)")
  test("case insensitive: 'JAZZ' normalizes same as 'jazz'")
  test("trims whitespace: ' lo-fi ' normalizes same as 'lo-fi'")

describe("mapVibe")
  test("all 8 dangerous vibes translate correctly")
  test("safe words pass through unchanged ('warm', 'bright', 'driving')")
  test("case insensitive: 'LAZY' translates same as 'lazy'")

describe("checkMoodConflicts")
  test("calm + aggressive: 1 conflict")
  test("calm + warm: 0 conflicts")
  test("relaxing + driving + pumping: 2 conflicts (relaxing-driving, relaxing-pumping)")
  test("empty array: 0 conflicts")

describe("checkGenreConflicts")
  test("death metal + lo-fi hip hop: 1 conflict")
  test("jazz + lo-fi: 0 conflicts")

describe("checkProductionConflicts")
  test("polished + raw: 1 conflict")
  test("clean + crisp: 0 conflicts")

describe("checkArtistBlocklist")
  test("'Drake' in moods: 1 hit with alternative")
  test("'Eminem' in production: 1 hit")
  test("no artist names: 0 hits")
  test("case insensitive: 'daft punk' matches")

describe("getTempoRange")
  test("'lo-fi hip hop' -> { min: 60, max: 95 }")
  test("'jazz' -> { min: 80, max: 200 }")
  test("'vapor-trap' (unknown) -> null")
  test("alias lookup: 'boom bap' resolves through canonical")

describe("getRealismDescriptors")
  test("tier 0: empty array regardless of family")
  test("tier 1 jazz: ['close mic', 'natural dynamics', 'room tone']")
  test("tier 2 jazz: tier 1 + tier 2 descriptors")
  test("tier 3 folk: all three tiers accumulated")
  test("tier 2 electronic: empty array (counterproductive)")
  test("tier 1 hip-hop: empty array (counterproductive)")
```

### pipeline.test.ts

```
describe("buildStyle - Validation")
  test("missing genres field: error diagnostic with field name")
  test("bpm out of range: error diagnostic")
  test("invalid platform: error diagnostic")
  test("empty input: error diagnostic listing all missing fields")

describe("buildStyle - End to End")
  test("BGM lo-fi jazz: complete output with correct structure")
  test("Vocal r&b: complete output with vocal style at slot 1")
  test("Minimal input (1 genre, lead only, no moods): valid output")
  test("Maximum input (all fields populated): valid output within char budget")

describe("buildStyle - Determinism")
  test("same input produces identical output across 100 runs")

describe("buildStyle - Known Good Tracks")
  // Extract inputs from 5 real tracks in vault/studio/tracks/
  // Run through builder, compare output to expected
  // NOTE: Wick should find 5 recent tracks, extract their Style block,
  //       reverse-engineer the StyleInput JSON, and use those as fixtures.
  //       If the builder output differs from the original, it should be
  //       demonstrably better (shorter, cleaner, same intent).
  test("known track 1: output matches or improves on original")
  test("known track 2: output matches or improves on original")
  test("known track 3: output matches or improves on original")
  test("known track 4: output matches or improves on original")
  test("known track 5: output matches or improves on original")

describe("buildStyle - Conflict Matrix Parity")
  // Port conflict test cases from validate-prompt.ts
  // Builder must catch everything validate-prompt catches
  // (except cross-prompt coherence which stays in validate-prompt)
  test("mood conflicts detected: all pairs from MOOD_CONFLICTS")
  test("genre conflicts detected: all pairs from GENRE_CONFLICTS")
  test("production conflicts detected: all pairs from PRODUCTION_CONFLICTS")
```

---

## Acceptance Criteria

1. `bun test src/libs/prompt-builder/` passes -- all tests green
2. Builder output for known-good tracks matches expected Style blocks (or is demonstrably better with documented reasoning)
3. **Determinism proof:** same input -> same output across 100 runs, zero diff
4. Conflict checks catch all cases from `validate-prompt.ts` conflict matrices
5. Both BGM and vocal outputs structurally correct (right metatags in right positions)
6. Auto-trim fires in correct order and stays within char budget
7. No diagnostics with level "error" in any valid input test case

---

## Dependencies on prompt-data/ Modules

All modules imported transitively through `shared.ts` (Session 1). The strategy does NOT import prompt-data directly -- it goes through SharedModules.

---

## Dependencies on Session 1

| From Session 1 | Used By |
|----------------|---------|
| `types.ts` - `StyleInput`, `StyleOutput`, `BuildStrategy`, `SharedModules`, `Diagnostic` | `suno.ts`, `index.ts`, all tests |
| `shared.ts` - `createSharedModules()` | `index.ts` |

---

## Out of Scope

- CLI entry point (Session 3)
- Tool registry integration (Session 3)
- Workflow doc updates (Session 3)
- MiniMax strategy (Phase 4)
- Vocal-first ordering test with period separators (noted in tech doc as "test, do not adopt" -- run the test, record results in a code comment, do not change the separator)
- Cross-prompt coherence checks (stays in validate-prompt.ts)
- Lyrics builder (Phase 2-3)

---

## Architectural Notes

- The strategy MUST NOT import from `prompt-data/` directly. All data access goes through `SharedModules`. This keeps the strategy testable with mock shared modules if needed.
- Genre formatting for same-family pairs: When both genres are in the same family (e.g., "lo-fi" + "jazzhop" both in "lo-fi" family), the canonical form already handles this via `GENRE_FAMILIES` aliases. Two "lo-fi" family genres should produce the parenthetical form from the canonical table, not a comma-joined pair. If both resolve to the exact same canonical string, deduplicate to one.
- Slot join logic: metatag slots (1 and 11) use space separation, not comma. The text `[Instrumental] lo-fi, shamisen...` has a space after `[Instrumental]`, not a comma. Same for `...instruments [No Vocals] [Background Music]` at the end.
- `buildStyle()` takes `unknown` input and validates via Zod. This is intentional -- the caller (CLI or registry) passes raw JSON. Type safety comes from Zod, not from TypeScript generics at the boundary.
