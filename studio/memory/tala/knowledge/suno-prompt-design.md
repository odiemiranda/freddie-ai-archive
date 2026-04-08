---
name: suno-prompt-design
description: Comprehensive guide for building Suno prompts, detailing Style block construction (including the canonical envelope for BGM), unified typed bracket framework for Lyrics, character budgets, performance notation, advanced multi-layered control, archetype blends, and core prompt engineering principles for clarity, genre precision, descriptor refinement, and effects management, including guidance for dynamic instrument roles, genre anchoring, descriptive vs. prescriptive prompting, and word repetition.
type: knowledge
agent: tala
tags: [suno, lyrics, typed-brackets, instrument, energy, mood, texture, vocal-style, unified, structure, tags, callback, atmosphere, performance-notation, ad-libs, arrangement, frequency-stacking, style-block, char-budget, production, lean, exclude, prompt-optimization, instrumentation, subordination, brightness, layered-control, prompt-design, complexity, genre, signal-clarity, audio-quality, cultural-labels, sfx, platform-behavior, validation, archetype, instrument-hallucination, genre-artifacts, dealbreaker, descriptor-refinement, dynamic-roles, directive-verbs, effects, delay, shamisen, genre-anchoring, bgm, canonical-envelope, quality-gate, instrumental, no-vocals, bpm, key, phase-3, critic-feedback, vetoes, descriptive-vs-prescriptive, source-prompt-mirroring, word-repetition, monotone-vs-melodic, surgical-fixes, melodicism, negative-constraints, prompt-failure, monotony, descriptor-count]
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
Adhere to the character budget above. Prioritize essential global elements and remove anything that can be implied by genre or controlled more precisely in the Lyrics section. Style blocks should consistently use comma-separated descriptors, not pipe delimiters. Pipe delimiters are reserved for combining elements within typed brackets in the Lyrics section.

### 1.1. BGM Style Block: Canonical Envelope

BGM (Background Music) Style blocks have a **canonical envelope** that is not optional. Missing any part of it is a Phase 3 quality failure that the user WILL catch during review.

#### Why
This structured approach ensures all mandatory elements for instrumental/BGM tracks are consistently present, preventing critical quality failures. The `[Instrumental]` and `[No Vocals]` tags lock structural meta-tag behavior at the prompt level, providing robust control.

#### How to apply
For BGM/instrumental tracks, adhere strictly to the following envelope structure and elements:

```
[Instrumental] {genre}, {instruments}, {mood}, {Key}, {NN BPM}, {atmosphere}, {mix} [No Vocals]
```

Elements in order:
1.  **`[Instrumental]` tag at the front** — not the bare word "instrumental" inline.
2.  **Genre** — at the 2-genre cap max.
3.  **Instruments** — at the 3-instrument cap max, with role/timbre hints.
4.  **Mood descriptors** — if warranted; often elided when Key/BPM carry the feel.
5.  **Key** — `{Letter} {Major/Minor}` format, e.g., `A Major`, `F Minor`. Paired with BPM.
6.  **BPM** — format `NN BPM` (integer, space, capital BPM). Never `bpm`, never bracketed, never `bpm: 104`.
7.  **Atmosphere** — scene/space descriptor, e.g., `sunlit cafe`, `moonlit temple`.
8.  **`clean mix`** — soul mandatory.
9.  **`[No Vocals]` tag at the end** — paired with the `[Instrumental]` opener. This provides belt-and-suspenders with Lyrics block tags, not a duplication.

#### Phase 3 Quality Gate for BGM

Before declaring Phase 3 done on ANY instrumental/BGM build, explicitly verify the envelope by walking these four slots against the Style block:

1.  ✓ `[Instrumental]` tag at start?
2.  ✓ `[No Vocals]` tag at end?
3.  ✓ BPM token inline (`NN BPM`)?
4.  ✓ Key token inline (`{Letter} Major/Minor`)?

Missing any one = Phase 3 is not done. Do NOT rely on charcount or validate-prompt to catch these — charcount's "slot 10 mandatory" warnings do catch "separated instruments" but not the envelope tags or BPM/Key tokens, and validate-prompt only flags structural issues.

