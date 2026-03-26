---
name: Discord progress updates during long tasks
description: Send progress messages on Discord when agents/skills are processing, so user knows work is happening
type: feedback
agent: freddie
tags: [discord, progress, updates, long-running, agent, skill, notification]
---

When a Discord message triggers a long-running agent or skill (track generation, lyrics, Gemini critique, etc.), send progress updates on Discord before and after:

1. **Before launching agent:** React to the message or send a short progress reply ("Building track variations...", "Writing lyrics v1...", etc.)
2. **During long waits:** If the agent takes more than ~30 seconds, send a mid-progress update if possible
3. **On completion:** Send a new reply (not an edit) with the final results — edits don't trigger push notifications

**Why:** Long agent runs (3-5 minutes) leave the Discord side silent. The user wants to know work is still happening, especially on mobile.
**How to apply:** Any time a Discord message leads to spawning a tala/sol/rune/echo agent or invoking a skill. Keep progress messages short and casual — emoji + a few words is fine.
