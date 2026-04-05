---
title: "Eidetic Memory: Raw Context Layer"
type: tech-approach
status: approved
created: 2026-04-01
tags: [memory, brain-db, architecture]
---

# Eidetic Memory: Raw Context Layer

## Problem

The reflection layer (`reflect.ts`) summarizes creative sessions into structured observations. This is useful for pattern recognition and knowledge consolidation, but it loses nuance — the exact sequence of decisions, the specific discover results that influenced a choice, the critic feedback that caused a pivot. When a user asks "what did we talk about yesterday regarding authentication?", reflected summaries may not have enough detail to answer accurately.

## Solution

Add a shared `memory/raw/` folder that stores full workflow transcripts as permanent, append-only records. Each file captures the complete context: user input, discover results, distill output, generated variations, critic feedback, final output, and conversation. Indexed in brain.db for semantic and keyword search, but excluded from default search to avoid polluting generation queries.

## Design

### Storage

```
memory/
├── raw/              ← shared, append-only, never consolidated
│   ├── 1743408000.md
│   ├── 1743494400.md
│   └── ...
├── shared/
├── tala/
├── rune/
└── ...
```

**File naming:** `{unix-timestamp}.md` — no collisions, sortable, cheap to generate.

**File structure:**

```markdown
---
ts: 1743408000
date: "2026-04-01"
agents: [tala, echo]
skill: /create-track
tags: [jazz, lo-fi, suno]
summary: "Jazz lo-fi track with brush drums, Echo flagged frequency collision in mids"
---

## User Input
{original request, vibe description, constraints}

## Discover
{matched references, music cards, brain.db results}

## Distill
{ingredients, health checks, hard limits, stacking detection}

## Variations
{each variation as built, with Style + Lyrics blocks}

## Critic Feedback
{tri-critic output, reconciliation, what changed}

## Final Output
{what was written to disk, file paths}

## Conversation
{key decision points, pivots, user feedback during session}
```

Sections are optional — non-creative sessions skip Variations/Critic Feedback, code reviews skip Discover/Distill. `writeRaw()` omits empty sections.

### Lifecycle Rules

- **Immutable.** Once written, raw files are never edited, merged, archived, or deleted.
- **Permanent.** No retention policy. No monthly distillation. The eidetic property is the point.
- **Excluded from consolidation.** `consolidate-memory.ts` skips `memory/raw/` entirely.
- **Excluded from memory sync.** `syncMemories()` in `sync.ts` must skip `memory/raw/` — raw has its own sync path. Without this guard, raw files leak into the `memories` table and pollute normal search. *(B1)*
- **Write before reflect.** Raw file is written first, then `reflect.ts` runs with `raw_ref` pointing to the timestamp. Reflection has access to the full transcript.
- **Atomic write.** `writeRaw()` writes to `.{timestamp}.md.tmp` then renames to `{timestamp}.md` to prevent corrupt partial files on crash. *(S4)*

### Reflection Integration

`reflect.ts` receives the raw timestamp and threads it through 4 touch points: *(R4)*

1. `ReflectionInput` — new `rawRef?: number` field
2. Reflection prompt — optional, raw context can be passed to Gemini
3. `writeLearnings()` — includes `raw_ref` in inbox observation frontmatter
4. `logDecision()` — includes `raw_ref` in brain.db decision record

```yaml
# in inbox observation frontmatter
raw_ref: 1743408000
```

```typescript
// in brain.db decision log
logDecision({ ..., raw_ref: 1743408000 })
```

Every reflected insight and logged decision traces back to the exact session transcript.

### brain.db Schema

**Parent table + chunks** (mirrors `references` → `reference_chunks` pattern): *(S2)*

| Table | Purpose |
|-------|---------|
| `raw_files` | Parent record per raw file — ts, date, agents, skill, tags, summary, source_path, file_hash |
| `raw_chunks` | Phase-level chunks with `raw_id` FK — section_name, content, line_start, line_end, token_count |
| `raw_fts` | FTS5 full-text index over raw chunks |
| `vec_raw` | 768-dim vector embeddings for semantic search |

The parent `raw_files` table enables per-session queries (`SELECT * FROM raw_files WHERE skill = '/create-track' ORDER BY ts DESC`) without touching chunks. *(S6)*

**Schema migration for `decisions` table:** `ALTER TABLE decisions ADD COLUMN raw_ref INTEGER` — plus extension of `DecisionInput` type and `logDecision()` to accept and persist the value. *(R2)*

**Chunking strategy:** Split at section boundaries (`## User Input`, `## Discover`, etc.) using existing `chunkMarkdown()` from `src/libs/brain/chunker.ts`. Each phase becomes one chunk. A typical raw file produces 5-7 chunks.

**Embedding cost:** ~5 embeddings per file × $0.000006 = negligible. ~$0.03/year at 1,000 sessions.

### Search Behavior

**Default search excludes raw.** Creative/generation queries (`--search-refs`, `--search-all`, `--hybrid`) hit references + memories + decisions only. The existing `stores` parameter in `searchAll()` controls this — raw is simply not in the default set.

**Recall search includes raw.** Two ways to access:

1. `--recall` flag — searches all stores INCLUDING raw. For conversational recall: "what did we discuss about authentication?", "last time we did jazz lo-fi?"

2. `--search-raw` flag — searches raw store ONLY. Direct eidetic lookup. Supports `--skill` filter for targeted session lookup (e.g., `--search-raw "jazz" --skill /create-track`). *(S6)*