#### What NOT to interpret as this envelope
The envelope is specifically for BGM/instrumental tracks. For vocal tracks, the pattern differs:
- `[Instrumental]` tag is WRONG for vocal tracks.
- `[No Vocals]` is WRONG for vocal tracks.
- BPM and Key still apply, but mood descriptors typically carry more weight than scene/atmosphere.

#### Preventing this in future Phase 3 runs
Add an explicit step at the end of Phase 3 generation (before returning to Freddie):
> **BGM Envelope Check** — if `type === "bgm"` or `"instrumental"`, walk the four envelope slots against the final Style block. If any are missing, fix them before reporting Phase 3 done, even if charcount and validate-prompt pass.

### What to Include (General)

#### How to apply
Ensure these essential elements are always present in your Style block:
-   Genre (up to 2, see "Genre & Cultural Label Precision" below for nuance)
-   Key, BPM
-   "clean mix, separated instruments" (mandatory — helps audio clarity)
-   `[Instrumental]` and `[No Vocals]` tags (if applicable, and mandatory for BGM as per Canonical Envelope)

### What to Drop (for ALL Tracks)

#### Why
These elements frequently confuse Suno, lead to hallucinations, are redundant, or are better controlled via typed brackets in the Lyrics section. Stacked negative constraints specifically fight Suno's training and produce unstable, oscillating failures, often cycling through contradictory interpretations (e.g., clean→spoken→falsetto→falsetto for vocal delivery).

#### How to apply
Avoid including these in your Style block:
-   **Chord progressions** — invite piano hallucination (tested: Gmaj7-Em7-Cmaj9-D7 caused piano at 75% mark).
-   **`[Background Music]` tag** — arrangement already implies it.
-   **Negative constraints** ("no melody", "no fills") — confuse Suno. Use positive descriptors instead, e.g., "pad chords", "sparse" in Lyrics `[Instrument:]` tags. Avoid stacking multiple negative directives in the main Style block, as this leads to inconsistent and undesirable output variations.
-   **Specific vocal styles** (e.g., 'deep chanting', 'whispered') — these belong in Lyrics using `[Vocal Style:]` tags.
-   **Specific instrument roles or behaviors** (e.g., "shamisen melodic plucks", "harp background pad chords", "brushed drums sparse") — these are generally better placed in Lyrics using `[Instrument:]` tags for per-section control. *However, see "Advanced Multi-Layered Control" below for exceptions regarding global placement/subordination.*
-   **Ambient SFX and direct atmospheric tags** (e.g., 'rain', 'thunder') — these are ignored in the Style block and must be placed in the Lyrics section. *Note: Scene/space atmosphere descriptors (e.g., 'sunlit cafe') are valid and required for BGM Style blocks as part of the canonical envelope.*

### Minimal Exclude Strategy

#### Why
When every section has `[Instrument:]` typed brackets in the Lyrics, Suno has positive per-section instructions and doesn't hallucinate unwanted instruments. This positive guidance replaces the need for extensive negative exclusions. Adding abstract terms confuses Suno, as it struggles to interpret "what NOT to feel" compared to "what instrument NOT to use." Excludes should be moved to the Exclude Style block only; the main prompt should describe what is wanted.

#### How to apply
Only add dealbreakers (e.g., auto-tune for raw vocal, electronic for acoustic genres). Strategic exclusion of instruments can be effectively used to prevent specific, unwanted genre-associated sonic artifacts or 'hallucinations' that Suno might inject (e.g., adding guitar to Exclude to prevent 'skank hallucination' in reggae/hip-hop contexts), even if the instrument isn't generally problematic. Otherwise, leave the Exclude Style block empty. Do not add abstract terms like: `aggressive, uptempo, crescendo, dramatic build, drop, bright melody, distortion`.

## 2. Suno Lyrics: Unified Typed Bracket Framework

Typed brackets per section provide precise control over instrumentation, energy, mood, texture, vocal style, and advanced structural/performance tags. Lyrics staging tags should follow the same rule as Style block elements: 3-5 elements per tag, title case, active verbs, atmospheric/role descriptors. Avoid negative directives.

### Core Rule
Typed brackets per section for ALL tracks (instrumental and vocal). Pipe (`|`) inside any bracket = play together. Separate untyped bracket lines = layers (intentional additions).

### Unified Typed Bracket Types

