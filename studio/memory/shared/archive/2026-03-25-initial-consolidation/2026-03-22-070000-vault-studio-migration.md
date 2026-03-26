---
id: mem-shared-20260322-070000-01
title: Vault Studio Migration
created: 2026-03-22 07:00:00
tags: [obsidian, vault, migration, file-paths, infrastructure]
type: decision
agent: shared
connected: []
---

All content folders moved into `vault/studio/` as part of Obsidian vault integration. The old root-level `artifacts/`, `references/`, and `memory/` folders no longer exist.

**New paths:**
- Tracks: `vault/studio/tracks/`
- Lyrics: `vault/studio/lyrics/`
- References: `vault/studio/references/`
- Memory: `vault/studio/memory/`
- Personal notes: `vault/notes/` (gitignored)

**Why:** The repo now doubles as an Obsidian vault for mobile access (iPhone/iPad). The `vault/` folder syncs via Obsidian Sync or iCloud. `vault/notes/` holds personal PKM (daily notes, inbox, curated resources) and is gitignored. The `studio/` subfolder name was chosen to fit the music production theme.

**How to apply:** Always use `vault/studio/` paths when reading or writing tracks, lyrics, references, or memory. Never reference the old `artifacts/`, `references/`, or `memory/` root paths. The discover script and gemini-proxy already resolve to the new paths.
