# Phase 1 Session 1: Types + Shared Modules

**Project:** Prompt Builder Engine
**Phase:** 1 of 5 (Suno Style Builder)
**Session:** 1 of 3
**Estimated:** 2-3 hours
**Depends on:** Phase 0 complete (all `src/libs/prompt-data/` modules exist)

---

## What to Build

Two files that form the foundation for the prompt builder:

1. **`types.ts`** -- All input/output types and Zod validation schemas for the builder
2. **`shared.ts`** -- SharedModules implementation that wraps `prompt-data/` exports into the builder's internal API

These are pure types and pure functions. No platform-specific logic. No CLI. No side effects.

---

## Files to Create

| File | Purpose |
|------|---------|
| `src/libs/prompt-builder/types.ts` | TypeScript interfaces + Zod schemas |
| `src/libs/prompt-builder/shared.ts` | SharedModules implementation |

**Do NOT create yet:** `index.ts`, `strategies/suno.ts`, CLI entry point. Those are Sessions 2 and 3.

---

## TypeScript Interfaces (copy-pasteable)

### types.ts

```typescript
import { z } from "zod";

// ============================================================
// Input Schemas
// ============================================================

export const InstrumentSchema = z.object({
  name: z.string().min(1),
  behavior: z.string().min(1),
});

export const ConstrainedInstrumentSchema = z.object({
  name: z.string().min(1),
  constraint: z.string().min(1),
});

export const VocalSchema = z.object({
  style: z.string().min(1),         // e.g. "breathy female vocal"
  technique: z.string().optional(),  // e.g. "falsetto", "belt"
});

export const StyleInputSchema = z.object({
  platform: z.enum(["suno", "minimax"]),
  type: z.enum(["bgm", "vocal"]),
  genres: z.array(z.string().min(1)).min(1).max(2),
  instruments: z.object({
    lead: InstrumentSchema,
    support: ConstrainedInstrumentSchema.optional(),
    rhythm: ConstrainedInstrumentSchema.optional(),
  }),
  vocal: VocalSchema.optional(),
  key: z.string().min(1),               // e.g. "G Major"
  bpm: z.number().int().min(40).max(240),
  moods: z.array(z.string()).max(5).optional(),
  textures: z.array(z.string()).max(2).optional(),
  chords: z.string().optional(),
  production: z.array(z.string()).optional(),
  realism: z.number().int().min(0).max(3).optional(),
});

export type StyleInput = z.infer<typeof StyleInputSchema>;

// ============================================================
// Output Types
// ============================================================

export type DiagnosticLevel = "error" | "warning" | "info";

export interface Diagnostic {
  level: DiagnosticLevel;
  code: string;           // e.g. "MOOD_CONFLICT", "BPM_RANGE", "VIBE_TRANSLATED"
  message: string;        // human-readable explanation
  field?: string;         // which input field triggered this
  suggestion?: string;    // what to do about it
}

export interface StyleOutput {
  text: string;                          // ready-to-paste Style block
  charCount: number;                     // exact character count
  elementCount: number;                  // comma-separated element count
  slotMap: Record<number, string>;       // slot number -> text (Suno only)
  trimmed: string[];                     // slot names cut during auto-trim
  diagnostics: Diagnostic[];             // all warnings, errors, info
}

// ============================================================
// Normalized Intermediates (internal pipeline)
// ============================================================

export interface NormalizedGenre {
  raw: string;
  canonical: string;
  family: string;
  wasNormalized: boolean;   // true if raw !== canonical
  isUnknown: boolean;       // true if not in lookup table
}

export interface MappedVibe {
  raw: string;
  safe: string;
  wasTranslated: boolean;
  problem?: string;         // why the original was dangerous
}

// ============================================================
// Conflict Check Results
// ============================================================

export interface ConflictResult {
  type: "mood" | "genre" | "production";
  a: string;
  b: string;
}

export interface BlocklistHit {
  name: string;
  field: string;            // which input field contained the match
  alternative: string;      // sonic description to use instead
  genre: string;            // blocklist entry's genre category
}

// ============================================================
// Strategy Interface
// ============================================================

export interface BuildStrategy {
  name: "suno" | "minimax";

  // Char budget
  charTarget: [number, number];       // [min, max] target range
  charHardMax: number;                // absolute maximum

  // Hard limits
  maxGenres: number;
  maxInstruments: number;
  maxMoods: number;

  // Mandatory production tags (always appended)
  mandatoryTags: string[];

  // Type-dependent metatags
  bgmTags: { start: string[]; end: string[] };
  vocalSlot1: (vocal: { style: string; technique?: string }) => string;

  // Assembly
  assembleStyle(input: StyleInput, shared: SharedModules): StyleOutput;
}

// ============================================================
// SharedModules Interface
// ============================================================

export interface SharedModules {
  normalizeGenre(raw: string): NormalizedGenre;
  mapVibe(mood: string): MappedVibe;
  checkMoodConflicts(moods: string[]): ConflictResult[];
  checkGenreConflicts(genres: string[]): ConflictResult[];
  checkProductionConflicts(production: string[]): ConflictResult[];
  checkArtistBlocklist(fields: Record<string, string[]>): BlocklistHit[];
  getTempoRange(genre: string): { min: number; max: number } | null;
  getRealismDescriptors(tier: 0 | 1 | 2 | 3, family: string): string[];
  isAcousticFamily(family: string): boolean;
}
```

