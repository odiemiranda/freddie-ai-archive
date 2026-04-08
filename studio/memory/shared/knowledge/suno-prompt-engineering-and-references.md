---
name: Suno Prompt Engineering and References
description: Unified syntax, best practices, and advanced techniques for Suno AI music generation, including lyrical structure (energy arcs), style block length, genre normalization, vocal style, arrangement, common pitfalls, and observed undocumented behaviors.
type: knowledge
agent: shared
tags: [suno, typed-brackets, instrument, energy, mood, texture, vocal-style, unified, prompt-design, clarity, performance-notation, ad-libs, genre-recipes, tempo, key, music-theory, special-techniques, char-budget, prompt-optimization, variation, control, weight, suno-interpretation, gradient, fine-tuning, genre-normalization, lo-fi-jazz, nuance, critic-feedback, prompt-refinement, instrumentation, arrangement, frequency-stacking, organ, audio-quality, sfx, atmosphere, subordination, multi-layered-control, platform-behavior, fade-in, undocumented-feature, energy-arc, lyrical-structure, validation, repetition, archetype, layered-control, complexity, cultural-labels, instrument-hallucination, exclude, genre-artifacts, dealbreaker, descriptor-refinement, timbre, dynamic-roles, instrument-density, structural-tags, fade-out, reliability, style-block, syntax, formatting, genre-specific, reggae, hip-hop, instrument-behavior, descriptor-choice, delay, ambient, directive-verbs, melodicism, negative-constraints, prompt-failure, prompt-complexity, variant-workflow, monotony, descriptor-count, creative_generation, efficiency, new-technique]
---

# Suno Prompt Engineering and References

## Prompt Design Philosophy: Descriptive vs. Prescriptive
For optimal results and to reduce iteration count when generating music with Suno, prompts should be descriptive (setting a scene, mood, or theme) rather than prescriptive (dictating specific musical elements too rigidly). Descriptive prompts allow Suno's underlying model more creative latitude to interpret and generate, leading to better outcomes.

**Why:** Enhances creative output and reduces iteration by leveraging Suno's generative capabilities through broader guidance.
**How to apply:** Focus on conveying the desired atmosphere, emotion, or narrative through descriptive language in your prompts, rather than over-specifying individual musical components.

## Core Rule: Unified Typed Brackets
Typed brackets per section are the single pattern for ALL Suno tracks. Use a pipe (`|`) inside any bracket to indicate elements that should play together as an ensemble. Separate untyped bracket lines can lead to layering conflicts. This pipe syntax has been confirmed to yield the same DNA as stacked instrument tags at certain `W`/`SI` settings, but is cleaner.

**Why:** Ensures consistent, high-quality AI music generation by providing clear, structured instructions to Suno models.
**How to apply:** Always use the specified typed bracket syntax for all prompt details.

## Typed Bracket Types
- `[Energy: Low/Rising/High/Maximum]` — section energy
- `[Instrument: lead | support | rhythm]` — per-section instrumentation (pipe = ensemble)
- `[Mood: descriptor]` — section mood
- `[Texture: descriptor]` — section texture/production
- `[Vocal Style: descriptor]` — vocal direction (vocal tracks only). For example, 'Projected vocals' is more precise and effective than 'Powerful vocals'.

## Vocal Style Specificity
When describing vocal styles, prioritize precise descriptors. 'Projected vocals' has been identified as a more effective instruction than 'Powerful vocals', which can be too broad or lead to less desirable outcomes. Precise, positive directive verbs and phrasing (e.g., 'melodic phrases' over 'chant phrases') are highly effective in guiding Suno towards desired melodicism and overcoming issues like monotony. This highlights the power of targeted positive phrasing over general mood or energy adjustments for vocal performance.

**Why:** Improves clarity and generation quality, aligning with critic feedback for specificity, and ensures desired melodicism.
**How to apply:** Use precise vocal style descriptors like 'Projected vocals' and positive directive verbs (e.g., 'melodic phrases') to guide Suno's vocal performance.

## Instrument and Texture Control
When describing instruments, especially support instruments, use specific instrument names (e.g., 'cello', 'bodhrán') instead of broader categories (e.g., 'low strings', 'percussion').

For instrument subordination, descriptive language like 'distant/low drone' is more effective than repeated 'muffled' descriptors, which can lead to the instrument being entirely removed. 'Subdued' is also a more precise and effective descriptor for achieving a background or less prominent instrument presence compared to 'muffled'. The MiniMax critic is effective not only at identifying detrimental layering or over-muffling but also at detecting sonic redundancy or lack of variation, indicating its utility for ensuring dynamic range and interest in a track.

