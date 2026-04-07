---
id: tasks-autonomous-self-learning
title: ASL Phase 2 — Task Registry
project-code: ASL
status: architecture-passed
created: 2026-04-07
updated: 2026-04-07
tags: [tasks, registry]
---

# ASL Phase 2 — Task Registry

Task numbering: `ASL-0001` onwards. Project code: `ASL`. **Numbering is permanent** — deferred or cancelled tasks retain their IDs to preserve stable references in decisions, commits, and cross-task links.

Tasks are created after the design has been validated via NotebookLM research (Holmes, 2026-04-07) and a McCall architecture pass (2026-04-07). ASL-0001 was an independent defensive fix that shipped ahead of the daemon work. The BRIEF was updated with research findings — three core assumptions were reversed (worker model, lease semantics, file-watcher pattern) and ASL-0010 was deferred per user decision.

## Shipped

| ID | Title | Commit | Doc |
|----|-------|--------|-----|
| ASL-0001 | MiniMax judge parser defensive fix + judge health check pre-flight | `8e7a92d` | [`2026-04-07-181755-ASL-0001-minimax-parser-fix.md`](2026-04-07-181755-ASL-0001-minimax-parser-fix.md) |
| ASL-0002 | brain.db schema: `tasks` + `agent_lifecycle` tables | `1f5fa1a` | [`2026-04-07-203621-ASL-0002-brain-db-schema.md`](2026-04-07-203621-ASL-0002-brain-db-schema.md) |
| ASL-0003 | Task queue library (heartbeat-based claim/lease) | `d61f6fc` | [`2026-04-07-210159-ASL-0003-task-queue-library.md`](2026-04-07-210159-ASL-0003-task-queue-library.md) |
| ASL-0004 | Agent lifecycle library (knowledge-dir hash staleness) | `bab968c` | [`2026-04-07-211314-ASL-0004-agent-lifecycle-library.md`](2026-04-07-211314-ASL-0004-agent-lifecycle-library.md) |
| ASL-0005 | File watcher service (chokidar@3.6.0 + polling + 1000ms coalesce) | `55132c8` | [`2026-04-07-215052-ASL-0005-file-watcher-service.md`](2026-04-07-215052-ASL-0005-file-watcher-service.md) |
| ASL-0006 | Worker pool (child processes via `Bun.spawn()` with IPC) | `a40ea77` | [`2026-04-08-000822-ASL-0006-worker-pool.md`](2026-04-08-000822-ASL-0006-worker-pool.md) |
| ASL-0007 | Daemon entry point + lifecycle scripts | `5716043` | [`2026-04-08-013633-ASL-0007-daemon-entry-point.md`](2026-04-08-013633-ASL-0007-daemon-entry-point.md) |
| ASL-0013 | asl-sync CLI (one-shot foreground catch-up) | `d60739c` | [`2026-04-08-022225-ASL-0013-asl-sync-cli.md`](2026-04-08-022225-ASL-0013-asl-sync-cli.md) |

## Immediate (next session)

| ID | Title | Priority | Status | Depends On |
|----|-------|----------|--------|------------|
| ASL-0008 | Hook decoupling (remove distill from session-end hooks) | P1 | planned | ASL-0007 |

## Planned

| ID | Title | Priority | Status | Depends On |
|----|-------|----------|--------|------------|
| ASL-0008 | Hook decoupling (remove distill from session-end hooks) | P1 | planned | ASL-0007 |
| ASL-0009 | `agent-state` CLI tool | P2 | planned | ASL-0004 |
| ASL-0010 | Local LLM for gate-regression (Qwen/llama.cpp) | P2 | **deferred** (no GPU commitment for this phase) | ASL-0007 |
| ASL-0011 | External MCP readiness gate (settings flag + startup check) | P1 | planned | ASL-0001, all P1 above |
| ASL-0012 | Review-staged CLI tool | P2 | planned | ASL-0002 |

**ASL-0010 deferral note:** Research (`research/2026-04-07-local-llm-findings.md`) confirmed llama.cpp + GBNF grammar could deliver 0.75-0.85 accuracy on binary contradiction checks with the right prompt design, potentially hitting the 90% gate via Chain-of-Thought + curated few-shot + temperature 0. However, this is gated on GPU hardware — CPU-only inference at 3-6 tok/s produces 6+ minutes of local LLM runtime per pipeline run, defeating the entire decoupling effort. User has explicitly declined the GPU commitment for this phase. The task ID is preserved (not renumbered) so if a GPU becomes available or a future 4B model changes the CPU latency math, the slot and research are ready to revive.

## Conventions

- Task files go in this folder: `YYYY-MM-DD-HHmmSS-ASL-00NN-{slug}.md`
- McCall writes task docs. Ryan implements. McCall reviews. Freddie commits.
- Sequential only — no parallel worktrees unless explicitly authorized.
- Bundle review findings into the originating commit where possible.
- Research findings for ASL Phase 2 live under `../research/` — reference them explicitly in task docs when they inform a design choice.
