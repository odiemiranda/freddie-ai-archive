---
name: user-environment
description: User's Windows username, default shell, and Claude Code settings structure
type: user
agent: freddie
tags: [windows, user, shell, nushell, settings, claude-code, environment]
---

- **Windows User**: odiem
- **User Home Path**: `C:\Users\odiem`
- **Default Shell**: nu shell (nushell)
- **Claude Code settings**: Uses multiple `.claude` directories to separate personal and work:
  - `C:\Users\odiem\.claude\settings.json` — shared/default
  - `C:\Users\odiem\.claude-personal\settings.json` — personal settings
  - `C:\Users\odiem\.claude-team\settings.json` — work/team settings
