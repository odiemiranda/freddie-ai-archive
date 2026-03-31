---
name: suno-prompt-design-guide
description: Comprehensive guide for building Suno prompts, detailing Style block construction, unified typed bracket framework for Lyrics, character budgets, performance notation, advanced multi-layered control, and core prompt engineering principles for clarity and genre precision.
type: knowledge
agent: tala
tags: [suno, lyrics, typed-brackets, instrument, energy, mood, texture, vocal-style, unified, structure, tags, callback, atmosphere, performance-notation, ad-libs, arrangement, frequency-stacking, style-block, char-budget, production, lean, exclude, prompt-optimization, instrumentation, subordination, brightness, layered-control, prompt-design, complexity, genre, signal-clarity, audio-quality, cultural-labels, sfx, platform-behavior, validation]
---

# Suno Prompt Design Guide

This guide unifies the approach to Suno prompt design, emphasizing a lean global Style block and precise per-section control via typed brackets in the Lyrics section, adhering to core engineering principles for clarity.

## 1. Style Block Strategy

The Style block sets the global genre, BPM, and foundational mood. Detailed per-section control is handled by typed brackets in the Lyrics.

### Character Budgets (Unified for ALL Tracks)

| Track Type | Target | Flexible Max | Hard Max |
|---|---|---|---|
| All tracks (with typed brackets in Lyrics) | **~75-120 chars** | ~150 chars | 200 |

#### Why
Leaner prompts result in cleaner audio. Every element added can dilute the core signal. When comprehensive per-section control is achieved via typed brackets in Lyrics, the Style block can operate at the higher end of its recommended character range (up to ~150 chars) without degrading audio quality, provided it remains lean on global elements and avoids redundancy with Lyrics. The upper end of the flexible character budget (~150 chars) for the Style block is validated as effective when combined with detailed per-section control via typed brackets in the Lyrics, maintaining audio clarity and prompt adherence.

#### How to apply
Adhere to the character budget above. Prioritize essential global elements and remove anything that can be implied by genre or controlled more precisely in the Lyrics section.

### What to Include (Never Cut)

#### How to apply
Ensure these essential elements are always present in your Style block:
-   Genre (single word preferred, see "Genre & Cultural Label Precision" below for nuance)
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
-   **Specific instrument roles or behaviors** (e.g., "shamisen melodic plucks", "harp background pad chords", "brushed drums sparse") — these are generally better placed in Lyrics using `[Instrument:]` tags for per-section control. *However, see "Advanced Multi-Layered Control" below for exceptions regarding global placement/subordination.*
-   **Ambient SFX and Atmospheric elements** (e.g., 'rain', 'thunder') — these are ignored in the Style block and must be placed in the Lyrics section.

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
| `[Fade In]` | Intro control | `[Short Fade In]` | Creates a subtle fade-in effect at the beginning of a track. Undocumented but validated. |
| Fused Tags | Emotion + Section | `[angry verse]`, `[hopeful chorus]` | Lowercase, combines mood and structure. |
| Audience | Participation | `[crowd sings]` | Adds audience participation/cheering. |

### Advanced Structural & Ambient Tags

#### Community-Confirmed Structure Tags
Use these for advanced section control: `[Pre-Chorus]`, `[Post-Chorus]`, `[Breakdown]`, `[Bridge 2]`, `[Interlude]`, `[Coda]`, `[Refrain]`, `[Drop]`, `[Solo]`, `[Transition]`, `[Vamp]`, `[Ad-lib Outro]`.

#### Atmosphere/Ambient Tags
These tags add environmental or musical effects.
-   **Environmental:** `[Rain]`, `[Thunder]` (Note: Currently produces soft crackling/brief sounds, not sustained atmosphere. Test with `[Effect: Rain]`, `[Instrument: Rain | Thunder]`, or `[Mood: Rainy]` and longer sections. **Crucially, these must be placed in Lyrics, not the Style block.**)
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

## 3. Core Prompt Engineering Principles

### Complexity Limits & Signal Clarity