**No auto-detect at launch.** Start with explicit `--recall` and `--search-raw` flags only. Freddie uses `--recall` when the user's question looks like memory recall. Auto-detection heuristics can be added later once usage patterns are clear. *(S5)*

**Ranking:**
- FTS5 keyword search on raw has no degradation — old raw files always findable by exact terms
- Vector search applies standard staleness degradation (90/180 day penalties) — old raw files rank lower in semantic search. Recent sessions surface first, but nothing is lost.

### Size Projections

| Metric | Per session | Per year (1,000 sessions) |
|--------|-------------|---------------------------|
| Markdown file | 5-10 KB | ~7.5 MB |
| brain.db vectors | 5 chunks × 3 KB = 15 KB | ~15 MB |
| brain.db FTS5 | ~2 KB | ~2 MB |
| Embedding cost | ~$0.00003 | ~$0.03 |

**Total brain.db growth:** ~17 MB/year from raw layer. SQLite handles hundreds of MB. sqlite-vec KNN stays fast up to ~100K vectors (we'd hit ~5,000/year).

**Obsidian Sync:** 7.5 MB/year is negligible against the 10 GB plan.

**Escape hatch:** If brain.db ever grows too large, drop and rebuild `vec_raw` without losing markdown files or FTS5 index. Markdown is always source of truth.

## Implementation Scope

### Files to Create
- `src/libs/raw.ts` — shared `writeRaw()` utility. Accepts structured sections, writes markdown to `memory/raw/{timestamp}.md`, returns `{ timestamp, filePath }`. Atomic write via tmp+rename. *(S3, S4)*
- `src/libs/brain/raw.ts` — raw_files + raw_chunks schema, insert, search, sync functions

### Files to Modify
- `src/libs/brain/schema.ts` — add `raw_files`, `raw_chunks`, `raw_fts`, `vec_raw` tables
- `src/libs/brain/index.ts` — migration: create raw tables + `ALTER TABLE decisions ADD COLUMN raw_ref INTEGER`
- `src/libs/brain/queries.ts` — add `--recall` and `--search-raw` modes, `skill` filter on raw search
- `src/libs/brain/sync.ts` — add raw folder sync path + **exclude `memory/raw/` from `syncMemories()`** *(B1)*
- `src/libs/reflect.ts` — add `rawRef` to `ReflectionInput`, thread through prompt/writeLearnings/logDecision *(R4)*
- `src/tools/brain.ts` — expose `--recall`, `--search-raw`, `--skill` CLI flags
- `.claude/rules/memory-boundaries.md` — document raw layer
- `.claude/rules/architecture.md` — add raw tables and search modes

### Wiring Points
Every workflow endpoint calls `writeRaw()` → passes timestamp to `reflect()`:
- Creative skills: `/create-track`, `/create-track-variant`, `/lyrics`, `/minimax-music`, `/minimax-art`, `/minimax-video`, `/tracknames`, `/create-music-card` *(S1)*
- Non-creative skills: `/architect`, `/code-review`, `/security-audit`, `/dev-start`, `/dev-end`
- Agent workflows: McCall analysis, Wick implementation, Ryan briefs/PRDs
- Freddie direct: architecture discussions, debugging, planning sessions — trigger mechanism TBD (likely manual `/save-raw` or end-of-session hook) *(R1)*

### Files Unchanged
- `src/tools/consolidate-memory.ts` — already skips non-standard folders, verify raw/ is excluded
- Agent files — no changes needed, raw capture happens at the orchestration layer

## Decisions

1. **Shared `writeRaw()` utility** — lives in `src/libs/raw.ts`, called by any skill/agent at end-of-workflow. Returns `{ timestamp, filePath }`. Atomic write.
2. **All sessions captured** — creative, architecture, code review, debugging. Any workflow that produces meaningful context gets a raw file.
3. **No file size cap** — variable sizes accepted. Chunking at section boundaries handles search regardless of file size.
4. **Parent `raw_files` table** — mirrors references → reference_chunks pattern. Enables per-session and per-skill queries without touching chunks.
5. **Explicit recall only** — no auto-detect heuristics at launch. Freddie uses `--recall` deliberately. Add auto-detection later.
6. **`raw_ref` on decisions** — schema migration adds column to existing `decisions` table for traceability.
7. **Sync guard** — `parseMemoryPath()` in `sync.ts` returns null for `agent === "raw"` to prevent raw files leaking into `memories` table.

## Review Findings (McCall)

All findings from architecture review have been incorporated:

| ID | Status | Resolution |
|----|--------|------------|
| B1 | RESOLVED | Sync guard added to lifecycle rules + sync.ts modification |
| R1 | ACKNOWLEDGED | Freddie direct trigger TBD — noted in wiring points |
| R2 | RESOLVED | Schema migration + type changes explicitly scoped |
| R3 | RESOLVED | Parent `raw_files` table + column spec added |
| R4 | RESOLVED | 4 touch points in reflect.ts explicitly listed |
| S1 | RESOLVED | Missing wiring points added |
| S2 | RESOLVED | Parent table adopted |
| S3 | RESOLVED | `writeRaw()` returns `{ timestamp, filePath }` |
| S4 | RESOLVED | Atomic write via tmp+rename |
| S5 | RESOLVED | Explicit `--recall` only, no auto-detect at launch |
| S6 | RESOLVED | `--skill` filter on raw search |
