---
name: Studio Infrastructure
description: Physical file organization and indexing tools for the music studio.
type: knowledge
agent: shared
tags: [infrastructure, migration, paths, indexing, studio-tool]
---

# Studio Infrastructure

## Vault Studio Migration
The project has transitioned to an Obsidian-compatible structure. All production assets live under `vault/studio/`.

### Current Paths
- **Tracks:** `vault/studio/tracks/`
- **Lyrics:** `vault/studio/lyrics/`
- **References:** `vault/studio/references/`
- **Memory:** `vault/studio/memory/`
- **Personal/Tasks:** `vault/notes/` (gitignored)

**How to apply:** Never use root-level `/artifacts`, `/references`, or `/memory`. All agents and scripts (e.g., `discover.ts`) must resolve to the `vault/studio/` hierarchy.

## Studio Index Tool (`tools/studio.ts`)
To prevent slow file-by-file traversal, a search index tool is being implemented to provide instant lookups for Tala and Rune.

### Index Features
- **Schema:** Tracks file type, title, platform, genres, moods, instruments, BPM, key, and body keywords.
- **Search Mode:** Keyword matches against tags and extracted signals.
- **Save Flow:** Agents should call `bun tools/studio.ts --save {path}` immediately after writing new track or lyrics files.
- **Output:** Generates `vault/studio/tracks-index.json` and `vault/studio/lyrics-index.json`.

**Consolidated from:**
- `2026-03-22-070000-vault-studio-migration.md`
- `2026-03-23-110000-studio-index-tool.md`
