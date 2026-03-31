---
name: Discord Interaction and Workflow
description: Communication style, progress notifications, and state management for Discord integration
type: knowledge
agent: freddie
tags: [discord, communication, progress, configuration, path]
---

## Communication Style
- **No @mentions:** Do not use @ mentions (e.g., `<@user_id>`) when replying. Respond directly to keep conversation flow natural.
- **Progress Updates:** For long-running tasks (track generation, lyrics, critique), provide visibility:
    1. **Initial:** React to the trigger message or send a short reply (e.g., "Building tracks...").
    2. **Mid-task:** If processing exceeds 30 seconds, send a brief status update.
    3. **Completion:** Always send a **new reply**. Do not edit previous progress messages.

## Configuration and State
- **Channel Path:** Discord state (access, approved, inbox) is stored at `d:/PROJECTS/freddie-ai/.claude/channels/discord/`.
- **Note:** Do not use the default `~/.claude/channels/discord/` path.

**Why:** User prefers natural conversation without pings. Edits are missed by mobile notifications. In-project state ensures portability across machines.

**How to apply:** Check `d:/PROJECTS/freddie-ai/.claude/channels/discord/` for state. Use the Discord reply tool without tagging users. Send incremental updates for any task using Tala, Sol, Rune, or Echo.

*Consolidated from: knowledge_discord_workflow.md (v1), 2026-03-25-session-summary.md*
