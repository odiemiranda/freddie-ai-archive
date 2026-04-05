---
name: Communication and Interaction Standards
description: Guidelines for agent communication style, progress updates, output formatting, and naming conventions to ensure clarity, natural conversation flow, and user preference alignment.
type: knowledge
agent: freddie
tags: [discord, communication, progress, configuration, user_preference, naming_convention, persona, output_formatting, agent_voice, clarity, music_card]
---

## General Communication Style
- **No @mentions:** Do not use @ mentions (e.g., `<@user_id>`) when replying. Respond directly to keep conversation flow natural.
- **Progress Updates:** For long-running tasks (track generation, lyrics, critique), provide visibility:
    1.  **Initial:** React to the trigger message or send a short reply (e.g., "Building tracks...").
    2.  **Mid-task:** If processing exceeds 30 seconds, send a brief status update.
    3.  **Completion:** Always send a **new reply**. Do not edit previous progress messages.
- **Agent Voice Attribution:** Implement a consistent `[Agent Name]:` prefix for all agent commentary. This ensures clear attribution, improved readability, and maintains clean output in multi-agent or tool-heavy interactions.

## Naming Conventions
- **Impactful Naming:** When naming internal or conceptual entities, prioritize names that convey a specific persona or impact, often favoring surnames or established archetypes. This aligns with user preference for clarity and thematic resonance.
- **Music Card Outputs:** All generated music card outputs must follow the `YYYY-MM-DD-HHmmss` naming convention for consistency and ease of organization.

## Discord Configuration
- **Channel Path:** Discord state (access, approved, inbox) is stored at `d:/PROJECTS/freddie-ai/.claude/channels/discord/`.
- **Note:** Do not use the default `~/.claude/channels/discord/` path.

**Why:** User prefers natural conversation without pings. Edits are missed by mobile notifications. In-project state ensures portability across machines. Clear attribution and impactful naming improve communication, readability, and align with user preferences for organized and resonant interactions, including standardized output naming for better organization.

**How to apply:**
- Use the Discord reply tool without tagging users.
- Send incremental updates for any task using Tala, Sol, Rune, or Echo.
- When reflecting or generating agent commentary, ensure it is prefixed with `[Agent Name]:`.
- When naming internal agents or conceptual entities, consider surnames or archetypes that convey impact.
- Ensure all generated music card outputs adhere to the `YYYY-MM-DD-HHmmss` naming convention.
- Check `d:/PROJECTS/freddie-ai/.claude/channels/discord/` for Discord state.

*Consolidated from: knowledge_discord_interaction.md, 20260330-123330-when-naming-internal-or-conceptual-entit-2.md, 20260330-211929-implementing-a-consistent-agent-name-pre-2.md, 20260401-080142-all-generated-music-card-outputs-must-fo-3.md*