### Descriptor Refinement and Timbre Anchors
Explicitly adding specific timbre anchors (e.g., 'twangy') to instrument descriptions or the Style block is an effective technique to guide Suno towards desired, unique sonic qualities for instruments, especially when aiming for cultural or specific textural characteristics.
Using more precise and less ambiguous descriptors like 'harmonic beds' instead of generic terms like 'pads' can effectively prevent unwanted sonic artifacts (e.g., 'synth drift') and enhance control over instrument and texture rendering.

### Dynamic Instrument Roles and Directive Verbs
Suno can effectively interpret and render specific, dynamic instrument roles (e.g., 'active fill/call-response' for a harp) beyond static background elements, especially when articulated with directive verbs in `[Instrument:]` tags. Using directive verbs within `[Instrument:]` typed brackets is an effective and validated technique for precisely guiding Suno's interpretation of instrument behavior and roles, contributing to successful outcomes.
**Refinement for Specific Instruments:** For instruments like shamisen, directive verbs implying increasing intensity (e.g., 'building') can inadvertently generate undesirable timbres such as 'twang'. This 'twang' artifact has been observed with 'Building' in prompts. Opt for more controlled descriptors like 'steady' to achieve desired sonic characteristics and avoid unintended artifacts.

**Why:** Improves clarity and generation quality, aligning with critic feedback for specificity, and ensures instruments are present and balanced as intended. Precise descriptors and timbre anchors allow for nuanced sonic control, while directive verbs enable dynamic instrument roles. The refinement provides specific guidance for avoiding problematic timbres with certain instruments and directive verb combinations.
**How to apply:** Replace generic instrument descriptions with precise names in `[Instrument:]` tags. For subordination, use 'distant/low drone' or 'subdued' language instead of 'muffled' to avoid accidental removal. Integrate specific timbre anchors (e.g., 'twangy') and refined descriptors (e.g., 'harmonic beds') for desired sonic qualities. Use directive verbs in `[Instrument:]` tags to define dynamic instrument behaviors, being mindful that some verbs (e.g., 'building') can produce unintended timbres for specific instruments (e.g., shamisen), in which case 'steady' or similar controlled descriptors are preferred.

## Instrumentation and Arrangement Best Practices
To maintain audio clarity and prevent frequency stacking, especially in vocal-heavy sections, limit instrumentation to a maximum of 3 instruments per section. For example, strategically place instruments like organs to avoid stacking, pulling them from verses and reintroducing them in choruses or bridges for impact.

### Effects and Audio Clarity
Be cautious when combining 'echo delay' with 'ambient' descriptors, as this can lead to an undesirable 'wash' effect, potentially reducing clarity or over-saturating the soundscape. This 'wash' effect has been observed when 'echo/delay/ambient' are combined. Consider alternative effect applications or more precise layering.

**Why:** Prevents frequency clashes and improves overall mix clarity and impact by enforcing lean instrumentation. The caution regarding effects combinations helps maintain audio quality and prevents unintended sonic artifacts.
**How to apply:** Strategically place instruments like organs to avoid stacking, especially in vocal-heavy sections, and adhere to a maximum of 3 instruments per section. When using effects, be mindful of combinations like 'echo delay' with 'ambient' to avoid a 'wash' effect; consider alternative approaches for clarity.

## Genre Specificity and Normalization
While single-word genres are generally preferred for clarity, certain two-word, commonly understood sub-genres can be effectively used in Style Blocks. Suno's interpretation of genre tags can lead to the emergence of related instruments even without explicit tagging (e.g., "fiddle" from "folk jazz").

Cultural or folk genre tags in the Style block can disproportionately elevate associated instruments to a lead role, even when other instruments are specified as primary. Explicitly avoiding cultural or regional labels in the Style block is crucial to prevent the injection of unrequested, culturally-associated instruments (e.g., koto for 'Japanese'). While cultural genre labels are generally discouraged for new prompts, mirroring them from a *proven source prompt* in a variant workflow might not lead to outright failure, but can still introduce subtle quality issues like monotony. This reinforces the principle of avoiding such tags to maintain precise control over instrument hierarchy and prevent unintended instrument prominence.

When aiming for a 'skank' rhythm in reggae/hip-hop, use the descriptor 'offbeat strum' instead of 'skank' to guide Suno more effectively and avoid problematic interpretations. Observations confirm that 'skank' can be flagged by Suno or lead to unexpected interpretations. This refines the existing advice to prevent 'skank hallucination' by providing a positive alternative to exclusion.

