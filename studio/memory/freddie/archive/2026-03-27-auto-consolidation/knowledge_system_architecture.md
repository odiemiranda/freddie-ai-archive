---
name: Workspace Architecture and Environment
description: OS-specific handling, timezone logic, and the migration to SDK-based AI integration
type: knowledge
agent: freddie
tags: [environment, windows, timezone, backup, sdk, gemini, minimax]
---

## Environment & Schedule
- **OS:** Windows (Home: `C:\Users\odiem`). Shell: `nushell`.
- **Timezone:** Mon-Fri (8pm-4am Manila) = **PST/PDT**. Weekends = Manila. Always check system time against the PST window for logs.

## Infrastructure Strategy
- **Backup:** `tools/backup.ts` performs 10-min incremental syncs of the `vault/` (Obsidian) symlink to the `archive/` submodule.
- **Vault Sync:** `vault/` is an iCloud symlink; never `git add` its contents directly.
- **Gemini SDK (complete):** `libs/gemini.ts` wraps `@google/genai`. All callers migrated — `consolidate-memory.ts`, `musiccard.ts`, Echo agent. `gemini-proxy.ts` and `execa` removed. Gemini CLI no longer required.
- **Usage Tracking (complete):** `libs/ai-usage.ts` auto-logs every Gemini call to `vault/studio/usage/YYYY-MM.jsonl`. View with `bun src/tools/usage.ts`. Pricing table covers Gemini, MiniMax, Claude.
- **MiniMax (pending):** Direct API integration for track generation; no audio input support (use Gemini for audio analysis).
- **Auto-memory scope:** Auto-dream (`.claude-personal/`) writes user/feedback only. Production rules live in vault knowledge. Prevents knowledge pollution.

**Why:** Gemini CLI was unstable on Windows. Direct SDK eliminates child process overhead, temp files, and session management. Usage tracking enables cost monitoring as paid APIs (MiniMax) come online.

**How to apply:** Use `npm run backup:status` to check sync health. Use `bun src/tools/usage.ts` to monitor API costs. Trust vault knowledge over auto-memory for production rules.

*Consolidated from: knowledge_workspace_infrastructure.md, 2026-03-25-sdk-architecture-research.md, 2026-03-25-session-summary.md*
