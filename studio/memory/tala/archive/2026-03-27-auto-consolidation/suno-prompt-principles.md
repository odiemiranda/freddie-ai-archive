---
name: suno-prompt-principles
description: Core principles for clean, high-signal prompt design, complexity limits, genre precision, quality expectations, and workflow feedback.
type: knowledge
agent: tala
tags: [suno, prompt-design, complexity, genre, quality, reroll, multi-critic, llm-feedback, iteration, signal-clarity]
---

# Suno Prompt Engineering Principles

## 1. Complexity Limits & Signal Clarity

Fewer ingredients lead to clearer signals and cleaner audio. Over-stuffing prompts results in muddled output and degraded audio quality.

### Why
Suno interprets every descriptor as a variable to satisfy, diluting the core intent. Audio quality degrades with excessive prompt length. Abstract or ambiguous 'vibe' terms can lead to unpredictable outputs.

### How to apply
-   **Max 2 genres:** Use rhythm-clear genres. Prefer single-word labels ("Jazz" not "Jazz fusion").
-   **Max 3 instruments:** Use specific instrument names (e.g., "cello" instead of "low strings"). Overlapping registers cause frequency fighting.
-   **Max 1 rhythm source:** Usually drums or a driving bassline.
-   **Max 2 textures, max 2 mood words:** Cut in order: chords → textures → mood (keep 1) if over budget.
-   **Translate abstract terms:** Convert ambiguous 'vibe' terms (e.g., 'raw') into concrete, positive, and less ambiguous descriptors (e.g., 'organic').
-   **Audio Quality vs. Length:** Aim for 9.5/10 clarity at ~75-120 characters in the Style block. Less information generally yields cleaner audio.

## 2. Genre & Cultural Label Precision

Suno interprets cultural labels as instructions to "add everything from this tradition," leading to instrument flooding.

### Why
Cultural labels trigger unrequested genre-associated instruments (e.g., violin, fiddle for "Celtic") even with explicit exclusions.

### How to apply
-   **Never use a culture, tradition, or region as a genre label** in Suno Style blocks (e.g., "Japanese folk fusion").
-   Use **neutral instrument labels** (e.g., "harp" instead of "Celtic harp"). Describe the sound with adjectives instead.
-   The specific instrument in the prompt already signals the cultural flavor.
-   **Pattern:** `{rhythm-template} {instrument} fusion` (e.g., "Jazz shamisen fusion"). The genre gives Suno the rhythm template (jazz = swing, brushed drums). The instrument gives the cultural color (shamisen = Japanese). For leaner blocks, use a single genre word like "Jazz" instead of "Jazz fusion".
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

## 4. Multi-Critic Feedback & LLM Reconciliation

Relying on a single critic for comprehensive feedback is insufficient, and LLMs can provide unreliable advice.

### Why
Different critics identify distinct issues, and no single critic reliably catches every potential problem. LLMs (like Gemini) may over-simplify or contradict established agent rules.

### How to apply
-   Implement a **multi-critic reconciliation process** to aggregate diverse insights and ensure all critical issues are identified and addressed.
-   When using LLMs for prompt refinement, be vigilant about their tendencies (e.g., stripping instrument direction from instrumental BGM lyrics, recommending unreliable structural tags). **Agent rules must override such suggestions.**

---
*Consolidated from: `suno-prompt-principles.md` (original), `20260327-040817-a-validated-genre-archetype-even-if-it-t-4.md`, `20260327-040817-abstract-or-potentially-ambiguous-vibe-t-3.md`, `20260327-040817-when-describing-instruments-especially-s-2.md`, `iteration-workflow-learnings.md`, `20260326-205216-relying-on-a-single-critic-for-comprehen-2.md` (partially), `20260326-205216-gemini-s-feedback-regarding-instrumental-1.md` (partially), `20260326-213239-when-using-llms-like-gemini-for-prompt-r-5.md` (partially)*