### Genre Archetype Pitfalls
Specific genre archetypes can introduce subtle audio quality issues like 'instrument bleed' due to Suno's interpretation. Critics are effective at pinpointing these genre-specific pitfalls, necessitating prompt adjustments such as exclusion or more precise instrument control. For example, the 'Jazz Lounge' descriptor has been observed to lead to undesirable 'instrument bleed'.

### Genre Anchoring Strength
When choosing between similar or related genres, some (e.g., 'Roots Reggae') may provide a more robust and consistent 'anchor' for Suno's interpretation than others (e.g., 'Reggae Dub'). Prioritize genres with proven stronger anchoring for higher fidelity to the intended style.

**Why:** Improves clarity and generation quality while acknowledging Suno's ability to interpret coherent stylistic entities beyond single-word tags and its "genre gravity." Prevents unintended instrument prominence and addresses specific audio quality issues. Understanding genre anchoring helps select more effective style descriptors.
**How to apply:** Prioritize single-word genres. For specific, well-established sub-genres (e.g., 'lo-fi jazz'), use the two-word descriptor if it accurately conveys the desired style. Be aware that genre tags can subtly influence instrumentation. Avoid cultural or folk genre tags in the Style block if precise instrument hierarchy is critical, and be aware that mirroring them from a proven source may still lead to monotony. When genre archetypes cause issues like 'instrument bleed' (e.g., 'Jazz Lounge'), adjust prompts with exclusions or more precise instrument control. For reggae/hip-hop 'skank' rhythms, use 'offbeat strum' instead of 'skank'. When selecting between similar genres, opt for those with proven stronger 'anchoring' effects (e.g., 'Roots Reggae' over 'Reggae Dub') for more consistent results.

## Section Order
Follow this structure for optimal results:
Structure tag → Energy → Mood → Vocal Style (if vocal) → Instrument → Texture → Lyrics (if vocal)

## Style Block Strategy
Style blocks should consistently use comma-separated descriptors (e.g., `genre, BPM, key, mood`), not pipe delimiters. Pipe delimiters (`|`) are reserved for combining elements within typed brackets in the Lyrics section (e.g., `[Instrument: lead | support | rhythm]`). The Style Block should be ultra-lean, typically targeting 75-120 characters. It should include genre, BPM, key, vocal character (if vocal), and overall mood. All specific detail for sections should be placed within typed brackets in the Lyrics section. Additionally, a high count of descriptive elements, even if within character budget, can contribute to a lack of dynamic range or monotony, reinforcing the need for lean prompts.

For ambient SFX and atmospheric elements to be effectively processed by Suno, they must be placed within the Lyrics section using specific typed brackets (e.g., `[Effect: Rain]`, `[Instrument: Rain]`, or `[Mood: Rainy]`). The Style block should be reserved for global genre and core instrumentation.

**Why:** Focuses the initial generation on core characteristics while allowing granular control within the lyrics. Allows for a slightly expanded Style Block when per-section control is robust. Ensures SFX are correctly interpreted.
**How to apply:**
- **Character Range:** Target 75-120 characters. However, when comprehensive per-section control is achieved via typed brackets in Lyrics, the Style Block can effectively operate up to ~150 characters without degrading audio quality, provided it remains lean on global elements and avoids redundancy with Lyrics. This upper limit (~150 chars) has been validated as effective when combined with detailed per-section control via typed brackets in the Lyrics, maintaining audio clarity and prompt adherence.
- **Instrumental Tracks:** Lean Style Block + ensemble pipe `[Instrument:]` tags within Lyrics.
- **Vocal Tracks:** Ultra-lean Style Block + rich, typed bracket `[Energy:]`, `[Vocal Style:]`, `[Instrument:]`, `[Texture:]`, `[Mood:]` tags within Lyrics.
- **SFX/Atmosphere:** Place in Lyrics section using typed brackets, not the Style Block.
- **Formatting:** Ensure Style blocks use comma-separated descriptors.

## Exclude Style
Keep the Exclude Style minimal or empty. Per-section `[Instrument:]` tags provide positive constraints that effectively replace the need for extensive excludes. Testing confirms that an empty Exclude Style works well when typed brackets provide sufficient instrument control.

However, strategic exclusion can be part of a multi-layered control strategy for subtle sonic characteristics. For example, explicitly excluding 'violin' can help ensure a 'fiddle' is rendered as intended when combined with other controls. Strategic exclusion of instruments can also be effectively used to prevent specific, unwanted genre-associated sonic artifacts or 'hallucinations' that Suno might inject (e.g., adding guitar to Exclude to prevent 'skank hallucination'). Conversely, attempting to control vocal delivery with multiple negative constraints (e.g., 'no falsetto', 'narrow range', 'glacial cadence') confuses Suno and leads to inconsistent and undesirable output variations, often cycling through contradictory interpretations of vocal delivery. This reinforces the principle to avoid stacked negative constraints, as they are a specific failure mode.

