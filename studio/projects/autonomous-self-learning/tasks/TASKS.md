---
id: tasks-autonomous-self-learning
title: ASL Phase 2 — Task Registry
project-code: ASL
status: architecture-passed
created: 2026-04-07
updated: 2026-04-08
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
| ASL-0008 | Hook decoupling (remove distill from /bye) | `7538cdb` | [`2026-04-08-051140-ASL-0008-hook-decoupling.md`](2026-04-08-051140-ASL-0008-hook-decoupling.md) |
| ASL-0011 | External MCP readiness gate + reusable process lib | `9dacc4b` | [`2026-04-08-054002-ASL-0011-external-mcp-readiness-gate.md`](2026-04-08-054002-ASL-0011-external-mcp-readiness-gate.md) |
| ASL-0014 | Migrate 5 call sites to `src/libs/process.ts` | `f515f83` | [`2026-04-08-072057-ASL-0014-process-lib-migration.md`](2026-04-08-072057-ASL-0014-process-lib-migration.md) |
| ASL-0009 | `agent-state` CLI tool | `3fa4147` | [`2026-04-08-082222-ASL-0009-agent-state-cli.md`](2026-04-08-082222-ASL-0009-agent-state-cli.md) |
| ASL-0012 | Review-staged CLI for human-in-the-loop closure | `d8bda07` | [`2026-04-08-084847-ASL-0012-review-staged-cli.md`](2026-04-08-084847-ASL-0012-review-staged-cli.md) |
| ASL-0016 | Fix multi-line gate reason truncation in staged writer | `560b544` | [`2026-04-08-092231-ASL-0016-gate-reason-escape-fix.md`](2026-04-08-092231-ASL-0016-gate-reason-escape-fix.md) |
| ASL-0017 | Backfill legacy 3-col staged files to 4-col format + parser cleanup | `8a719c6` / `aa6f7bb` / `794ffbf` | [`2026-04-08-092231-ASL-0017-legacy-staged-backfill.md`](2026-04-08-092231-ASL-0017-legacy-staged-backfill.md) |
| ASL-0024 | Gate distill on knowledge-content delta + force-stage in degraded quorum | `cecd73b` | [`2026-04-08-214141-ASL-0024-knowledge-delta-gate-and-fail-closed-degraded.md`](2026-04-08-214141-ASL-0024-knowledge-delta-gate-and-fail-closed-degraded.md) |
| ASL-0023 | Rejection feedback loop + audit trail (Bug 2 residual closed WONTFIX) | `8442263` / `ae449e8` | [`2026-04-08-211556-ASL-0023-rejection-feedback-and-degraded-autoapply.md`](2026-04-08-211556-ASL-0023-rejection-feedback-and-degraded-autoapply.md) |
| ASL-0025 | Canonical compliance + critic authority reversal (5 tiers) | `428ab08` / `bdbd2f6` / `6ae6ea5` / `d37570f` | [`2026-04-09-012115-ASL-0025-canonical-compliance-and-critic-authority-reversal.md`](2026-04-09-012115-ASL-0025-canonical-compliance-and-critic-authority-reversal.md) |

## Immediate (next session)
| ASL-0021 | applySoulChange modify branch rewrite — shipped as `8d24a33` 2026-04-09 | P1 | **shipped** | — | [`2026-04-08-190000-ASL-0021-applysoulchange-refinement-semantics.md`](2026-04-08-190000-ASL-0021-applysoulchange-refinement-semantics.md) |
| ASL-0020 | Investigate MiniMax judge stability — transport failures and degraded mode | P1 | planned | — | [`2026-04-08-184156-ASL-0020-minimax-judge-stability.md`](2026-04-08-184156-ASL-0020-minimax-judge-stability.md) |
| ASL-0018 | Fix Penny distill: proposals cite knowledge files as evidence | P2 | planned | — | [`2026-04-08-182558-ASL-0018-penny-distill-evidence-bug.md`](2026-04-08-182558-ASL-0018-penny-distill-evidence-bug.md) |
| ASL-0019 | Fix Sol distill: proposals regenerate traits already in the soul | P2 | planned | — | [`2026-04-08-184156-ASL-0019-sol-distill-redundancy-bug.md`](2026-04-08-184156-ASL-0019-sol-distill-redundancy-bug.md) |
| ASL-0026 | Add tri-critic to /minimax-music + --lyrics-only mode for validate-envelope | P2 | planned | ASL-0025 | [`2026-04-09-025213-ASL-0026-minimax-tricritic-and-lyrics-only-envelope.md`](2026-04-09-025213-ASL-0026-minimax-tricritic-and-lyrics-only-envelope.md) |
| ASL-0022 | Remove deprecated "wick" agent name — shipped as `a418521` 2026-04-09 | P3 | **shipped** | — | [`2026-04-08-210759-ASL-0022-wick-deprecation-cleanup.md`](2026-04-08-210759-ASL-0022-wick-deprecation-cleanup.md) |

## Planned

| ID | Title | Priority | Status | Depends On | Doc |
|----|-------|----------|--------|------------|-----|
| ASL-0010 | Local LLM for gate-regression (Qwen/llama.cpp) | P2 | **deferred** (no GPU commitment for this phase) | ASL-0007 | — |

**ASL-0010 deferral note:** Research (`research/2026-04-07-local-llm-findings.md`) confirmed llama.cpp + GBNF grammar could deliver 0.75-0.85 accuracy on binary contradiction checks with the right prompt design, potentially hitting the 90% gate via Chain-of-Thought + curated few-shot + temperature 0. However, this is gated on GPU hardware — CPU-only inference at 3-6 tok/s produces 6+ minutes of local LLM runtime per pipeline run, defeating the entire decoupling effort. User has explicitly declined the GPU commitment for this phase. The task ID is preserved (not renumbered) so if a GPU becomes available or a future 4B model changes the CPU latency math, the slot and research are ready to revive.

## Conventions

- Task files go in this folder: `YYYY-MM-DD-HHmmSS-ASL-00NN-{slug}.md`
- McCall writes task docs. Ryan implements. McCall reviews. Freddie commits.
- Sequential only — no parallel worktrees unless explicitly authorized.
- Bundle review findings into the originating commit where possible.
- Research findings for ASL Phase 2 live under `../research/` — reference them explicitly in task docs when they inform a design choice.
