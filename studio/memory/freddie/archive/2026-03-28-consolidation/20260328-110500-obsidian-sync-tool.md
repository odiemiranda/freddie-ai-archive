---
title: "Obsidian Sync Tool — Build Learnings"
agent: freddie
scope: self
created: 2026-03-28
tags: [obsidian, sync, daemon, windows, powershell, tool-development]
---

## Obsidian Sync CLI Wrapper (obsidian.ts)

Built dual-mode tool wrapping obsidian-headless for vault sync.

**Key learnings:**
1. Node.js `spawn({ detached: true })` on Windows always pops a console window — `windowsHide: true` is ignored for console apps. This is a known, unfixed Node.js bug.
2. Only reliable fix: PowerShell `Start-Process -WindowStyle Hidden` — manages window state at OS level.
3. pm2 also pops terminal windows on Windows (node.exe title) — same underlying spawn issue. Removed pm2, went with direct PowerShell approach.
4. `ob sync --continuous` is a long-running watcher, not interval-based — daemon just spawns it and manages PID file.
5. Vault path resolved dynamically from `OBSIDIAN_VAULT_ID` env var matched against `ob sync-list-local` output — portable across machines.
6. PowerShell `RedirectStandardOutput` and `RedirectStandardError` cannot point to same file — must use separate log files.
7. PowerShell `-ArgumentList` needs comma-separated quoted items, not a single string — `'sync','--path','vault'` not `'sync --path vault'`.
