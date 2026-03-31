---
name: suno-prompt-construction-guide
description: Comprehensive guide for building Suno prompts, detailing the unified typed bracket framework for Lyrics, Style block construction, character budgets, performance notation, and advanced multi-layered control techniques for instrumentation and sonic characteristics.
type: knowledge
agent: tala
tags: [suno, lyrics, typed-brackets, instrument, energy, mood, texture, vocal-style, unified, structure, tags, callback, atmosphere, performance-notation, ad-libs, arrangement, frequency-stacking, style-block, char-budget, production, lean, exclude, prompt-optimization, instrumentation, subordination, brightness, layered-control]
---

# Suno Prompt Construction Guide

This guide unifies the approach to Suno prompt design, emphasizing a lean global Style block and precise per-section control via typed brackets in the Lyrics section.

## 1. Style Block Strategy

The Style block sets the global genre, BPM, and foundational mood. Detailed per-section control is handled by typed brackets in the Lyrics.

### Character Budgets (Unified for ALL Tracks)

| Track Type | Target | Flexible Max | Hard Max |
|---|---|---|---|
| All tracks (with typed brackets in Lyrics) | **~75-120 chars** | ~150 chars | 200 |

#### Why
Leaner prompts result in cleaner audio. Every element added dilutes the core signal. When comprehensive per-section control is achieved via typed brackets in Lyrics, the Style block can operate at the higher end of its recommended character range (up to ~150 chars) without degrading audio quality, provided it remains lean on global elements and avoids redundancy with Lyrics.

#### How to apply
Adhere to the character budget above. Prioritize essential global elements and remove anything that can be implied by genre or controlled more precisely in the Lyrics section.

### What to Include (Never Cut)

#### How to apply
Ensure these essential elements are always present in your Style block:
-   Genre (single word preferred, see `suno-prompt-engineering-principles.md` for nuance)
-   Key, BPM
-   "clean mix, separated instruments" (mandatory — helps audio clarity)
-   `[Instrumental]` and `[No Vocals]` tags (if applicable)

### What to Drop (for ALL Tracks)

#### Why
These elements frequently confuse Suno, lead to hallucinations, are redundant, or are better controlled via typed brackets in the Lyrics section.

#### How to apply
Avoid including these in your Style block:
-   **Chord progressions** — invite piano hallucination (tested: Gmaj7-Em7-Cmaj9-D7 caused piano at 75% mark).
-   **`[Background Music]` tag** — arrangement already implies it.
-   **Negative constraints** ("no melody", "no fills") — confuse Suno. Use positive descriptors instead, e.g., "pad chords", "sparse" in Lyrics `[Instrument:]` tags.
-   **Specific vocal styles** (e.g., 'deep chanting', 'whispered') — these belong in Lyrics using `[Vocal Style:]` tags.
-   **Specific instrument roles or behaviors** (e.g., "shamisen melodic plucks", "harp background pad chords", "brushed drums sparse") — these are generally better placed in Lyrics using `[Instrument:]` tags for per-section control. *However, see "Multi-Layered Control" below for exceptions regarding global placement/subordination.*

### Minimal Exclude Strategy

#### Why
When every section has `[Instrument:]` typed brackets in the Lyrics, Suno has positive per-section instructions and doesn't hallucinate unwanted instruments. This positive guidance replaces the need for extensive negative exclusions. Adding abstract terms confuses Suno, as it struggles to interpret "what NOT to feel" compared to "what instrument NOT to use."

#### How to apply
Only add dealbreakers (e.g., auto-tune for raw vocal, electronic for acoustic genres). Otherwise, leave the Exclude Style block empty. Do not add abstract terms like: `aggressive, uptempo, crescendo, dramatic build, drop, bright melody, distortion`.

## 2. Suno Lyrics: Unified Typed Bracket Framework

Typed brackets per section provide precise control over instrumentation, energy, mood, texture, vocal style, and advanced structural/performance tags.

### Core Rule
Typed brackets per section for ALL tracks (instrumental and vocal). Pipe (`|`) inside any bracket = play together. Separate untyped bracket lines = layers (intentional additions).

### Unified Typed Bracket Types

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

### Advanced Structural & Ambient Tags

#### Community-Confirmed Structure Tags
Use these for advanced section control: `[Pre-Chorus]`, `[Post-Chorus]`, `[Breakdown]`, `[Bridge 2]`, `[Interlude]`, `[Coda]`, `[Refrain]`, `[Drop]`, `[Solo]`, `[Transition]`, `[Vamp]`, `[Ad-lib Outro]`.