| Type | Purpose | Examples | Behavior |
|------|---------|---------|----------|
| `[Energy:]` | Section energy | `[Energy: Low]`, `[Energy: Rising]`, `[Energy: Maximum]`, `[Energy: Medium→High]` | Controls intensity and drive. Gradient notation works. |
| `[Instrument:]` | Per-section instrumentation and dynamic roles | `[Instrument: Shamisen alone | single slow pluck]`, `[Instrument: Harp active fill/call-response]`, `[Instrument: Shamisen lazy jazz phrases | warm Rhodes underneath | brushed drums barely there]` | Positive per-section instructions for instruments. **Using directive verbs (e.g., 'active fill', 'plucks', 'phrases') effectively guides Suno's interpretation of instrument behavior and dynamic roles beyond static background elements.** Pipes combine elements. **Refinement:** For instruments like shamisen, directive verbs implying increasing intensity (e.g., 'building') can inadvertently generate undesirable timbres such as 'twang'. Opt for more controlled descriptors like 'steady' to achieve desired sonic characteristics and avoid unintended artifacts. |
| `[Mood:]` | Section mood/atmosphere | `[Mood: Morning quiet]`, `[Mood: Triumphant]`, `[Mood: Anticipation]` | Guides emotional tone and ambiance. |
| `[Texture:]` | Section sonic quality | `[Texture: Tape-Saturated]`, `[Texture: Lo-fi]` | Adds specific audio characteristics. |
| `[Vocal Style:]` | Vocal delivery direction (vocal only) | `[Vocal Style: Power]`, `[Vocal Style: Belt]`, `[Vocal Style: Open, Confident]` | Directs how vocals are performed. **Refinement:** 'Projected vocals' is more precise and effective than 'Powerful vocals'. Precise, positive directive verbs and phrasing in vocal delivery instructions (e.g., within `[Vocal Style:]` or implied by lyrical phrasing) are highly effective in guiding Suno towards desired melodicism and overcoming issues like monotony. |
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

#### Known Unreliable Structural Tags
#### Why
Certain structural tags have been consistently flagged by critics as unreliable, leading to inconsistent or undesirable outputs.
#### How to apply
-   The `[Fade Out]` structural tag is confirmed to be unreliable. Minimize its use or manage fade-outs carefully through other means (e.g., `[Outro]` with specific instrumentation fading).

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

### 3.1. Descriptive vs. Prescriptive Prompting

#### Why
Descriptive prompts work *with* Suno's training, guiding it towards desired qualities natively implied by genre and positive descriptors. Prescriptive prompts, especially stacked negative directives, fight Suno's training, leading to unstable, oscillating failures (e.g., cycling through contradictory vocal deliveries like clean→spoken→falsetto→falsetto).

#### How to apply
-   **Prioritize descriptive language:** Focus on describing what you *want* to hear, rather than what you *don't* want.
-   **Leverage genre:** Pick a genre/sub-genre that natively has the qualities you want. This implicitly handles many "vetoes" without needing explicit negative constraints.
-   **Mirror successful source prompts:** When creating variants, precisely mirror the structure of successful source prompts, especially their use of positive, descriptive language and lack of negative directives.

### 3.2. Word Repetition and Weighting

#### Why
Word repetition in a Suno Style block or within typed brackets doubles that word's interpretive weight, forcing Suno to lock into the strongest meaning and potentially override other tonal cues. This can lead to unintended sonic characteristics, such as monotone delivery when a word like `chant` is repeated.

#### How to apply
-   **Avoid repeating content words:** Do not repeat significant content words across elements within the Style block or closely related typed brackets.
-   **Use synonyms and counter-cues:** If similar concepts are needed in multiple elements, balance them with synonyms or counter-cues. For example, instead of `chant phrases` and `chant harmonies`, use `melodic phrases` and `harmonies low and lyrical` to suggest a chant aesthetic without forcing a monotone.
-   **Be aware of specific word pulls:** Words like `chant` specifically pull towards monotone in Suno. If used, it should be balanced with explicit melodic cues (e.g., `melodic phrases`, `lyrical`, `lament`).
-   **Employ surgical word swaps:** When the overall prompt structure is working but a specific sonic detail is off, identify the exact word(s) producing the unwanted pull and make minimal, surgical word swaps rather than rebuilding the entire prompt.

### Complexity Limits & Signal Clarity