**Why:** Positive constraints are generally more effective than negative ones. Strategic exclusion can be used as a targeted tool within a broader control strategy to prevent unwanted instrument bleed or specific sonic artifacts. Avoiding stacked negative constraints prevents unpredictable and contradictory output.
**How to apply:** Rely on detailed `[Instrument:]` tags rather than extensive exclude lists. Only use `Exclude` for specific, targeted control when part of a multi-layered approach to prevent unwanted instrument bleed, ensure a desired instrument is rendered uniquely, or block specific genre-associated 'hallucinations'. Absolutely avoid stacked negative constraints for vocal delivery.

## Layers (Intentional Additions)
Intentional additions between lyrics should also use brackets or clear descriptions, e.g., `[instrumental break saxophone]`, `[Whispered]`, `(Hey! Hey!)`. This includes ambient SFX and atmospheric elements.

**Why:** Provides granular control over non-lyrical vocalizations, instrumental breaks, and environmental sounds.
**How to apply:** Integrate these bracketed descriptions directly into the lyrics section where the event should occur.

## Multi-Layered Prompt Control
Subtle global sonic characteristics, such as brightness or specific instrument placement, can be effectively controlled through a multi-layered approach that combines:
1.  **Avoiding problematic genre tags:** Prevents unintended instrument prominence.
2.  **Descriptive language in the Style block:** For global texture/mood (e.g., 'distant low fiddle drone').
3.  **Precise instrument phrasing within Lyrics `[Instrument:]` tags:** For per-section control (e.g., 'underneath').
4.  **Strategically excluding closely related instruments:** To ensure the desired instrument is rendered as intended (e.g., 'violin' in Exclude to emphasize 'fiddle').
5.  **Global sonic descriptors:** (e.g., 'rolled-off highs' for brightness control).

**Why:** Provides comprehensive and precise control over nuanced sonic characteristics that single-layer prompting might miss.
**How to apply:** Implement a combination of these strategies, considering the interaction between Style Block, Lyrics, and Exclude sections, to achieve desired global sonic outcomes.

## Archetype Blending for Complex Styles
Explicitly defining a 'primary + secondary' archetype blend with specific vocal or thematic roles (e.g., 'chanting' for primary, 'spoken word' for secondary) is an effective prompt design strategy for achieving complex, multi-layered thematic and vocal styles within a single track.

**Why:** Enables the creation of nuanced, multi-layered thematic and vocal styles, enhancing creative control and depth in generated tracks.
**How to apply:** When designing prompts for complex vocal or thematic tracks, define a primary archetype with its associated vocal/thematic role (e.g., 'chanting') and a secondary archetype with its distinct role (e.g., 'spoken word').

## Lyrical Structure: Energy Arcs
The concept of an 'energy arc' with predefined shapes is a valuable and effective structural element for generating BGM lyrics, providing a high-level control over the dynamic flow and intensity of the lyrical output.

**Why:** Provides a structured method to control the dynamic flow and intensity of BGM lyrical output, enhancing creative control.
**How to apply:** Integrate predefined energy arc shapes into the lyrics generation process to guide the dynamic progression of the lyrical content.

## Suno Performance Notation
A set of 8 standardized symbols for conveying specific performance instructions.

**Why:** Provides a standardized way to convey specific performance instructions to Suno models, ensuring consistent interpretation.
**How to apply:** Use the 8 defined symbols within prompts to guide vocal or instrumental performance.

## Suno Ad-Lib Table
A structured reference for common ad-lib vocalizations and their typical placements.

**Why:** Offers a structured reference for incorporating common ad-lib vocalizations and their typical placements, enriching vocal tracks.
**How to apply:** Consult the ad-lib table for appropriate ad-lib choices and integrate them into vocal prompts.

## Suno Reference Tags & Recipes
Comprehensive lookup for atmosphere, ambient, genre-specific elements, instrument/production tags, tempo/key, and translation of music theory to prompt tags.

**Why:** Provides a comprehensive lookup for atmosphere, ambient, genre-specific elements, instrument/production tags, tempo/key, and translates music theory to prompt tags, enhancing prompt accuracy and richness.
**How to apply:** Utilize these references to construct detailed and precise Style Blocks and typed bracket sections.

## Suno Special Techniques
Defined formats for advanced vocal arrangements, including duet and call-and-response.

**Why:** Enables the creation of more complex vocal arrangements like duets and call-and-response patterns, expanding creative possibilities.
**How to apply:** Follow the specified formats for duet and call-and-response structures in vocal prompts.

