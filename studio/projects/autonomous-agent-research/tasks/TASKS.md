---
title: "AAR Phase 1: Self-Learning Autonomy — Task Index"
type: task-index
project: autonomous-agent-research
project-code: AAR
phase: 1
status: active
created: 2026-04-07
author: mccall
---

# AAR Phase 1: Self-Learning Autonomy

## Architecture Overview

Evolve the reflect-consolidate-distill pipeline to autonomous operation. Routine soul file refinements auto-apply through 5 validation gates and a 3-model judge panel. Unusual changes stage for human review. Morning briefing reports both.

```mermaid
graph TD
    R[reflect.ts<br/>importance 7+ only] --> C[consolidate-memory.ts<br/>per-session, no threshold]
    C --> D[distill-soul.ts<br/>proposes diff]
    D --> G[gates.ts<br/>5 validation gates]
    G -->|all pass| J[judge-panel.ts<br/>3 models, unanimous]
    G -->|any fail| S[staged/<br/>human review]
    J -->|approved| A[auto-apply<br/>+ soul_changes log]
    J -->|rejected| S
    S --> M[morning.ts<br/>briefing]
    A --> M
```

## Execution Order

```mermaid
graph LR
    T1[AAR-0001<br/>Importance Scoring] --> T2[AAR-0002<br/>Autonomous Consolidation]
    T2 --> T3[AAR-0003<br/>Validation Gates]
    T2 --> T4[AAR-0004<br/>Judge Panel]
    T2 --> T5[AAR-0005<br/>Staged Proposals]
    T3 --> T6[AAR-0006<br/>Morning Briefing]
    T4 --> T6
    T5 --> T6
    T6 --> T7[AAR-0007<br/>Orchestration Wiring]
    T3 --> T7
    T4 --> T7
    T5 --> T7
```

| Wave | Tasks | Constraint |
|------|-------|-----------|
| 1 (sequential) | AAR-0001, AAR-0002 | Both touch `src/libs/reflect.ts` — must be sequential |
| 2 (parallel) | AAR-0003, AAR-0004, AAR-0005 | Independent files — can run in parallel worktrees |
| 3 | AAR-0006 | Depends on soul_changes table + staged files existing |
| 4 | AAR-0007 | Depends on ALL prior tasks — wires them into end-to-end pipeline |

## Task Table

| ID | Title | Priority | Status | Files | Depends On |
|----|-------|----------|--------|-------|------------|
| AAR-0001 | Importance Scoring in reflect.ts | P1 | pending | `src/libs/reflect.ts`, `src/tools/reflect.ts` | none |
| AAR-0002 | Autonomous Consolidation | P1 | pending | `src/libs/reflect.ts`, `src/tools/consolidate-memory.ts` | AAR-0001 |
| AAR-0003 | Validation Gates | P2 | pending | `src/libs/gates.ts` (new), `src/libs/brain/schema.ts`, `src/libs/brain/index.ts`, `src/libs/brain/fts.ts` | AAR-0002 |
| AAR-0004 | LLM Judge Panel | P2 | pending | `src/libs/judge-panel.ts` (new) | AAR-0002 |
| AAR-0005 | Staged Proposals Storage + brain.db Integration | P2 | pending | `src/libs/brain/schema.ts`, `src/libs/brain/index.ts`, `src/libs/brain/fts.ts`, `src/libs/brain/sync.ts`, `src/libs/brain/queries.ts`, `src/tools/brain.ts` | AAR-0002 |
| AAR-0006 | Morning Briefing Integration | P3 | pending | `src/tools/morning.ts` | AAR-0003, AAR-0004, AAR-0005 |
| AAR-0007 | Orchestration Wiring | P1 | pending | `src/libs/autonomous-distill.ts` (new), `src/tools/autonomous-distill.ts` (new), `.claude/skills/bye/SKILL.md` | AAR-0001 through AAR-0006 |
