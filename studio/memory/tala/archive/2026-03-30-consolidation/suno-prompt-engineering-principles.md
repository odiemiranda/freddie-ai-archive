---
name: suno-prompt-engineering-principles
description: Core principles for clean, high-signal prompt design, complexity limits, genre precision, and quality expectations for Suno, including nuanced instrumentation and genre considerations.
type: knowledge
agent: tala
tags: [suno, prompt-design, complexity, genre, quality, reroll, iteration, signal-clarity, instrumentation, arrangement, audio-quality, critic, prompt-refinement, cultural-labels]
---

# Suno Prompt Engineering Principles

## 1. Complexity Limits & Signal Clarity

Fewer ingredients lead to clearer signals and cleaner audio. Over-stuffing prompts results in muddled output and degraded audio quality.

### Why
Suno interprets every descriptor as a variable to satisfy, diluting the core intent. Audio quality degrades with excessive prompt length. Abstract or ambiguous 'vibe' terms can lead to unpredictable outputs. Overlapping instrument registers cause frequency fighting.

### How to apply
-   **Max 2 genres:** Use rhythm-clear genres.
-   **Max 3 instruments:** Use specific instrument names (e.g., "cello" instead of "low strings"). Overlapping registers cause frequency fighting. For example, organs can cause frequency stacking issues, particularly in verses. An effective strategy is to pull organs from verses (where they might clash with vocals or other instruments) and reintroduce them in choruses or bridges for impact and clarity.
-   **Max 1 rhythm source:** Usually drums or a driving bassline.
-   **Max 2 textures, max 2 mood words:** Cut in order: chords → textures → mood (keep 1) if over budget.
-   **Translate abstract terms:** Convert ambiguous 'vibe' terms (e.g., 'raw') into concrete, positive, and less ambiguous descriptors (e.g., 'organic').
-   **Audio Quality vs. Length:** Aim for 9.5/10 clarity with a lean Style block (see `suno-prompt-construction-guide.md`). Less information generally yields cleaner audio.

## 2. Genre & Cultural Label Precision

Suno interprets cultural labels as instructions to "add everything from this tradition," leading to instrument flooding and overriding specific instrument intent.

### Why
Cultural labels trigger unrequested genre-associated instruments (e.g., violin, fiddle for "Celtic") even with explicit exclusions. Specific genre archetypes can also introduce subtle audio quality issues. Using cultural or folk genre tags in the Style block can disproportionately elevate associated instruments to a lead role, even when other instruments are specified as primary, overriding the intended instrument hierarchy.

### How to apply
-   **Never use a culture, tradition, or region as a genre label** in Suno Style blocks (e.g., "Japanese folk fusion", "Folk").
-   Use **neutral instrument labels** (e.g., "harp" instead of "Celtic harp"). Describe the sound with adjectives instead.
-   The specific instrument in the prompt already signals the cultural flavor.
-   **Genre Naming:** While single-word genres are generally preferred for clarity (e.g., "Jazz" not "Jazz fusion"), certain two-word, commonly understood sub-genres like 'lo-fi jazz' can be effectively used, indicating Suno's ability to interpret them as coherent stylistic entities. However, specific genre archetypes (e.g., 'Jazz Lounge') can introduce subtle audio quality issues like 'instrument bleed' due to Suno's interpretation. Critics are effective at pinpointing these genre-specific pitfalls, necessitating prompt adjustments such as exclusion or more precise instrument control.
-   **Pattern:** `{rhythm-template} {instrument} fusion` (e.g., "Jazz shamisen fusion"). The genre gives Suno the rhythm template (jazz = swing, brushed drums). The instrument gives the cultural color (shamisen = Japanese). For leaner blocks, a single genre word like "Jazz" is often sufficient.
-   **Validated Archetypes:** A pre-tested genre archetype, even if it triggers 'genre-flood' warnings from critics, can sometimes override individual critic flags due to its proven robustness.

## 3. Prompt Quality & Reroll Expectations

A good Suno prompt doesn't guarantee a perfect generation on the first try; it guarantees that most generations will be in the right ballpark. User testing is an irreplaceable part of the workflow.

### Why
Suno's generative nature involves inherent variability. The prompt's job is to narrow the possibility space. The agent cannot hear the output, making user feedback essential.

### How to apply
-   Expect to perform **3-5 rerolls** to find at least one keeper.
-   **Quality Gate:** The prompt is working if the *worst* generation is still recognizably what was requested. If every generation is chaotic, the prompt is broken.
-   **Iteration Workflow:** Treat the first generation as a draft. When issues arise (e.g., "chaotic," "layering"), iterate on bracket syntax first, as it's the most impactful lever.
-   **Track History:** Document iteration history in Style Block Notes or similar logs to learn from past attempts.
---
Consolidated from: `suno-prompt-engineering-principles.md`, `20260329-072140-cultural-or-folk-genre-tags-in-the-style-1.md`
