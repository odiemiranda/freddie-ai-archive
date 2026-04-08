---
id: ASL-0022
title: Remove deprecated "wick" agent name from all allowlists, configs, and docstrings
project-code: ASL
status: done
priority: P3
created: 2026-04-08
author: freddie
depends-on: []
blocks: []
tags: [task, asl, cleanup, tech-debt, agent-identity]
---

# ASL-0022 — Remove deprecated "wick" references

## Background

"Wick" was the original developer agent name. During the agent-improvement project (shipped 2026-04-02), Wick was replaced by "Ryan — Analyst-turned-operator." CLAUDE.md and the agent table were updated, but several allowlists and code paths still reference `wick` as a valid agent identifier.

On 2026-04-08 during ASL-0017 follow-up, `holmes` was added to three allowlists (`consolidate-memory.ts`, `reflect.ts`, `timers.ts`) to unblock asl-sync. Rather than remove `wick` at the same time, it was left in place pending a safe audit of all remaining references.

## Known references to `wick` (from 2026-04-08 grep)

| File | Line | What it does | Risk if removed |
|---|---|---|---|
| `src/tools/consolidate-memory.ts` | 46 | `AGENTS` allowlist for consolidation | Low — no wick inbox exists |
| `src/libs/reflect.ts` | 76 | `AGENTS` allowlist for reflection | Low — same |
| `src/servers/freddie-ai/timers.ts` | 22 | `ALL_AGENTS` for scheduled timers | Low — same |
| `src/libs/brain/reranker.ts` | 57 | Per-agent rerank weights including a `wick:` key | **Medium — needs verification** |
| `src/tools/agent-save.ts` | 5, 13, 99 | CLI docstring examples and schema description | Low — docs only |

## Goals

1. Audit every `wick` reference in the codebase and classify as: safe to delete / needs migration / load-bearing (keep).
2. Delete all safe-to-delete references.
3. Migrate anything load-bearing to `ryan` (if the semantics still apply).
4. If brain.db has any rows keyed on `wick`, decide whether to archive them, migrate them, or drop them.

## Investigation path

1. `rg -n "\bwick\b" src/ vault/ .claude/` — comprehensive grep across code, data, and config.
2. `bun run tool brain --query "wick"` — check if brain.db has wick-keyed decisions, memories, or knowledge chunks that would orphan.
3. Check `vault/studio/memory/wick/` — does that directory exist? If yes, what's in it? (Likely superseded by `ryan/` but confirm.)
4. Check `src/libs/brain/reranker.ts:57` to understand what the wick reranker boost was doing — was it a dev-agent-specific weighting that ryan inherited, or dead config?
5. Check `.claude-personal/` for wick references — auto-memory files may have wick-era feedback.

## Why P3

No user-facing bug today. The presence of wick in allowlists is cruft, not failure. All real work flows through ryan. This is a cleanup task whose main value is preventing future confusion when someone greps the codebase and finds two agent names for one persona.

Priority could escalate to P2 if we discover brain.db data divergence (e.g., some rerank decisions using wick boost, others using ryan boost, producing inconsistent retrieval).

## Not in scope

- Removing the `ryan` agent or rerouting its responsibilities.
- Auditing other deprecated agent names (if any exist — none known at this time).
- Changing the reranker's per-agent boost mechanism itself.

## Context

- Holmes allowlist fix committed as `c04b303` on 2026-04-08.
- Agent improvement project context: `project_agent_improvement.md` in auto-memory.

## Completion notes (2026-04-09)

### Files changed

| File | Change |
|---|---|
| `src/tools/consolidate-memory.ts` | Removed `"wick"` from `AGENTS` array |
| `src/libs/reflect.ts` | Removed `"wick"` from `AGENTS` array |
| `src/servers/freddie-ai/timers.ts` | Removed `"wick"` from `ALL_AGENTS` array |
| `src/libs/brain/reranker.ts` | Deleted `wick:` boost profile block (refs: 0.6, memories: 1.3, decisions: 1.2). `ryan:` already exists (refs: 0.8, memories: 1.0, decisions: 1.3) — profiles diverged by design, `ryan` entry is authoritative |
| `src/tools/agent-save.ts` | Updated 3 docstring references: CLI example (line 5), usage example (line 13), schema `.describe()` (line 99) — all changed from `wick` to `ryan` |

### Reranker decision

`wick` and `ryan` had different profiles. `ryan` was already present and intentionally authored post-migration. `wick` was dead config — no code path calls `getProfile("wick")` since the agent rename. Deleted `wick` block without migrating weights to `ryan`. The profiles are not the same; `ryan`'s weights are the correct post-migration values.

### Brain.db data references found (informational, not touched)

- 2 memory inbox files with `agent: wick` in frontmatter (historical saves from 2026-03-31 worktree sessions, now in `memory/ryan/inbox/processed/`)
- 1 shared/knowledge chunk referencing "Wick" as a name example — data, not modified

### Vault memory

No `wick/` directory exists under `vault/studio/memory/` — was never created or already merged into `ryan/`.

### Test results

795 pass / 1 fail — pre-existing jazzhop failure unrelated to this change.
