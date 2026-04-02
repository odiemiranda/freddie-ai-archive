---
name: Wick's Obsidian MD Integration
description: Details Wick's capability to parse, serialize, and manage Obsidian Markdown files, including migration of existing tools.
type: knowledge
agent: wick
tags: [tool-integration, obsidian, markdown, parser, serializer, data-management, knowledge-base]
---

## Knowledge
Wick has developed a dedicated library `src/libs/obsidian-md/` (OT-0013) for handling Obsidian Markdown files. This library includes types, a parser, a serializer, and various operations. It has undergone thorough testing, with 48 operation tests passing and roundtrip verification on 5 vault files, confirming its reliability.

Subsequently, existing Obsidian-related tools have been successfully migrated to utilize this new `obsidian-md` library (OT-0014), replacing the deprecated `libs/sections.ts`. All CLI tools leveraging this integration have been verified, and roundtrip tests continue to pass, ensuring seamless functionality and data integrity.

## Why
This integration provides a robust and standardized way for Wick to interact with and manage knowledge stored in Obsidian vaults. It ensures accurate parsing, manipulation, and serialization of Obsidian Markdown, improving data integrity and facilitating advanced knowledge retrieval and processing from external knowledge bases.

## How to apply
Utilize Wick's `obsidian-md` library for any tasks involving reading, writing, or manipulating Obsidian Markdown files. This ensures compatibility with Obsidian's specific syntax and structure, enabling seamless integration with external knowledge bases and structured note-taking systems.

## Consolidated from
- `raw-log-20260401-165808-note.md`
- `raw-log-20260401-171811-note.md`
