---
title: "Eidetic Memory Task 7 — Rules + Docs Update"
type: task
status: pending
parent: 2026-04-01-000700-eidetic-memory-raw-layer
task: 7
depends: [1, 2, 3, 4, 5]
created: 2026-04-01
author: mccall
---

# Task 7 — Rules + Docs Update

## Objective

Update project rules and documentation to reflect the raw context layer. Agents and workflows need to know about `memory/raw/`, the search modes, and the lifecycle rules.

## What to Build

### 1. Update `.claude/rules/memory-boundaries.md`

Add a new section documenting the raw layer:

```markdown
## Raw Context Layer (`memory/raw/`)

| Property | Value |
|----------|-------|
| Location | `vault/studio/memory/raw/` |
| Naming | `{unix-timestamp}.md` |
| Lifecycle | Immutable, permanent, append-only |
| Ownership | Shared — all agents write, all agents can search |
| Search | Excluded from default search. Use `--recall` or `--search-raw` explicitly. |

**Rules:**
- Raw files are NEVER edited, merged, archived, or deleted
- `consolidate-memory.ts` does NOT process raw files
- `syncMemories()` does NOT index raw files (they have their own sync path)
- Write raw BEFORE reflect — reflection receives the raw timestamp
- Every workflow that produces meaningful context gets a raw file
```

Add raw to the comparison table:

```markdown
| System | Scope | Examples |
|--------|-------|---------|
| Auto-memory | User preferences, feedback, communication style | "user prefers terse responses" |
| Vault knowledge | Production rules, agent techniques, platform mechanics | Suno Style formula |
| Raw context | Full session transcripts, complete workflow traces | Discover results, critic output, conversation |
```

### 2. Update `CLAUDE.md`

Add `raw_files`, `raw_chunks`, `raw_fts`, `vec_raw` to the brain.db tables list:

```markdown
| `raw_files` | Parent record per raw context file — ts, date, agents, skill, tags, summary |
| `raw_chunks` + `raw_fts` + `vec_raw` | Phase-level chunks of raw context with FTS5 + vector embeddings |
```

Add `raw_ref` column note to the `decisions` row.

Add to the Search modes section:

```markdown
- `--recall` — Cross-store search INCLUDING raw (conversational memory recall)
- `--search-raw` — Raw store ONLY search with optional `--skill` filter
```

Add to the memory folder structure:

```markdown
memory/
├── raw/              <- shared, append-only, never consolidated
├── shared/
├── tala/
├── rune/
└── ...
```

Add `src/libs/raw.ts` to the Shared Libraries table:

```markdown
| `raw.ts` | Raw context file writer — `writeRaw()` for eidetic memory |
```

Add `src/libs/brain/raw.ts` description to the brain/ module list.

### 3. Verify `consolidate-memory.ts` exclusion

Check that `src/tools/consolidate-memory.ts` does not process `memory/raw/`. The tool iterates agent folders looking for `inbox/` subdirectories. Since `raw/` has no `inbox/` subfolder, it should be naturally excluded. Verify this and add an explicit skip if needed:

```typescript
// In the agent iteration loop:
if (agent === "raw") continue;  // raw context is immutable, never consolidated
```

## Files to Modify

| File | Change |
|------|--------|
| `.claude/rules/memory-boundaries.md` | Add raw context layer section and rules |
| `CLAUDE.md` | Add raw tables, search modes, memory structure, libs |
| `src/tools/consolidate-memory.ts` | Verify raw exclusion, add explicit guard if needed |

## Acceptance Criteria

1. `memory-boundaries.md` documents raw layer lifecycle rules
2. `memory-boundaries.md` includes raw in the comparison table
3. `CLAUDE.md` brain.db section lists all four raw tables
4. `CLAUDE.md` search modes section includes `--recall` and `--search-raw`
5. `CLAUDE.md` memory folder structure shows `raw/`
6. `CLAUDE.md` shared libraries table includes `raw.ts`
7. `consolidate-memory.ts` explicitly skips `raw` agent (either naturally or via guard)

## Test Criteria

1. Read `memory-boundaries.md` and verify raw section is present and accurate.
2. Read `CLAUDE.md` and verify all brain.db table additions are present.
3. Run `bun src/tools/consolidate-memory.ts --list` and verify `raw` does not appear in the agent list (or appears with 0/0 counts and is noted as excluded).
