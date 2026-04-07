---
name: Wick's Raw Data Management System
description: Details Wick's comprehensive system for capturing, processing, and indexing raw interaction data from various sources.
type: knowledge
agent: wick
tags: [data-management, raw-data, capture, processing, indexing, cli, claude, jsonl, fts, reflect]
---

## Knowledge
Wick has a robust and evolving system for capturing, processing, and managing raw interaction data, designed to ensure comprehensive logging and structured access.

**Key capabilities include:**
*   **Automated Raw Data Capture (OT-0006):** Wick automatically captures raw data using `writeRaw()` and `insertRawFile()` within `src/tools/reflect.ts` when the `--raw-ref` option is absent. A `buildTaskSection()` helper assists in formatting the Task section of captured data. The system verifies that raw files are created, indexed in `brain.db`, and reflect JSON output is clean.
*   **Claude Code JSONL Integration (OT-0007):** The `write-raw.ts` component has been rewritten to read Claude Code JSONL files. This involved updating `raw.ts` types to include `CapturedMessage` and `RawFileInput` with a `messages[]` array. The `/bye` and `/save-raw` skills have been updated to leverage this. The tool can auto-detect the most recent JSONL from `~/.claude/projects/{slug}/`, with slug generation converting `lowercase drive + :\` to `--`.
*   **Advanced JSONL Insertion and Chunking (OT-0011):** The `insertRawJsonl()` function, added to `src/libs/brain/raw.ts`, handles exchange-level chunking for both native transcript and PostToolUse JSONL formats. `raw_tool_calls` are populated per `tool_use` event. `findRawFiles()` returns a format tag, and `syncRaw()` dispatches processing by format, clearing `raw_tool_calls` on a full rebuild.
*   **Enhanced Search Capabilities (OT-0012):** A `raw_tools_fts` FTS5 table with associated triggers has been added in `fts.ts` to enable full-text search. The `searchRawByTool()` function in `queries.ts` allows for efficient searching of raw data based on tool usage. The `/bye` and `/save-raw` skills were updated to support these new search features.

## Why
This comprehensive raw data management system ensures that all interactions are meticulously recorded and structured. This is critical for debugging, auditing, providing historical context, and enabling advanced analysis of Wick's operational patterns and tool usage. The ability to process various JSONL formats and perform tool-specific searches significantly enhances data utility and accessibility.

## How to apply
Rely on Wick's automatic raw data capture for comprehensive logging of its operations. Utilize the updated `/bye` and `/save-raw` skills for managing raw data. For specific inquiries into tool usage or interaction patterns, leverage the `searchRawByTool()` functionality. The system's support for both native transcript and PostToolUse JSONL formats allows for flexible integration with different data sources.

## Consolidated from
- `raw-log-20260401-081236-note.md`
- `raw-log-20260401-094416-note.md`
- `raw-log-20260401-115507-note.md`
- `raw-log-20260401-115742-note.md`
