---
name: suno-lyrics-and-structure-control
description: Unified typed bracket framework for ALL Suno Lyrics, providing section-level control over instrumentation, energy, mood, texture, vocal style, and advanced structural/performance tags, including refined vocal descriptors and instrument arrangement strategies.
type: knowledge
agent: tala
tags: [suno, lyrics, typed-brackets, instrument, energy, mood, texture, vocal-style, unified, structure, tags, callback, atmosphere, performance-notation, ad-libs, arrangement, frequency-stacking]
---

# Suno Lyrics: Unified Typed Bracket Framework

## Core Rule

Typed brackets per section for ALL tracks (instrumental and vocal). Pipe (`|`) inside any bracket = play together. Separate untyped bracket lines = layers (intentional additions).

## Unified Typed Bracket Types

| Type | Purpose | Examples | Behavior |
|------|---------|---------|----------|
| `[Energy:]` | Section energy | `[Energy: Low]`, `[Energy: Rising]`, `[Energy: Maximum]`, `[Energy: Medium→High]` | Controls intensity and drive. Gradient notation works. |
| `[Instrument:]` | Per-section instrumentation | `[Instrument: Shamisen alone | single slow pluck]`, `[Instrument: Shamisen lazy jazz phrases | warm Rhodes underneath | brushed drums barely there]` | Positive per-section instructions for instruments. Pipes combine elements. |
| `[Mood:]` | Section mood/atmosphere | `[Mood: Morning quiet]`, `[Mood: Triumphant]`, `[Mood: Anticipation]` | Guides emotional tone and ambiance. |
| `[Texture:]` | Section sonic quality | `[Texture: Tape-Saturated]`, `[Texture: Lo-fi]` | Adds specific audio characteristics. |
| `[Vocal Style:]` | Vocal delivery direction (vocal only) | `[Vocal Style: Power]`, `[Vocal Style: Belt]`, `[Vocal Style: Open, Confident]` | Directs how vocals are performed. **Refinement:** 'Projected vocals' is more precise and effective than 'Powerful vocals'. |
| `[modulate]` | Key change | `[modulate up a key]` | Works for 1 step up. MUST go *before* lyrics. Numeric values ignored. |
| `[Buildup]` | Section intensity ramp | `[Buildup]`, `[Buildup, 4 bars]` | Creates intensity. Stackable (multiple = longer/more intense). Bar count hint works. |
| `[loop-friendly]` | Section preparation | `[loop-friendly]` | Prepares section for natural repeat, not seamless audio loop. |
| `[Callback:]` | Musical memory | `[Callback: Chorus melody]`, `[Callback: Verse 1 groove]` | References/reuses musical material from a named section within a single generation. |
| Fused Tags | Emotion + Section | `[angry verse]`, `[hopeful chorus]` | Lowercase, combines mood and structure. |
| Audience | Participation | `[crowd sings]` | Adds audience participation/cheering. |

## Advanced Structural & Ambient Tags

### Community-Confirmed Structure Tags
Use these for advanced section control: `[Pre-Chorus]`, `[Post-Chorus]`, `[Breakdown]`, `[Bridge 2]`, `[Interlude]`, `[Coda]`, `[Refrain]`, `[Drop]`, `[Solo]`, `[Transition]`, `[Vamp]`, `[Ad-lib Outro]`.

### Atmosphere/Ambient Tags
These tags add environmental or musical effects.
-   **Environmental:** `[Rain]`, `[Thunder]` (Note: Currently produces soft crackling/brief sounds, not sustained atmosphere. Test with `[Effect: Rain]` or `[Instrument: Rain | Thunder]` and longer sections).
-   **Human/Crowd:** `[Crowd cheers]`, `[Applause]`
-   **Mechanical:** `[Static]`, `[Radio crackle]`
-   **Musical Effects:** `[Reverb wash]`, `[Delay trails]`
-   **Era Anchors:** `[60s]`, `[80s]`, `[90s]` (Use in pipes for genre accuracy, e.g., `[Instrument: Synthwave | [80s] drums]`).

### Bar Counts
Bar counts can be included in any pipe element (e.g., `[Buildup, 4 bars]`, `[Instrument: Guitar solo, 8 bars]`).

## Performance Notation

| Symbol | Purpose | Example |
|---|---|---|
| `~word~` | Elongated Notes | `~loooong~` |
| `word-` | Smooth Cutoff | `stop-` |
| `""` | Spoken Word | `"Hello"` |
| `/` | Pause | `word / word` |
| `ALL CAPS` | Emphatic Delivery | `SHOUT` |
| `...` | Trailing Off | `fading...` |
| `(text)` | Parenthetical Action | `(laughs)` |
| `[text]` | Typed Bracket | `[Instrument: Piano]` |

## Ad-Lib Tags

| Ad-Lib | Placement Rule |
|---|---|
| `(ad-lib)` | Before or after lyrics |
| `(oh)` | Interjection |
| `(yeah)` | Interjection |
| `(hmm)` | Thoughtful pause |
| `(ah)` | Realization/sigh |
| `(ooh)` | Vocalization |
| `(hey)` | Call-out |
| `(uh-huh)` | Affirmation |

### Scream/Growl Formatting
For screams or growls, stretch the vowel (e.g., `screaaaaam`, `growwwl`).

## Why
Suno reads Lyrics typed brackets more carefully than Style descriptors for per-section control. Moving instruments, energy, texture, and vocal style INTO Lyrics gives section-level precision. The Style block is just the global foundation. This unified approach simplifies prompt design for all track types.

### How to apply
Every section gets: Structure tag (e.g., `[Intro]`, `[Verse]`) → Energy → Mood → Vocal Style (if vocal) → Instrument → Texture → Lyrics (if vocal).

## Layers

Intentional additions between lyrics within a section. Layers can also use pipes.

```
[instrumental break saxophone | soft echo]
[Whispered]
(Hey! Hey!)
```

## What Fails

-   **Separate untyped instrument lines** (`[Saxophone]` then `[Harp]` then `[Drums]`) = competing layers = chaos.
-   **"Duet" framing** = equal weight = instruments battle.
-   **Over-describing instruments** in the Lyrics section leads to Suno layering everything simultaneously.

## What Works

-   **`[Instrument: Shamisen jazz phrases | warm Rhodes underneath | brushed drums barely there]`** — all in one typed bracket with pipes.
-   **Atmosphere cues as mood** — `[Mood: Morning quiet]`, `[Mood: Still]`.
-   **Support solo sections** — give support instruments their own section instead of fighting for shared space.

## Reference Pattern (Instrumental Example)

```
[Instrumental]

[Intro]
[Energy: Low]
[Mood: Morning quiet]
[Instrument: Shamisen alone | single slow pluck]

[Verse]
[Energy: Low]
[Mood: Warm, Lazy]
[Instrument: Shamisen lazy jazz phrases | warm Rhodes underneath | brushed drums barely there]

[Bridge]
[Energy: Low]
[Mood: Still]
[Instrument: Shamisen alone | space between notes]

[Outro]
[Energy: Fading]
[Instrument: Shamisen fades | last pluck hangs | warm air]

[No Vocals]
[End]
```

---
*Consolidated from: `suno-lyrics-and-structure.md`*