#### Why
Fewer ingredients lead to clearer signals and cleaner audio. Over-stuffing prompts results in muddled output and degraded audio quality. Suno interprets every descriptor as a variable to satisfy, diluting the core intent. Audio quality degrades with excessive prompt length. Abstract or ambiguous 'vibe' terms can lead to unpredictable outputs. Overlapping instrument registers cause frequency fighting. A high count of descriptive elements, even if within character budget, can contribute to a lack of dynamic range or monotony.

#### How to apply
-   **Max 2 genres:** Use rhythm-clear genres.
-   **Max 3 instruments per section:** Use specific instrument names (e.g., "cello" instead of "low strings"). Overlapping registers cause frequency fighting. Consistent critic feedback validates the importance of adhering to this lean instrumentation for audio clarity. For example, organs can cause frequency stacking issues, particularly in verses. An effective strategy is to pull organs from verses (where they might clash with vocals or other instruments) and reintroduce them in choruses or bridges for impact and clarity.
-   **Max 1 rhythm source:** Usually drums or a driving bassline.
-   **Max 2 textures, max 2 mood words:** Cut in order: chords → textures → mood (keep 1) if over budget.
-   **Translate abstract terms:** Convert ambiguous 'vibe' terms (e.g., 'raw') into concrete, positive, and less ambiguous descriptors (e.g., 'organic'). Explicitly adding specific timbre anchors (e.g., 'twangy') to instrument descriptions or the Style block is an effective technique to guide Suno towards desired, unique sonic qualities for instruments, especially when aiming for cultural or specific textural characteristics. Using more precise and less ambiguous descriptors like 'harmonic beds' instead of generic terms like 'pads' can effectively prevent unwanted sonic artifacts (e.g., 'synth drift') and enhance control over instrument and texture rendering.
-   **Audio Quality vs. Length:** Aim for 9.5/10 clarity with a lean Style block. Less information generally yields cleaner audio.

### Effects & Audio Clarity

#### Why
Combining certain effects and descriptors can lead to undesirable sonic artifacts that reduce clarity.

#### How to apply
-   Be cautious when combining 'echo delay' with 'ambient' descriptors, as this can lead to an undesirable 'wash' effect, potentially reducing clarity or over-saturating the soundscape. Consider alternative effect applications or more precise layering.

### Genre & Cultural Label Precision

#### Why
Suno interprets cultural labels as instructions to "add everything from this tradition," leading to instrument flooding and overriding specific instrument intent. Cultural labels trigger unrequested genre-associated instruments (e.g., violin, fiddle for "Celtic") even with explicit exclusions. Explicitly avoiding cultural or regional labels in the Style block is crucial to prevent the injection of unrequested, culturally-associated instruments (e.g., koto for 'Japanese'). This provides concrete validation of the principle, demonstrating the specific risks associated with such labels. Specific genre archetypes can also introduce subtle audio quality issues. Using cultural or folk genre tags in the Style block can disproportionately elevate associated instruments to a lead role, even when other instruments are specified as primary, overriding the intended instrument hierarchy. While cultural genre labels are generally discouraged for new prompts, mirroring them from a *proven source prompt* in a variant workflow might not lead to outright failure, but can still introduce subtle quality issues like monotony.

#### How to apply
-   **Never use a culture, tradition, or region as a genre label** in Suno Style blocks (e.g., "Japanese folk fusion", "Folk") for *new* prompts.
-   Use **neutral instrument labels** (e.g., "harp" instead of "Celtic harp"). Describe the sound with adjectives instead.
-   The specific instrument in the prompt already signals the cultural flavor.
-   **Genre Naming:** While single-word genres are generally preferred for clarity (e.g., "Jazz" not "Jazz fusion"), certain two-word, commonly understood sub-genres like 'lo-fi jazz' can be effectively used, indicating Suno's ability to interpret them as coherent stylistic entities. However, specific genre archetypes (e.g., 'Jazz Lounge') can introduce subtle audio quality issues like 'instrument bleed' due to Suno's interpretation. Critics are effective at pinpointing these genre-specific pitfalls, necessitating prompt adjustments such as exclusion or more precise instrument control.
-   **Pattern:** `{rhythm-template} {instrument} fusion` (e.g., "Jazz shamisen fusion"). The genre gives Suno the rhythm template (jazz = swing, brushed drums). The instrument gives the cultural color (shamisen = Japanese). For leaner blocks, a single genre word like "Jazz" is often sufficient.
-   **Validated Archetypes:** A pre-tested genre archetype, even if it triggers 'genre-flood' warnings from critics, can sometimes override individual critic flags due to its proven robustness.
-   When choosing between similar or related genres, some (e.g., 'Roots Reggae') may provide a more robust and consistent 'anchor' for Suno's interpretation than others (e.g., 'Reggae Dub'). Prioritize genres with proven stronger anchoring for higher fidelity to the intended style.

