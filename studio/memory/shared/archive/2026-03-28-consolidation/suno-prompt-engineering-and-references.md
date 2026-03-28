---
name: Suno Prompt Engineering and References
description: Unified syntax, best practices, and advanced techniques for Suno AI music generation, including new insights on style block length, genre normalization, and variation control.
type: knowledge
agent: shared
tags: [suno, typed-brackets, instrument, energy, mood, texture, vocal-style, unified, prompt-design, clarity, performance-notation, ad-libs, genre-recipes, tempo, key, music-theory, special-techniques, char-budget, prompt-optimization, variation, control, weight, suno-interpretation, gradient, fine-tuning, genre-normalization, lo-fi-jazz, nuance]
---

# Suno Prompt Engineering and References

## Core Rule: Unified Typed Brackets
Typed brackets per section are the single pattern for ALL Suno tracks. Use a pipe (`|`) inside any bracket to indicate elements that should play together as an ensemble. Separate untyped bracket lines can lead to layering conflicts.

**Why:** Ensures consistent, high-quality AI music generation by providing clear, structured instructions to Suno models.
**How to apply:** Always use the specified typed bracket syntax for all prompt details.

## Typed Bracket Types
- `[Energy: Low/Rising/High/Maximum]` — section energy
- `[Instrument: lead | support | rhythm]` — per-section instrumentation (pipe = ensemble)
- `[Mood: descriptor]` — section mood
- `[Texture: descriptor]` — section texture/production
- `[Vocal Style: descriptor]` — vocal direction (vocal tracks only)

## Instrument Specificity
When describing instruments, especially support instruments, use specific instrument names (e.g., 'cello', 'bodhrán') instead of broader categories (e.g., 'low strings', 'percussion').

**Why:** Improves clarity and generation quality, aligning with critic feedback for specificity.
**How to apply:** Replace generic instrument descriptions with precise names in `[Instrument:]` tags.

## Genre Specificity and Normalization
While single-word genres are generally preferred for clarity, certain two-word, commonly understood sub-genres can be effectively used in Style Blocks.

**Why:** Improves clarity and generation quality while acknowledging Suno's ability to interpret coherent stylistic entities beyond single-word tags.
**How to apply:** Prioritize single-word genres. For specific, well-established sub-genres (e.g., 'lo-fi jazz'), use the two-word descriptor if it accurately conveys the desired style.

## Section Order
Follow this structure for optimal results:
Structure tag → Energy → Mood → Vocal Style (if vocal) → Instrument → Texture → Lyrics (if vocal)

## Style Block Strategy
The Style Block should be ultra-lean, typically targeting 75-120 characters. It should include genre, BPM, key, vocal character (if vocal), and overall mood. All specific detail for sections should be placed within typed brackets in the Lyrics section.

**Why:** Focuses the initial generation on core characteristics while allowing granular control within the lyrics. Allows for a slightly expanded Style Block when per-section control is robust.
**How to apply:**
- **Character Range:** Target 75-120 characters. However, when comprehensive per-section control is achieved via typed brackets in Lyrics, the Style Block can effectively operate up to ~150 characters without degrading audio quality, provided it remains lean on global elements and avoids redundancy with Lyrics.
- **Instrumental Tracks:** Lean Style Block + ensemble pipe `[Instrument:]` tags within Lyrics.
- **Vocal Tracks:** Ultra-lean Style Block + rich, typed bracket `[Energy:]`, `[Vocal Style:]`, `[Instrument:]`, `[Texture:]`, `[Mood:]` tags within Lyrics.

## Exclude Style
Keep the Exclude Style minimal or empty. Per-section `[Instrument:]` tags provide positive constraints that effectively replace the need for extensive excludes.

**Why:** Positive constraints are more effective than negative ones for guiding AI generation.
**How to apply:** Rely on detailed `[Instrument:]` tags rather than exclude lists.

## Layers (Intentional Additions)
Intentional additions between lyrics should also use brackets or clear descriptions, e.g., `[instrumental break saxophone]`, `[Whispered]`, `(Hey! Hey!)`.

**Why:** Provides granular control over non-lyrical vocalizations or instrumental breaks.
**How to apply:** Integrate these bracketed descriptions directly into the lyrics section where the event should occur.

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

## Variation Control: Weight (W) and Suno Interpretation (SI)
The 'Weight' (W) and 'Suno Interpretation' (SI) parameters offer precise control over the output spectrum of musical variations.

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
