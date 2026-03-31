---
name: suno-quality-and-iteration
description: Guidelines for managing quality expectations, the iterative reroll process, and insights into internal tool performance for Suno prompt engineering.
type: knowledge
agent: tala
tags: [suno, quality, reroll, iteration, critic, prompt-refinement, internal-tool, tool-feedback]
---

# Suno Quality and Iteration

This document outlines expectations for Suno output quality, the iterative process of prompt refinement, and observations on internal tool performance.

## 1. Prompt Quality & Reroll Expectations

A good Suno prompt doesn't guarantee a perfect generation on the first try; it guarantees that most generations will be in the right ballpark. User testing is an irreplaceable part of the workflow.

### Why
Suno's generative nature involves inherent variability. The prompt's job is to narrow the possibility space. The agent cannot hear the output, making user feedback essential.

### How to apply
-   Expect to perform **3-5 rerolls** to find at least one keeper.
-   **Quality Gate:** The prompt is working if the *worst* generation is still recognizably what was requested. If every generation is chaotic, the prompt is broken.
-   **Iteration Workflow:** Treat the first generation as a draft. When issues arise (e.g., "chaotic," "layering"), iterate on bracket syntax first, as it's the most impactful lever.
-   **Track History:** Document iteration history in Style Block Notes or similar logs to learn from past attempts.

## 2. Internal Tool Quality Concerns

### Tracknames Tool Quality Concern

#### Observation
The 'tracknames' internal tool generated track names that were noted for 'quality concern (fatigue words)'.

#### Learning
The 'tracknames' tool may produce repetitive or low-quality suggestions ('fatigue words'), indicating a need for internal review, refinement, or a more robust selection/generation process for track names.

#### How to apply
-   Implement a review and refinement process for the 'tracknames' tool to improve the quality and variety of generated suggestions.

---
Consolidated from: `suno-prompt-engineering-principles.md`, `suno-system-notes.md`
