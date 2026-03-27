---
name: Studio Infrastructure and Tools
description: Core infrastructure, file organization, indexing, and tool execution constraints for the music studio.
type: knowledge
agent: shared
tags: [architecture, infrastructure, file-paths, indexing, tools, obsidian, mcp, claude-code, penny, music-cards, tasks]
---

# Studio Infrastructure and Tools

## Vault Studio Migration & File Paths
The project has transitioned to an Obsidian-compatible structure, with `vault/studio/` as the root for all production assets.

**Why:** Ensures consistent file organization and prevents conflicts with personal notes.
**How to apply:** All agents and scripts must resolve to the `vault/studio/` hierarchy. Never use root-level `/artifacts`, `/references`, or `/memory`.
- **Tracks:** `vault/studio/tracks/`
- **Lyrics:** `vault/studio/lyrics/`
- **References:** `vault/studio/references/`
- **Memory:** `vault/studio/memory/`
- **Personal/Tasks:** `vault/notes/` (gitignored)

## Music Cards: Audio Reference System
Music Cards are playable Markdown files in Obsidian that link real-world audio samples to AI generation constraints.

**Why:** Provides a structured, playable reference system for AI music generation.
**How to apply:** Use these cards to inform BPM targets, instrument phrasing, and style translations for AI models.
- **Location:** `.md` cards in `vault/studio/references/shared/music-cards/`; `.mp3` clips in `audio/` subfolder (gitignored).
- **Card Types:** `background-mix`, `music-video`, `single-track`, `live-performance`.
- **Content:** Embedded audio, Mix Profile (BPM, key, groove, palette), Suno/MiniMax Translation (style voice, bracket tags, exclude lists), Vocal Map (emotional arc, phrasing, space/silence).
- **Workflow Integration:** Tala uses matched cards for BPM/phrasing. Rune uses phrasing maps for lyrical rhythm. Extraction involves `yt-dlp` + `ffmpeg` and Gemini analysis.

## Persistent Task Documents
Tasks that span multiple sessions are managed in dedicated source-of-truth documents.

**Why:** Ensures persistent task tracking and clear history across sessions.
**How to apply:** Create task docs in the specified format for long-running tasks.
- **Location:** `vault/notes/tasks/`.
- **Format:** `OT-####-description.md` containing status, description, sub-tasks.
- **Linking:** Daily notes reference tasks via Wikilinks (e.g., `[[notes/tasks/OT-123|Task Name]]`).
- **Penny Integration:** Penny will manage these via `penny_task` for automatic creation, status updates, and Obsidian Graph View visualization.

## Studio Index Tool (`tools/studio.ts`)
A search index tool is being implemented to provide instant lookups for Tala and Rune, preventing slow file-by-file traversal.

**Why:** Improves performance and efficiency for agents needing to access studio assets.
**How to apply:** Agents should call `bun src/tools/studio.ts --save {path}` immediately after writing new track or lyrics files.
- **Schema:** Tracks file type, title, platform, genres, moods, instruments, BPM, key, and body keywords.
- **Search Mode:** Keyword matches against tags and extracted signals.
- **Output:** Generates `vault/studio/tracks-index.json` and `vault/studio/lyrics-index.json`.

## Claude Code Tool Execution
Tools are dual-mode: MCP (`mcp__freddie-tools__*`) in the main conversation, CLI (`bun src/tools/...`) when running as subagents.

**Why:** Subagent types (tala, rune, echo, sol, iris, veo) only have access to Read, Write, Glob, Grep, and Bash — not MCP tools. Only the main conversation and `general-purpose` agents get MCP access.
**How to apply:** Agent files use CLI syntax for workflow examples. MCP tool names are documented in tool tables for reference only.

**Consolidated from:**
- `studio-architecture.md`
