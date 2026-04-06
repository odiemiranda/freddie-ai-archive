---
name: Freddie AI Lyrics Builder Design Principles
description: Consolidates design patterns and principles applied to the BGM and Vocal lyrics builders within the Freddie AI project.
type: knowledge
agent: mccall
tags: [freddie-ai, lyrics-generation, music-structure, design-pattern, system-design, modularity, data-normalization, vocal-lyrics, BGM-lyrics, configuration]
---

## Knowledge
The design of the Freddie AI lyrics builders incorporates several key principles for consistency, control, and adaptability:
*   **Derivation Mechanisms**: Specific tags (e.g., `PEAK_TAGS`) are effectively derived from higher-level structural priorities (e.g., `SECTION_PRIORITY`). This mechanism maintains consistency across generated outputs and reduces the need for manual configuration in complex generation systems.
*   **Energy Arc Concept**: For BGM lyrics, the concept of an 'energy arc' with predefined shapes (e.g., 4 distinct shapes) is a valuable structural element. It provides high-level control over the dynamic flow and intensity of the lyrical output, guiding the emotional and structural progression of the music.
*   **Modular Vocal Builder Design**: The vocal lyrics builder is designed with modularity and flexibility in mind, featuring:
    *   A dedicated section parser.
    *   Voice header management.
    *   Configurable output formatting via two distinct wrap modes (a header-only default and a full wrap option).
    *   Data normalization using `SECTION_TAG_ALIASES` for consistent processing of Rune tags. This modular approach significantly enhances the tool's utility and adaptability for various output requirements.

## Why
These design principles are crucial for building robust, controllable, and adaptable lyrics generation systems. Derivation mechanisms ensure internal consistency, energy arcs provide expressive control over musical dynamics, and modular designs with data normalization enhance the reusability, maintainability, and flexibility of the builders, allowing them to cater to diverse creative and technical requirements.

## How to apply
When developing or extending lyrics generation features, prioritize establishing clear derivation mechanisms for dependent parameters to ensure consistency. Consider incorporating high-level structural concepts like 'energy arcs' to provide intuitive control over the dynamic flow of generated content. Always favor modular designs for builders, separating concerns like parsing, formatting, and data normalization, to maximize adaptability and ease of maintenance. Leverage tag aliases for data normalization to ensure consistent processing across different parts of the system.

## Consolidated from
- `20260331-145249-establishing-a-derivation-mechanism-wher-2.md`
- `20260331-145249-the-concept-of-an-energy-arc-with-predef-1.md`
- `20260331-165434-a-modular-and-flexible-design-for-vocal--3.md`
