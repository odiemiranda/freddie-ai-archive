---
name: Ryan's Data Management and Persistence Contributions
description: Details Ryan's work on staged content persistence, Brain DB schema enhancements, knowledge directory hashing for data integrity, robust table formatting, and advanced soul change semantics with before-text tracking.
type: knowledge
agent: ryan
tags: [data-management, persistence, database, schema, sqlite, fts, hashing, data-integrity, migration, testing, staged-content, table-formatting, soul-changes, before-text, prompt-versioning, asl-0002, asl-0004, asl-0016, aar-0008, asl-0021]
---

## Knowledge
Ryan has significantly enhanced the system's data management and persistence capabilities, focusing on structured data storage, integrity, and detailed change tracking.

**Staged Content Persistence (AAR-0008):**
A critical bug in staged-content persistence was fixed. This involved adding a `title` field to the `StagedProposal` interface and updating numerous files across the system, including `staged.ts`, `autonomous-distill.ts`, `brain/schema.ts`, `brain/index.ts` (with `ALTER` and `FTS5` drop/recreate), `brain/fts.ts`, `brain/sync.ts`, `brain/queries.ts`, and `morning.ts`. A new `staged-roundtrip.test.ts` (1 pass) and a migration script (`migrate-staged-pre-fix.ts`) were created to support this. The `title` column was confirmed in `brain.db`, and `staged_fts` was updated to include the new field. A migration dry-run showed 77 affected files.

**Brain DB Schema for Tasks & Agent Lifecycle (ASL-0002):**
The `brain.db` schema was extended with new `tasks` and `agent_lifecycle` tables. This involved adding 48 lines to `index.ts` and 36 lines to `schema.ts`. A new test file, `__tests__/schema-asl-0002.test.ts`, was created with 6 passing tests, confirming the schema additions. These new tables were strategically placed above the `staged_fts` rebuild block, ensuring no scope drift.

**Knowledge Directory Hashing (ASL-0004):**
A library was implemented for hashing knowledge directories, ensuring their integrity and state tracking. This involved computing sentinel hex values for `EMPTY`, `MISSING`, and `COMPOSITE` states. The Drizzle ORM was updated to correctly handle `onConflictDoUpdate` with single-column targets. The library (473 LOC) and its comprehensive test suite (505 LOC) passed all 24 tests. A spec deviation regarding an assumption about an empty shared knowledge vault was resolved by dynamically computing the expected composite hash within the tests.

**Staged Content Table Formatting (ASL-0016):**
Ryan implemented `escapeTableCell` and `unescapeTableCell` functions in `staged.ts` to ensure robust handling of table data, particularly for display and parsing. Parsing functions (`parseGateResultsTable`, `parseJudgeResultsTable`) were moved from `review-staged.ts` to `staged.ts` for better centralization, and formatting functions were updated to utilize the new escaping mechanisms. This was validated with 53 passing regression tests.

**Soul Change Semantics and Before-Text Tracking (ASL-0021):**
The `applySoulChange` semantics were implemented to track the state of knowledge before modifications. This involved adding a `replaceInSoul` helper, extending `ParsedProposal.beforeText` and `StagedProposal.beforeText` fields, and updating `parseDistillOutput` to extract `BEFORE_TEXT`. The `staged` module's write/read operations were plumbed to handle this new field, `toParsedProposal` copies `beforeText`, and `auditProposal` now includes `beforeText` for comprehensive auditing. The distill prompt contract was updated to include `BEFORE_TEXT`, and `DISTILL_PROMPT_VERSION` was bumped to `v3-2026-04-09`. Related `rejection-feedback` tests were updated.

## Why
These developments ensure robust data persistence, accurate schema management for core operational data (tasks, agent lifecycle), reliable verification of knowledge directory states, and secure, consistent formatting of staged content. The introduction of `beforeText` for soul changes is critical for maintaining data integrity, enabling traceability of agent actions, and supporting consistent system behavior across various operations by providing a clear audit trail of modifications.

## How to apply
Rely on the enhanced staged content persistence for reliable storage and retrieval of in-progress proposals and intermediate data. Utilize the `tasks` and `agent_lifecycle` tables for monitoring, managing, and auditing agent operations and their associated tasks. The knowledge directory hashing mechanism provides a robust way to verify the state and integrity of the shared knowledge base. The table formatting functions ensure that structured data within staged content is correctly escaped and parsed. When reviewing or auditing soul changes, leverage the `beforeText` field to understand the context of modifications, and ensure that any custom distill prompts adhere to the `BEFORE_TEXT` contract for `DISTILL_PROMPT_VERSION v3`.

## Consolidated from
- `raw-log-20260407-152918-note.md`
- `raw-log-20260407-205610-note.md`
- `raw-log-20260407-213703-note.md`
- `raw-log-20260407-214308-note.md`
- `raw-log-20260408-093906-note.md`
- `raw-log-20260409-031937-note.md`
