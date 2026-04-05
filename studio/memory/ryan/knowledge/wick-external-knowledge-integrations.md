---
name: Wick's External Knowledge Integrations
description: Details Wick's integrations with external knowledge systems like NotebookLM and Obsidian Markdown for enhanced data access and management.
type: knowledge
agent: wick
tags: [tool-integration, notebooklm, cli, data-retrieval, external-tools, search, list, obsidian, markdown, parser, serializer, data-management, knowledge-base]
---

## Knowledge
Wick extends its capabilities through robust integrations with external knowledge management systems.

**NotebookLM CLI Wrapper Integration:**
Wick has completed the integration of a CLI wrapper for NotebookLM (identified as OT-0005). This integration includes 8 files located in `src/tools/notebooklm/` and provides functionalities for listing and searching within NotebookLM. Both `list` and `search` operations have been verified as passing.

**Obsidian MD Integration:**
Wick has developed a dedicated library `src/libs/obsidian-md/` (OT-0013) for handling Obsidian Markdown files. This library includes types, a parser, a serializer, and various operations. It has undergone thorough testing, with 48 operation tests passing and roundtrip verification on 5 vault files. Subsequently, existing Obsidian-related tools have been successfully migrated to utilize this new `obsidian-md` library (OT-0014), replacing the deprecated `libs/sections.ts`. All CLI tools leveraging this integration have been verified, and roundtrip tests continue to pass, ensuring seamless functionality and data integrity.

## Why
These integrations extend Wick's capabilities by allowing it to interact with external knowledge resources directly via the command line (NotebookLM) and to robustly manage knowledge stored in Obsidian vaults. This facilitates efficient data retrieval, exploration, and accurate parsing/manipulation of external knowledge, which is crucial for informing prompt generation and other knowledge-intensive tasks.

## How to apply
Utilize the NotebookLM CLI wrapper within Wick's environment to programmatically list and search content within NotebookLM, gathering context or retrieving specific information. Leverage Wick's `obsidian-md` library for any tasks involving reading, writing, or manipulating Obsidian Markdown files, ensuring compatibility with Obsidian's specific syntax and structure for seamless integration with external knowledge bases.

## Consolidated from
- `wick-notebooklm-cli-wrapper.md`
- `wick-obsidian-md-integration.md`
