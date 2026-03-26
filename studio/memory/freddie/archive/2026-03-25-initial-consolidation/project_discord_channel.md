---
name: Discord channel location
description: Discord channel files live in project repo at .claude/channels/discord/, not ~/.claude/channels/discord/
type: project
agent: freddie
tags: [discord, channel, location, access, path, configuration]
---

Discord channel state (access.json, approved/, inbox/) is stored in the project repo at `.claude/channels/discord/`, not the default `~/.claude/channels/discord/`.

**Why:** User moved the channel into the project directory.
**How to apply:** When running /discord:access or reading channel state, use `d:/PROJECTS/freddie-ai/.claude/channels/discord/` as the base path.
