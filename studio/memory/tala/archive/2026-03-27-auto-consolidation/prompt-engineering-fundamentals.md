---
name: prompt-engineering-fundamentals
description: Core principles for clean, high-signal prompt design, complexity limits, genre precision, and quality expectations.
type: knowledge
agent: tala
tags: [suno, prompt-design, complexity, genre, quality, reroll]
---

# Prompt Engineering Fundamentals

## 1. Complexity Limits & Signal Clarity
Fewer ingredients = clearer signal. Over-stuffing prompts leads to muddled output.
- **Max 2 genres:** Use rhythm-clear genres (e.g., "Jazz fusion", "Reggae fusion"). Avoid vague labels (e.g., "Groove pop").
- **Max 3 instruments:** Overlapping registers cause frequency fighting.
- **Max 1 rhythm source:** Usually drums or a driving bassline.
- **Max 2 textures, max 2 mood words:** Every additional descriptor is another variable Suno tries to satisfy, diluting the core intent.
- **Leaner Style Blocks:** Target ~200-250 chars for instrumental, ~250-350 chars for vocal. Hard max: 350 instrumental, 500 vocal. First tag = ~60% influence, second = ~25%, third = ~10%, fourth = ~5%. Genre and lead instrument must be the first two elements. Optimal: 3-5 tags total. When over budget, cut in order: chords → textures → mood (keep 1). Never cut genre, lead instrument, key/BPM, or production keywords ("clean mix", "separated instruments"). Audio quality degrades with length: 9.5/10 at 500-1500 chars → 8.0/10 at 3500-5000 chars.

## 2. Genre & Cultural Label Precision
Suno interprets cultural labels as instructions to "add everything from this tradition," leading to instrument flooding.
- **Why:** Cultural labels trigger unrequested genre-associated instruments (e.g., violin, fiddle for "Celtic") even with excludes.
- **How to apply:**
    - **Never use a culture, tradition, or region as a genre label** in Suno Style blocks (e.g., "Japanese folk fusion").
    - Use **neutral instrument labels** (e.g., "harp" instead of "Celtic harp"). Describe the sound with adjectives instead.
    - The specific instrument in the prompt already signals the cultural flavor.
    - **Pattern:** `{rhythm-template} {instrument} fusion` (e.g., "Jazz shamisen fusion"). The genre gives Suno the rhythm template (jazz = swing, brushed drums). The instrument gives the cultural color (shamisen = Japanese).

## 3. Prompt Quality & Reroll Expectations
A good Suno prompt doesn't guarantee a perfect generation on the first try; it guarantees that most generations will be in the right ballpark.
- **Why:** Suno's generative nature involves variability. The prompt's job is to narrow the possibility space.
- **How to apply:**
    - Expect to perform **3-5 rerolls** to find at least one keeper.
    - **Quality Gate:** The prompt is working if the *worst* generation is still recognizably what was requested. If every generation is chaotic, the prompt is broken.

**Consolidated from:** instrumental-prompt-engineering.md, 20260326-generation-is-selection.md, 20260326-genre-label-precision.md, 20260326-style-block-formula.md
