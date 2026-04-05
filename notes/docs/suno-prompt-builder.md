---
title: "Tech Approach: Suno Prompt Builder"
date: 2026-03-30
status: planning
source: ChatGPT Suno GPT analysis, existing Tala/Rune workflows
tags: [suno, prompt-builder, lyrics, style, generation, typescript]
---

# Tech Approach: Suno Prompt Builder (`suno-prompt-builder.ts`)

## Problem

All Suno generation intelligence currently lives in markdown (agent files, skill workflows, critics). The AI rebuilds Style blocks and Lyrics scaffolds from scratch every time, relying on prompt instructions to "remember" genre rules, bracket patterns, and platform constraints. This leads to:

- **Inconsistency** — same genre, different sessions = different Style formatting
- **Lost knowledge** — acting directions, SFX-as-instruments, prose-style vibe paragraphs are techniques we've discovered but not codified
- **Genre blindness** — comedy/novelty tracks need different formatting than production tracks, but the agent doesn't know this until told
- **Repeated validation** — charcount, phonetics, bracket format checks happen late. A code-generated baseline would start valid.

## Solution

`src/libs/suno-prompt-builder.ts` — a TypeScript library that builds Style blocks and Lyrics scaffolds from structured inputs, with genre-specific rules baked in as data.

**Code provides the floor, AI provides the ceiling.** Agents still refine and customize — but they start from a deterministic, validated baseline.

## What the Builder Produces

Given structured input → outputs:

1. **Style block** — genre-specific format (keyword tags for production, prose for comedy/novelty), pre-validated charcount
2. **Lyrics scaffold** — section structure with typed brackets pre-filled, genre-appropriate density targets
3. **Acting directions** — performance cues appropriate to genre/mood (comedy = `[Spoken, playful]`, ballad = `[Tender, intimate]`)
4. **Validation report** — charcount, instrument count, genre compatibility, bracket format check

## Architecture

```typescript
// Usage by Tala agent:
import { buildSunoPrompt } from "../libs/suno-prompt-builder.js";

const result = buildSunoPrompt({
  genre: ["filipino novelty", "reggae"],
  mood: ["playful", "comedic", "unserious"],
  instruments: ["offbeat guitar", "rubbery bass", "kitchen percussion"],
  energy: "high",
  vocalStyle: "animated, spoken asides, laugh breaks",
  bpm: 95,
  key: "G Major",
  type: "song",  // bgm | song
  language: "tagalog",
  sfx: ["kawali sizzle", "chopping", "spoon taps"],
});

// result.style — formatted Style block (genre-appropriate)
// result.lyrics — Lyrics scaffold with brackets pre-filled
// result.actingDirections — per-section performance cues
// result.validation — { charcount, instrumentCount, genreMatch, warnings }
```

## Genre Profiles

The builder uses genre profiles — data objects that define formatting rules per genre family:

```typescript
interface GenreProfile {
  name: string;
  family: string;  // production | novelty | ambient | hip-hop | ...
  styleFormat: "keywords" | "prose" | "hybrid";
  styleCharTarget: [number, number];  // [min, max]
  lyricsFormat: "typed-brackets" | "acting-directions" | "minimal";
  densityTarget: { verse: number; chorus: number; bridge: number };  // syllables per line
  actingDirections: boolean;  // include performance cues?
  sfxInStyle: boolean;  // SFX as instruments in Style, or staging in Lyrics?
  bracketExamples: string[];
}
```

### Example Profiles

| Genre Family | Style Format | Lyrics Format | SFX In Style? | Acting Dirs? |
|-------------|-------------|---------------|---------------|-------------|
| **Production** (lo-fi, jazz, ambient) | keywords, 75-120 chars | typed brackets | No — Lyrics staging | No |
| **Comedy/Novelty** | prose, 150-250 chars | acting directions | Yes — as instruments | Yes |
| **Hip-Hop/Rap** | keywords, 80-130 chars | typed brackets + flow cues | No | Partial (delivery cues) |
| **Ballad/Emotional** | keywords, 75-120 chars | typed brackets + emotion tags | No | Yes (vocal performance) |
| **Reggae/Island** | hybrid, 100-180 chars | acting directions | Partial (percussion SFX) | Yes |

## Data Sources

The builder draws from existing reference files + new codified data:

