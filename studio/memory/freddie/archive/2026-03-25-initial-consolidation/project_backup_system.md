---
name: backup_system
description: Vault backup system — freddie-ai-archive submodule with manifest-based incremental sync and cross-platform scheduler
type: project
agent: freddie
tags: [backup, archive, submodule, sync, scheduler, vault, incremental, manifest]
---

Vault content is backed up to a separate GitHub repo (`freddie-ai-archive`) via a git submodule at `archive/`. The backup script (`tools/backup.ts`) uses a local manifest file (`out/backup-manifest.json`, gitignored) to track file mtime/size for incremental sync — only copies changed files, detects deletions.

Commands:
- `npm run backup` — one-time sync
- `npm run backup:watch` — foreground sync every 10 min
- `npm run backup:start` — install system scheduler (Windows Task Scheduler / macOS launchd / Linux cron)
- `npm run backup:stop` — remove system scheduler
- `npm run backup:status` — check if scheduler is active

**Why:** iCloud is the primary sync for vault content, but having a git-backed redundant backup provides version history and protection against iCloud issues. Manifest is machine-local to avoid cross-machine timestamp conflicts.

**How to apply:** The scheduler runs automatically once installed. When modifying backup.ts, keep it cross-platform (no shell-specific commands). The manifest in `out/` should never be committed.
