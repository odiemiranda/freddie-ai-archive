---
name: Wick's Prompt Data and Quality Tools
description: Covers Wick's capabilities for managing core prompt-data libraries and analyzing generated text for quality, such as repetition.
type: knowledge
agent: wick
tags: [prompt-data, data-management, library-initialization, file-operations, verification, core-resources, lyrics-analysis, repetition-analyzer, n-gram, levenshtein, anaphora]
---

## Knowledge
Wick possesses comprehensive capabilities for managing its internal prompt-data libraries and ensuring the quality of generated text, particularly lyrics.

**Prompt Data Management and Integrity:**
Wick manages its internal prompt-data libraries, which are critical for consistent and reliable prompt generation. This includes the initialization, updating, and routine verification of key structured data files. Wick ensures the existence and correct content of files such as `homographs.ts`, `artist-blocklist.ts`, and `cliches.ts`, located in `src/libs/prompt-data/`. The initialization process involves creating new structured data files (e.g., `artist-blocklist.ts` with 105 entries across 14 genres) and merging content from multiple sources (e.g., combining 'ubiquitous' and 'ai-slop' data into `cliches.ts`, resulting in 182 entries, with 4 tagged as both). Routine verification processes confirm the integrity, compilation cleanliness, and full existence of these core resources.

**Lyrics Repetition Analyzer:**
Wick has developed a `repetition-analyzer` (OT-0024) located at `src/libs/lyrics-analyzer/tools/repetition-analyzer.ts`. This tool detects exact n-gram repeats (2-6 words), word-level Levenshtein near-matches (using a >0.7 threshold), and performs anaphora detection. It also provides per-section ratio flagging to highlight areas of high repetition. All 7 acceptance criteria tests for this analyzer have passed.

## Why
The consistent integrity and availability of pre-defined prompt-data files are fundamental dependencies for Wick's prompt generation system, ensuring prompts are generated accurately and avoiding issues like homograph confusion, undesirable artist mentions, or cliché overuse. The repetition analyzer enhances the quality of generated lyrics by identifying and flagging repetitive patterns, leading to more diverse and engaging outputs.

## How to apply
Leverage Wick's established capabilities for prompt data maintenance. When introducing new prompt-data requirements or updating existing ones, utilize Wick's initialization and merging functionalities. Rely on the routine verification processes to confirm the ongoing health and accuracy of these critical resources. Utilize the `repetition-analyzer` to analyze generated lyrics for unwanted repetition, near-matches, or anaphora, and use the per-section ratio flagging to guide refinements for higher quality outputs.

## Consolidated from
- `20260331-063543-wick-s-prompt-generation-relies-on-the-c.md`
- `20260331-064541-wick-possesses-the-capability-to-initial.md`
- `raw-log-20260403-190846-note.md`