| Source | What it provides | Format |
|--------|-----------------|--------|
| `references/suno/production.md` | BPM/key ranges per genre | Already exists |
| `references/suno/platform-knowledge.md` | Style anatomy, tag reliability | Already exists |
| `references/lyrics/rhythm-and-arcs.md` | Genre density targets | Already exists |
| **NEW: `src/data/genre-profiles.json`** | Genre family → formatting rules | Structured JSON |
| **NEW: `src/data/acting-directions.json`** | Mood → performance cue templates | Structured JSON |
| **NEW: `src/data/sfx-instruments.json`** | SFX → instrument role mapping | Structured JSON |

## Acting Direction Templates

Discovered from Suno GPT analysis. These are bracket-format performance cues that produce more natural vocal delivery:

```json
{
  "playful": ["[Spoken, playful]", "[Cheeky, bouncy]", "[Mock-serious, then breaks]"],
  "intimate": ["[Tender, close mic]", "[Whispered, vulnerable]", "[Soft, barely there]"],
  "aggressive": ["[Shouted, raw]", "[Intense, building]", "[Explosive, no restraint]"],
  "comedic": ["[Over-the-top announcer]", "[Deadpan delivery]", "[Breaks character]", "[Fast, comedic]"],
  "dramatic": ["[Dramatic pause]", "[Building intensity]", "[Quiet before the storm]"]
}
```

These supplement (not replace) our typed brackets. A section could have both:
```
[Verse]
[Energy: High]
[Mood: Comedic]
[Spoken, playful, with sizzling sounds]
[Instrument: Offbeat guitar bright | Rubbery bass groove | Kawali percussion]
```

## SFX-as-Instrument Mapping

When SFX are framed as instruments (percussion role), they can work in Style. When framed as atmosphere, they belong in Lyrics.

```json
{
  "kitchen": {
    "styleOk": true,
    "role": "percussion",
    "mapping": "kawali sizzles, chopping, spoon taps as percussive elements"
  },
  "rain": {
    "styleOk": false,
    "role": "atmosphere",
    "mapping": "[Texture: rain ambience, distant thunder]"
  },
  "crowd": {
    "styleOk": true,
    "role": "texture",
    "mapping": "crowd chatter, glasses clinking as ambient texture"
  }
}
```

## Integration with Existing Workflow

The builder slots into `/create-track` P3-002 (Build Style block) and P3-005 (Write Lyrics):

```
Current flow:
  P3-002: Tala builds Style from scratch using references + instincts
  P3-005: Tala writes Lyrics from scratch using genre targets

With builder:
  P3-002: Tala calls buildSunoPrompt() → gets baseline Style → refines
  P3-005: Tala gets Lyrics scaffold from builder → fills in actual lyrics

  Builder output is a STARTING POINT, not final output.
  Tala still applies creativity, user context, and music card constraints.
```

## What This Doesn't Replace

- **Tri-critic loop** — still runs on the refined output
- **Phonetics check** — still runs after lyrics are written
- **User approval** — still gates every phase
- **Brain.db queries** — still inform the analysis
- **Discover.ts** — still surfaces reference matches
- **Tala's creative judgment** — the builder is a floor, not a ceiling

## Implementation Plan

1. Define `GenreProfile` interface and create `src/data/genre-profiles.json` with 5-8 core genre families
2. Create `src/data/acting-directions.json` with mood → performance cue templates
3. Create `src/data/sfx-instruments.json` with SFX → role mapping
4. Build `src/libs/suno-prompt-builder.ts` — main builder function
5. Add CLI wrapper `src/tools/suno-prompt-builder.ts` for testing and direct use
6. Wire into `/create-track` P3-002 and P3-005 as optional baseline generator
7. Test across genres: lo-fi, comedy, ballad, reggae, hip-hop
8. Update Tala agent file to reference the builder

## Open Questions

- Should there be a `minimax-prompt-builder.ts` too? MiniMax has different formatting rules.
- How do we handle genre fusion? (e.g., "comedy jazz fusion" — which profile wins?)
- Should the builder output be visible to the user in the Phase 2 review, or only to Tala?
- Should genre profiles live in `src/data/` (code-versioned) or `vault/studio/references/` (vault-versioned)?
- Can the builder learn from past tracks? e.g., analyze successful tracks in `vault/studio/tracks/` to refine profiles.