### shared.ts

```typescript
import {
  GENRE_LOOKUP,
  CULTURE_INSTRUMENT_MAP,
} from "../prompt-data/genre-normalization.js";
import { VIBE_LOOKUP } from "../prompt-data/vibe-map.js";
import {
  MOOD_CONFLICTS,
  GENRE_CONFLICTS,
  PRODUCTION_CONFLICTS,
  GENRE_TEMPO_RANGES,
} from "../prompt-data/conflict-rules.js";
import { ARTIST_LOOKUP } from "../prompt-data/artist-blocklist.js";
import { REALISM_TIERS } from "../prompt-data/realism-tiers.js";
import type {
  SharedModules,
  NormalizedGenre,
  MappedVibe,
  ConflictResult,
  BlocklistHit,
} from "./types.js";

const ACOUSTIC_FAMILIES = new Set([
  "folk", "jazz", "blues", "classical", "country",
]);

export function createSharedModules(): SharedModules {
  return {
    normalizeGenre(raw: string): NormalizedGenre {
      const lower = raw.toLowerCase().trim();

      // Check cultural label rejection first
      for (const entry of CULTURE_INSTRUMENT_MAP) {
        if (lower === entry.dontSay.toLowerCase()) {
          // Return as unknown with the suggestion available via diagnostic
          return {
            raw,
            canonical: raw,
            family: "unknown",
            wasNormalized: false,
            isUnknown: true, // caller emits error diagnostic with entry.sayInstead
          };
        }
      }

      const match = GENRE_LOOKUP.get(lower);
      if (match) {
        return {
          raw,
          canonical: match.canonical,
          family: match.family,
          wasNormalized: raw.toLowerCase() !== match.canonical.toLowerCase(),
          isUnknown: false,
        };
      }

      // Unknown genre -- pass through with info warning
      return {
        raw,
        canonical: raw,
        family: "unknown",
        wasNormalized: false,
        isUnknown: true,
      };
    },

    mapVibe(mood: string): MappedVibe {
      const entry = VIBE_LOOKUP.get(mood.toLowerCase().trim());
      if (entry) {
        return {
          raw: mood,
          safe: entry.styleTranslation,
          wasTranslated: true,
          problem: entry.problem,
        };
      }
      return { raw: mood, safe: mood, wasTranslated: false };
    },

    checkMoodConflicts(moods: string[]): ConflictResult[] {
      // Check all pairs in the moods array against MOOD_CONFLICTS
      const lower = moods.map(m => m.toLowerCase().trim());
      const results: ConflictResult[] = [];
      for (const conflict of MOOD_CONFLICTS) {
        const hasA = lower.includes(conflict.a.toLowerCase());
        const hasB = lower.includes(conflict.b.toLowerCase());
        if (hasA && hasB) {
          results.push({ type: "mood", a: conflict.a, b: conflict.b });
        }
      }
      return results;
    },

    checkGenreConflicts(genres: string[]): ConflictResult[] {
      const lower = genres.map(g => g.toLowerCase().trim());
      const results: ConflictResult[] = [];
      for (const conflict of GENRE_CONFLICTS) {
        const hasA = lower.includes(conflict.a.toLowerCase());
        const hasB = lower.includes(conflict.b.toLowerCase());
        if (hasA && hasB) {
          results.push({ type: "genre", a: conflict.a, b: conflict.b });
        }
      }
      return results;
    },

    checkProductionConflicts(production: string[]): ConflictResult[] {
      const lower = production.map(p => p.toLowerCase().trim());
      const results: ConflictResult[] = [];
      for (const conflict of PRODUCTION_CONFLICTS) {
        const hasA = lower.includes(conflict.a.toLowerCase());
        const hasB = lower.includes(conflict.b.toLowerCase());
        if (hasA && hasB) {
          results.push({ type: "production", a: conflict.a, b: conflict.b });
        }
      }
      return results;
    },

    checkArtistBlocklist(fields: Record<string, string[]>): BlocklistHit[] {
      const hits: BlocklistHit[] = [];
      for (const [fieldName, values] of Object.entries(fields)) {
        for (const value of values) {
          const match = ARTIST_LOOKUP.get(value.toLowerCase().trim());
          if (match) {
            hits.push({
              name: match.name,
              field: fieldName,
              alternative: match.alternative,
              genre: match.genre,
            });
          }
        }
      }
      return hits;
    },

    getTempoRange(genre: string): { min: number; max: number } | null {
      // Try exact match first, then canonical lookup
      const lower = genre.toLowerCase().trim();
      const direct = GENRE_TEMPO_RANGES[lower];
      if (direct) return { min: direct.min, max: direct.max };

      // Try canonical form
      const normalized = GENRE_LOOKUP.get(lower);
      if (normalized) {
        const canonical = GENRE_TEMPO_RANGES[normalized.canonical.toLowerCase()];
        if (canonical) return { min: canonical.min, max: canonical.max };
      }

      return null;
    },

    getRealismDescriptors(tier: 0 | 1 | 2 | 3, family: string): string[] {
      if (tier === 0) return [];
      if (!ACOUSTIC_FAMILIES.has(family.toLowerCase())) return [];

      // Accumulate descriptors up to the requested tier
      const descriptors: string[] = [];
      for (const t of REALISM_TIERS) {
        if (t.tier <= tier) {
          descriptors.push(...t.descriptors);
        }
      }
      return descriptors;
    },

    isAcousticFamily(family: string): boolean {
      return ACOUSTIC_FAMILIES.has(family.toLowerCase());
    },
  };
}
```