## Undocumented but Observed Suno Behaviors
Suno appears to interpret `[Fade In]` (and potentially `[Short Fade In]`) as a valid structural/performance tag for controlling track intros, even though it is not explicitly documented in the unified typed bracket framework. Conversely, the `[Fade Out]` structural tag has been consistently flagged by critics as unreliable, reinforcing the need to minimize its use or manage it carefully.

**Why:** This suggests an undocumented capability for subtle intro control, allowing for more nuanced track beginnings, while also highlighting an unreliable tag that requires careful management.
**How to apply:** Experiment with `[Fade In]` or `[Short Fade In]` in the prompt for intro buildup. Minimize or carefully manage the use of `[Fade Out]` due to its unreliability.

## Variation Control: Weight (W) and Suno Interpretation (SI)
The 'Weight' (W) and 'Suno Interpretation' (SI) parameters offer precise control over the output spectrum of musical variations. For example, setting `W:30 / SI:70` has been shown to effectively anchor warmth, tempo, and texture from Music Cards while allowing typed brackets to handle fusion elements.

**Why:** Provides a powerful method for generating a spectrum of musical variations, effectively controlling the balance between adherence to the prompt (tightness) and creative interpretation (e.g., groove vs. BGM focus).
**How to apply:** Adjust W and SI parameters in a deliberate gradient across multiple variations to explore a desired spectrum of tightness vs. creative interpretation. For example, use a gradient from looser (higher SI, lower W) to tighter (lower SI, higher W) to control the balance between groove and background music focus.

**Consolidated from:**
- `suno-prompt-engineering.md`
- `community-reference-session.md`
- `suno-ensemble-layer-syntax.md`
- `typed-brackets-vocal-strategy.md`
- `20260327-040817-when-describing-instruments-especially-s-2.md`
- `20260327-220429-when-comprehensive-per-section-control-i-2.md`
- `20260328-025517-adjusting-the-weight-w-and-suno-interpre-2.md`
- `20260328-025517-while-single-word-genres-are-generally-p-3.md`
- `20260328-110500-music-card-mixing-v1.md` (specific W/SI application)
- `20260328-221905-for-vocal-style-descriptors-in-suno-proj-2.md`
- `20260328-221905-organs-can-cause-frequency-stacking-issu-3.md`
- `20260329-002043-specific-genre-archetypes-can-introduce--1.md` (detailed genre archetype pitfalls)
- `20260329-072140-cultural-or-folk-genre-tags-in-the-style-1.md`
- `20260331-193044-minimax-is-effective-not-only-at-identif-2.md`
- `20260329-072140-for-instrument-subordination-descriptive-2.md`
- `20260329-072140-subtle-global-sonic-characteristics-such-3.md`
- `20260329-155500-for-ambient-sfx-and-atmospheric-elements-1.md`
- `20260331-041608-suno-appears-to-interpret-fade-in-and-po.md`
- `20260331-145249-the-concept-of-an-energy-arc-with-predef-1.md`
- `20260331-193044-the-upper-end-of-the-flexible-character--3.md`
- `20260405-122651-explicitly-defining-a-primary-secondary--2.md`
- `20260406-061139-explicitly-avoiding-cultural-or-regional-1.md`
- `20260406-061139-strategic-exclusion-of-instruments-can-b-3.md`
- `20260406-061139-subdued-is-a-more-precise-and-effective--2.md`
- `20260406-064001-explicitly-adding-specific-timbre-anchor-3.md`
- `20260406-064001-using-more-precise-and-less-ambiguous-de-2.md`
- `20260406-070153-suno-can-effectively-interpret-and-rende-3.md`
- `20260406-070153-using-directive-verbs-within-instrument--2.md`
- `20260406-173715-consistent-critic-feedback-validates-the-3.md`
- `20260406-173715-style-blocks-should-consistently-use-com-2.md`
- `20260406-173715-when-aiming-for-a-skank-rhythm-in-reggae-1.md`
- `20260406-205642-be-cautious-when-combining-echo-delay-wi-2.md`
- `20260406-205642-for-instruments-like-shamisen-directive--1.md`
- `20260406-205642-when-choosing-between-similar-or-related-3.md`
- `20260406-222827-specific-musical-terms-skank-can-trigger-1.md`
- `20260409-071819-precise-positive-directive-verbs-and-phr-3.md`
- `20260409-071819-stacked-negative-constraints-confuse-sun-1.md`
- `20260409-071819-while-cultural-genre-labels-are-generall-2.md`
- `20260409-072505-for-optimal-results-and-to-reduce-iterat-3.md`