#### Why
Fewer ingredients lead to clearer signals and cleaner audio. Over-stuffing prompts results in muddled output and degraded audio quality. Suno interprets every descriptor as a variable to satisfy, diluting the core intent. Audio quality degrades with excessive prompt length. Abstract or ambiguous 'vibe' terms can lead to unpredictable outputs. Overlapping instrument registers cause frequency fighting.

#### How to apply
-   **Max 2 genres:** Use rhythm-clear genres.
-   **Max 3 instruments:** Use specific instrument names (e.g., "cello" instead of "low strings"). Overlapping registers cause frequency fighting. For example, organs can cause frequency stacking issues, particularly in verses. An effective strategy is to pull organs from verses (where they might clash with vocals or other instruments) and reintroduce them in choruses or bridges for impact and clarity.
-   **Max 1 rhythm source:** Usually drums or a driving bassline.
-   **Max 2 textures, max 2 mood words:** Cut in order: chords → textures → mood (keep 1) if over budget.
-   **Translate abstract terms:** Convert ambiguous 'vibe' terms (e.g., 'raw') into concrete, positive, and less ambiguous descriptors (e.g., 'organic').
-   **Audio Quality vs. Length:** Aim for 9.5/10 clarity with a lean Style block. Less information generally yields cleaner audio.

### Genre & Cultural Label Precision

#### Why
Suno interprets cultural labels as instructions to "add everything from this tradition," leading to instrument flooding and overriding specific instrument intent. Cultural labels trigger unrequested genre-associated instruments (e.g., violin, fiddle for "Celtic") even with explicit exclusions. Specific genre archetypes can also introduce subtle audio quality issues. Using cultural or folk genre tags in the Style block can disproportionately elevate associated instruments to a lead role, even when other instruments are specified as primary, overriding the intended instrument hierarchy.

#### How to apply
-   **Never use a culture, tradition, or region as a genre label** in Suno Style blocks (e.g., "Japanese folk fusion", "Folk").
-   Use **neutral instrument labels** (e.g., "harp" instead of "Celtic harp"). Describe the sound with adjectives instead.
-   The specific instrument in the prompt already signals the cultural flavor.
-   **Genre Naming:** While single-word genres are generally preferred for clarity (e.g., "Jazz" not "Jazz fusion"), certain two-word, commonly understood sub-genres like 'lo-fi jazz' can be effectively used, indicating Suno's ability to interpret them as coherent stylistic entities. However, specific genre archetypes (e.g., 'Jazz Lounge') can introduce subtle audio quality issues like 'instrument bleed' due to Suno's interpretation. Critics are effective at pinpointing these genre-specific pitfalls, necessitating prompt adjustments such as exclusion or more precise instrument control.
-   **Pattern:** `{rhythm-template} {instrument} fusion` (e.g., "Jazz shamisen fusion"). The genre gives Suno the rhythm template (jazz = swing, brushed drums). The instrument gives the cultural color (shamisen = Japanese). For leaner blocks, a single genre word like "Jazz" is often sufficient.
-   **Validated Archetypes:** A pre-tested genre archetype, even if it triggers 'genre-flood' warnings from critics, can sometimes override individual critic flags due to its proven robustness.

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

## 5. What Fails & What Works

### What Fails
-   **Separate untyped instrument lines** (`[Saxophone]` then `[Harp]` then `[Drums]`) = competing layers = chaos.
-   **"Duet" framing** = equal weight = instruments battle.
-   **Over-describing instruments** in the Lyrics section leads to Suno layering everything simultaneously.

### What Works
-   **`[Instrument: Shamisen jazz phrases | warm Rhodes underneath | brushed drums barely there]`** — all in one typed bracket with pipes.
-   **Atmosphere cues as mood** — `[Mood: Morning quiet]`, `[Mood: Still]`.
-   **Support solo sections** — give support instruments their own section instead of fighting for shared space.

---
Consolidated from: `suno-prompt-construction-guide.md`, `suno-prompt-engineering-principles.md`, `20260329-155500-for-ambient-sfx-and-atmospheric-elements-1.md`, `20260331-041608-suno-appears-to-interpret-fade-in-and-po.md`, `20260331-193044-the-upper-end-of-the-flexible-character--3.md`
