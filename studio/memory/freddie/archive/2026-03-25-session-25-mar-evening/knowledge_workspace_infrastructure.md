---
name: Workspace Infrastructure and Sync
description: Environment configuration, timezone logic, and vault synchronization strategy
type: knowledge
agent: freddie
tags: [environment, timezone, vault, icloud, backup, windows, sync]
---

## Environment & Schedule
- **User:** `odiem` on Windows (Home: `C:\Users\odiem`).
- **Shell:** `nu shell` (nushell).
- **Timezone:** During Mon-Fri work hours (8pm-4am Manila), operate in **PST/PDT**. "Today" at 10pm Manila on Monday refers to Monday PST. Default to Manila time on weekends.

## Vault & Backup System
- **Obsidian/iCloud:** `vault/` is a symlink to `D:\iCloudDrive\Obsidian\vault\`. It is **not tracked by git** to avoid symlink/sync conflicts.
- **Backup Pipeline:** `tools/backup.ts` performs incremental syncs to the `archive/` (freddie-ai-archive) submodule.
    - `npm run backup`: Manual sync.
    - `npm run backup:start`: Installs system scheduler (Windows Task Scheduler) for 10-min interval backups.
    - `out/backup-manifest.json`: Local file (gitignored) used to track changes.

**Why:** iCloud is the source of truth for mobile/desktop sync. Git submodule provides version history and redundancy without conflicting with iCloud's file handling.

**How to apply:** Never `git add` vault contents. Use `npm run backup:status` to verify sync health. Always check system time against the PST work window before creating daily logs.

*Consolidated from: user_environment.md, feedback_work_timezone.md, project_obsidian_vault.md, project_backup_system.md*