---

## Acceptance Criteria

1. **Types compile.** `bunx tsc --noEmit src/libs/prompt-builder/types.ts` passes with zero errors
2. **Zod rejects bad input with actionable errors:**
   - `genres: []` (empty) -> error mentioning min 1
   - `genres: ["a", "b", "c"]` (3 items) -> error mentioning max 2
   - `type: "vocal"` with missing `vocal` field -> Zod parses OK (optional), but strategy will enforce requirement (Session 2)
   - `bpm: 300` -> error mentioning max 240
   - `platform: "spotify"` -> error listing valid enum values
3. **SharedModules wrap all prompt-data correctly:**
   - `normalizeGenre("lofi hip hop")` returns `{ canonical: "lo-fi", family: "lo-fi", wasNormalized: true, isUnknown: false }`
   - `normalizeGenre("vapor-trap")` returns `{ canonical: "vapor-trap", family: "unknown", isUnknown: true }`
   - `normalizeGenre("Japanese folk fusion")` returns `{ isUnknown: true }` (cultural label rejection)
   - `mapVibe("lazy")` returns `{ safe: "laid-back", wasTranslated: true }`
   - `mapVibe("warm")` returns `{ safe: "warm", wasTranslated: false }`
   - `checkMoodConflicts(["calm", "aggressive"])` returns 1 conflict
   - `checkArtistBlocklist({ moods: ["Drake"] })` returns 1 hit with alternative
   - `getTempoRange("lo-fi hip hop")` returns `{ min: 60, max: 95 }`
   - `getRealismDescriptors(2, "jazz")` returns tier 1 + tier 2 descriptors
   - `getRealismDescriptors(2, "electronic")` returns `[]` (counterproductive)
4. **No runtime dependencies** beyond prompt-data/ and zod

---

## Dependencies on prompt-data/ Modules

| Module | Used By | How |
|--------|---------|-----|
| `genre-normalization.ts` | `shared.ts` | `GENRE_LOOKUP`, `CULTURE_INSTRUMENT_MAP` |
| `vibe-map.ts` | `shared.ts` | `VIBE_LOOKUP` |
| `conflict-rules.ts` | `shared.ts` | `MOOD_CONFLICTS`, `GENRE_CONFLICTS`, `PRODUCTION_CONFLICTS`, `GENRE_TEMPO_RANGES` |
| `artist-blocklist.ts` | `shared.ts` | `ARTIST_LOOKUP` |
| `realism-tiers.ts` | `shared.ts` | `REALISM_TIERS` |

Modules NOT used in Session 1 (future phases): `homographs.ts`, `cliches.ts`, `density-rules.ts`, `section-priority.ts`, `syllable-heuristic.ts`

---

## Out of Scope

- Suno strategy implementation (Session 2)
- CLI entry point (Session 3)
- MiniMax strategy (Phase 4)
- Lyrics builder types (Phase 2-3)
- Tests for shared modules (Session 2 tests exercise these transitively)
- The `vocal` field being required when `type: "vocal"` -- this is enforced at strategy level in Session 2 via a `.refine()` or pre-assembly check, not at schema level (because Zod `.refine()` breaks generic typing). Document this clearly in a code comment.

---

## Architectural Notes

- `types.ts` exports both Zod schemas AND inferred TypeScript types. Consumers import the type; the builder uses the schema for runtime validation.
- `shared.ts` exports a `createSharedModules()` factory function, not a singleton. This keeps it testable (no module-level state).
- `Diagnostic` replaces the split `warnings: string[]` / `errors: string[]` from the tech doc. Single array with `level` field is cleaner for filtering. The tech doc's `StyleOutput.warnings` and `StyleOutput.errors` become `StyleOutput.diagnostics` filtered by level. Document this deviation.
- The `BuildStrategy.vocalSlot1` is a function (not just a string array) because vocal slot 1 needs to combine `style` + optional `technique` at runtime. Simpler than making the strategy reassemble it.
- Cultural label rejection in `normalizeGenre` returns `isUnknown: true` rather than throwing. The caller (strategy) is responsible for emitting an error diagnostic with the suggestion from `CULTURE_INSTRUMENT_MAP`. This keeps shared modules pure (no side effects, no diagnostics emission).
