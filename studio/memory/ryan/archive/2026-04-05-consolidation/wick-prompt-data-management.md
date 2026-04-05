---
name: Wick's Prompt Data Management and Integrity
description: Details Wick's capabilities in initializing, updating, and verifying critical prompt-data libraries for consistent prompt generation.
type: knowledge
agent: wick
tags: [prompt-data, data-management, library-initialization, file-operations, verification, core-resources]
---

## Knowledge
Wick possesses comprehensive capabilities for managing its internal prompt-data libraries, which are critical for consistent and reliable prompt generation. This management includes the initialization, updating, and routine verification of key structured data files.

Specifically, Wick ensures the existence and correct content of files such as `homographs.ts`, `artist-blocklist.ts`, and `cliches.ts`, located in `src/libs/prompt-data/`.

The initialization process involves creating new structured data files (e.g., `artist-blocklist.ts` with 105 entries across 14 genres) and merging content from multiple sources (e.g., combining 'ubiquitous' and 'ai-slop' data into `cliches.ts`, resulting in 182 entries, with 4 tagged as both).

Routine verification processes are in place and effective, confirming the integrity, compilation cleanliness, and full existence of these core resources.

## Why
The consistent integrity and availability of these pre-defined prompt-data files are fundamental dependencies for Wick's prompt generation system. Effective data management ensures that prompts are generated accurately, avoiding issues like homograph confusion, undesirable artist mentions, or cliché overuse.

## How to apply
Leverage Wick's established capabilities for prompt data maintenance. When introducing new prompt-data requirements or updating existing ones, utilize Wick's initialization and merging functionalities. Rely on the routine verification processes to confirm the ongoing health and accuracy of these critical resources, ensuring high-quality and reliable prompt outputs.

## Consolidated from
- `20260331-063543-wick-s-prompt-generation-relies-on-the-c.md`
- `20260331-064541-wick-possesses-the-capability-to-initial.md`
