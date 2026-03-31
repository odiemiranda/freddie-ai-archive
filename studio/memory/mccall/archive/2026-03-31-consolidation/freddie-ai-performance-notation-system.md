---
name: Freddie AI Performance Notation System
description: Details the design, decisions, and implementation aspects of the Freddie AI performance notation system, including its mechanical and creative components.
type: knowledge
agent: mccall
tags: [freddie-ai, performance-notation, music-notation, rules-engine, LLM-selection, Gemini-2.5-Flash, system-design, tool-development, design-decisions, creative-generation, mechanical-generation, Suno-notation]
---

## Knowledge
The Freddie AI performance notation system is designed to provide both precise, mechanical control and nuanced, creative generation capabilities for musical notation.

**Core Components & Design:**
*   **Hybrid Approach**: The system ships with both creative and mechanical modes, with the creative mode set as the default, and the mechanical mode accessible via a `--mechanical` flag.
*   **Mechanical Rules Engine**: A structured mechanical rules engine, comprising 7 distinct rules, effectively controls complex performance notation. Parameters like `BPM_NOTATION_DENSITY` are wired into these rules to manage output density and other musical characteristics with precision.
*   **Creative Notation with Gemini 2.5 Flash**: Gemini 2.5 Flash has been selected for creative notation generation due to its superior capability for nuanced and imaginative outputs, outperforming other models like MiniMax/Grok in this domain.
*   **Design Brief & Taxonomy**: The design brief cataloged the full Suno notation taxonomy (6 confirmed, 2 unverified) and defined style-driven rules for mapping musical parameters (BPM, mood, energy, genre, vocal-style) to notation type and density.
*   **Tool Shape**: The tool is specified with a CLI interface, clear input/output types, and a defined position within the overall pipeline.

**Key Decisions & Deferred Items:**
*   **LLM Selection**: Gemini 2.5 Flash is the preferred choice for creative mode due to its zero cost, proven lyrical understanding, and high consistency.
*   **Phonetics Integration**: Integration of phonetics is deferred to a post-launch phase.
*   **Density Ceilings**: Initial density ceilings are set conservatively.
*   **Strip Flag**: The `--strip` flag functionality is confirmed.
*   **Pipeline Order**: Notation generation is confirmed to occur before the tri-critic analysis phase.
*   **Open Questions**: Ongoing questions include the optimal system prompt location, handling of non-English inputs, and whether the builder should be opt-in or opt-out.

## Why
This comprehensive approach ensures that the performance notation system can cater to a wide range of musical generation needs, from strictly controlled mechanical outputs to highly creative and expressive ones. The structured design facilitates maintainability and extensibility, while clear decision-making ensures alignment with project goals and efficient resource allocation.

## How to apply
When generating performance notation, utilize the default creative mode for imaginative outputs, or activate the mechanical mode with the `--mechanical` flag for precise, rule-based control. Leverage Gemini 2.5 Flash for any creative text generation tasks, especially those requiring nuanced musical understanding. Refer to the defined style-driven rules and Suno taxonomy for guidance on notation parameters. Be aware of deferred features like phonetics integration and conservative density ceilings in current implementations.

## Consolidated from
- `20260331-165434-complex-performance-notation-can-be-effe-2.md`
- `20260331-165434-gemini-2-5-flash-demonstrates-superior-c-1.md`
- `raw-log-20260331-151817-note.md`
- `raw-log-20260331-152712-decision.md`
- `raw-log-20260331-153406-note.md` (superseded task doc, but content on rules/integration is relevant to design)
- `raw-log-20260331-154745-note.md` (for context on task doc unification)