## 4. Advanced Multi-Layered Control

Subtle global sonic characteristics, such as brightness or specific instrument placement/subordination, and complex thematic/vocal styles can be effectively controlled through a multi-layered approach.

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
For instrument subordination, use descriptive language like 'distant/low drone' rather than repeated 'muffled' descriptors. 'Subdued' is a more precise and effective descriptor for achieving a background or less prominent instrument presence compared to 'muffled', which can risk the instrument being entirely removed or poorly rendered by Suno. The MiniMax critic is effective at identifying detrimental layering or over-muffling that would effectively erase an instrument from the mix, so heed its warnings.

### Archetype Blends

#### Why
Explicitly defining primary and secondary archetypes with specific roles allows for complex, multi-layered thematic and vocal styles within a single track.

#### How to apply
-   Define a 'primary + secondary' archetype blend with specific vocal or thematic roles (e.g., 'chanting' for primary, 'spoken word' for secondary) as an effective prompt design strategy.

## 5. What Fails & What Works

### What Fails
-   **Separate untyped instrument lines** (`[Saxophone]` then `[Harp]` then `[Drums]`) = competing layers = chaos.
-   **"Duet" framing** = equal weight = instruments battle.
-   **Over-describing instruments** in the Lyrics section leads to Suno layering everything simultaneously.

### What Works
-   **`[Instrument: Shamisen jazz phrases | warm Rhodes underneath | brushed drums barely there]`** — all in one typed bracket with pipes.
-   **Atmosphere cues as mood** — `[Mood: Morning quiet]`, `[Mood: Still]`.
-   **Support solo sections** — give support instruments their own section instead of fighting for shared space.
-   When aiming for a 'skank' rhythm in reggae/hip-hop, use the descriptor 'offbeat strum' instead of 'skank' to guide Suno more effectively and avoid problematic interpretations, refining the existing 'skank hallucination' exclusion advice by providing a positive alternative.

---
Consolidated from: `suno-prompt-construction-guide.md`, `suno-prompt-engineering-principles.md`, `20260329-155500-for-ambient-sfx-and-atmospheric-elements-1.md`, `20260331-041608-suno-appears-to-interpret-fade-in-and-po.md`, `20260331-193044-the-upper-end-of-the-flexible-character--3.md`, `20260405-122651-explicitly-defining-a-primary-secondary--2.md`, `20260406-061139-explicitly-avoiding-cultural-or-regional-1.md`, `20260406-061139-strategic-exclusion-of-instruments-can-b-3.md`, `20260406-061139-subdued-is-a-more-precise-and-effective--2.md`, `20260406-064001-explicitly-adding-specific-timbre-anchor-3.md`, `20260406-064001-using-more-precise-and-less-ambiguous-de-2.md`, `20260406-070153-suno-can-effectively-interpret-and-rende-3.md`, `20260406-070153-using-directive-verbs-within-instrument--2.md`, `20260406-173715-consistent-critic-feedback-validates-the-3.md`, `20260406-173715-style-blocks-should-consistently-use-com-2.md`, `20260406-173715-when-aiming-for-a-skank-rhythm-in-reggae-1.md`, `20260406-205642-be-cautious-when-combining-echo-delay-wi-2.md`, `20260406-205642-for-instruments-like-shamisen-directive--1.md`, `20260406-205642-when-choosing-between-similar-or-related-3.md`, `20260409-003327-bgm-style-envelope-checklist.md`, `20260409-051540-descriptive-prompts-beat-prescriptive-vetoes.md`, `20260409-051540-word-repetition-doubles-suno-weight.md`, `20260409-071819-precise-positive-directive-verbs-and-phr-3.md`, `20260409-071819-stacked-negative-constraints-confuse-sun-1.md`, `20260409-071819-while-cultural-genre-labels-are-generall-2.md`