#### Atmosphere/Ambient Tags
These tags add environmental or musical effects.
-   **Environmental:** `[Rain]`, `[Thunder]` (Note: Currently produces soft crackling/brief sounds, not sustained atmosphere. Test with `[Effect: Rain]` or `[Instrument: Rain | Thunder]` and longer sections).
-   **Human/Crowd:** `[Crowd cheers]`, `[Applause]`
-   **Mechanical:** `[Static]`, `[Radio crackle]`
-   **Musical Effects:** `[Reverb wash]`, `[Delay trails]`
-   **Era Anchors:** `[60s]`, `[80s]`, `[90s]` (Use in pipes for genre accuracy, e.g., `[Instrument: Synthwave | [80s] drums]`).

#### Bar Counts
Bar counts can be included in any pipe element (e.g., `[Buildup, 4 bars]`, `[Instrument: Guitar solo, 8 bars]`).

### Performance Notation

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

### Ad-Lib Tags

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

#### Scream/Growl Formatting
For screams or growls, stretch the vowel (e.g., `screaaaaam`, `growwwl`).

### Layers
Intentional additions between lyrics within a section. Layers can also use pipes.

```
[instrumental break saxophone | soft echo]
[Whispered]
(Hey! Hey!)
```

### How to apply
Every section gets: Structure tag (e.g., `[Intro]`, `[Verse]`) → Energy → Mood → Vocal Style (if vocal) → Instrument → Texture → Lyrics (if vocal).

## 3. What Fails & What Works

### What Fails
-   **Separate untyped instrument lines** (`[Saxophone]` then `[Harp]` then `[Drums]`) = competing layers = chaos.
-   **"Duet" framing** = equal weight = instruments battle.
-   **Over-describing instruments** in the Lyrics section leads to Suno layering everything simultaneously.

### What Works
-   **`[Instrument: Shamisen jazz phrases | warm Rhodes underneath | brushed drums barely there]`** — all in one typed bracket with pipes.
-   **Atmosphere cues as mood** — `[Mood: Morning quiet]`, `[Mood: Still]`.
-   **Support solo sections** — give support instruments their own section instead of fighting for shared space.

## 4. Advanced Multi-Layered Control

Subtle global sonic characteristics, such as brightness or specific instrument placement/subordination, can be effectively controlled through a multi-layered approach.

### Why
Relying on a single prompt element for complex sonic control is often insufficient. Combining multiple strategies across the Style block, Lyrics, and Exclude sections creates a robust, precise instruction set for Suno.

#### How to apply
-   **Combine strategies:** For nuanced control (e.g., brightness, distant instrument placement), integrate:
    1.  **Avoid problematic genre tags:** Prevent unintended instrument prominence (e.g., dropping 'Folk' genre tag to prevent fiddle from becoming lead).
    2.  **Descriptive language in Style block:** Use for global texture/mood or to establish an instrument's *subordinate placement* (e.g., "distant low fiddle drone" in Style). This refines the general rule against specific instrument roles in Style, allowing for global *placement* or *subordination* when part of a layered strategy.
    3.  **Precise instrument phrasing within Lyrics `[Instrument:]` tags:** For per-section control and to reinforce global intent (e.g., "underneath" in all Lyrics for a subordinate instrument).
    4.  **Strategically excluding closely related instruments:** To ensure the desired instrument is rendered as intended (e.g., "violin" in Exclude to support a fiddle drone).
    5.  **Global texture descriptors:** Use sparingly in Style for overarching characteristics (e.g., "rolled-off highs" globally for brightness control), especially when not implied by genre. This refines the general advice to avoid textures in Style, showing they can be effective for subtle global characteristics in a layered approach.

### Effective Instrument Subordination

#### Why
Over-muffling or repetitive negative descriptors can lead to an instrument being entirely removed. Precise, positive language is more effective.

#### How to apply
-   For instrument subordination, use descriptive language like 'distant/low drone' rather than repeated 'muffled' descriptors.
-   The MiniMax critic is effective at identifying detrimental layering or over-muffling that would effectively erase an instrument from the mix, so heed its warnings.

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
Consolidated from: `suno-lyrics-and-structure-control.md`, `suno-style-block-strategy.md`, `20260329-072140-for-instrument-subordination-descriptive-2.md`, `20260329-072140-subtle-global-sonic-characteristics-such-3.md`
