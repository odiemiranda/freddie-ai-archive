---
name: obsidian_vault_icloud_sync
description: Vault lives in iCloud Drive, symlinked into repo — untracked by git, synced to iPhone via Obsidian + iCloud, backed up to freddie-ai-archive submodule
type: project
agent: freddie
tags: [vault, obsidian, icloud, symlink, sync, iphone, git, backup, archive]
---

The `vault/` folder is no longer git-tracked. It lives at `D:\iCloudDrive\Obsidian\vault\` and is symlinked into the repo at `D:\PROJECTS\freddie-ai\vault`. Git ignores the symlink entirely.

- **Desktop Obsidian** points directly to the iCloud vault folder (not the symlink)
- **iPhone Obsidian** syncs via iCloud Drive
- **Claude Code** accesses vault content through the symlink — reads/writes work normally
- **Git** tracks only the toolkit (agents, skills, tools, references scripts) — not generated output
- **Backup** — `freddie-ai-archive` git submodule mirrors vault/studio/ every 10 minutes via scheduled task

**Why:** Symlinks + git on Windows caused repeated conflicts — `git checkout` would overwrite the symlink with a real directory. Untracking vault/ entirely eliminates the issue and lets iCloud be the single source of truth.

**How to apply:** Never run `git add` or `git checkout` on vault paths. All vault content (tracks, lyrics, memory, references) is read/written through the symlink but not version-controlled. Backup is handled by `tools/backup.ts`.
