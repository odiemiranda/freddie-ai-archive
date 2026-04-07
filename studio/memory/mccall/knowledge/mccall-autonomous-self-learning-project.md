---
name: mccall Autonomous Self-Learning (ASL) Project Work
description: Details mccall's architectural design, task management, and implementation work for the Autonomous Self-Learning (ASL) project, including specific components and decisions.
type: knowledge
agent: mccall
tags: [autonomous-self-learning, ASL, AAR, project-management, architecture, system-design, task-management, worker-pool, daemon, distill-pipeline, LLM-judging, ADRs, Bun, testing, quality-assurance, migration]
---

## Knowledge
mccall has undertaken significant architectural design, task decomposition, and implementation work for the Autonomous Self-Learning (ASL) project, demonstrating robust development practices and critical review capabilities.

**Architectural Design & Review:**
*   **Effective Review Process**: mccall's architectural review process is effective at identifying critical implementation gaps, such as zero test coverage on the `isDistillEligible` guardrail, ensuring system robustness and adherence to quality standards.
*   **Phase 2 Architecture Pass**: Completed a Phase 2 architecture pass, updating `BRIEF.md` and reversing three core assumptions: worker model (now `Bun.spawn` over `worker_threads`), heartbeat (fixed lease), and coalesce timing (2-5s debounce over 1000ms).
*   **Deferred Features**: ASL-0010 (GPU integration) was deferred based on user decision.
*   **ADRs for Key Decisions**: Formal Architectural Decision Records (ADRs) are used to document critical design choices, including:
    *   File-system based distill cache (preferred over `brain.db`).
    *   Two-phase gate ordering with synthetic 'skipped' results for short-circuiting.
    *   Per-run rule embedding cache scoped to soul hash.
    *   Batched contradiction checks with `sim > 0.92` fallback.
    *   Flipping the 1-alive degradation policy for the judge panel (single judge cannot auto-apply, always staged with `degradedMode=true`).
    *   Explicitly defining `MINIMAX_PARSE_ERROR_MSG` and `MINIMAX_TIMEOUT_ERROR_MSG` constants for LLM interaction.

**Task Management & Implementation Details:**
*   **Task Document Design**: Designed detailed task documents for various ASL components, including:
    *   **AAR-0008 & AAR-0009**: For staged content bug fixes (AAR-0008, including 3 review findings and option B for frontmatter escape) and distill pipeline optimization (AAR-0009). These require sequential execution.
    *   **ASL-0001**: Updated with expanded acceptance criteria (22 to 29), new `JudgePanelResult.degradedMode` field, and specific markers for degraded mode reasons.
    *   **ASL-0002**: Schema-only migration for `tasks` and `agent_lifecycle` tables, explicitly placed before `initFts` to avoid a known bug.
    *   **ASL-0006 (Worker Pool)**: Initial design specified `child_process.spawn`, but was amended after review to use `Bun.spawn` to match `src/servers/freddie-ai/daemon.ts` and minimize Windows risk. Includes details on `consolidateAgent` and `autonomousDistill` extractions, chain reaction locking for idempotency, error classification (DB errors trigger worker shutdown, substantive errors use `failTask`), and pool backoff/crash handling (1/2/4/8/16s backoff, 5 crashes/60s = permanently-crashed).
    *   **ASL-0007 (Self-Learning Daemon)**: Task document covers `src/services/self-learning-daemon/index.ts` (composition/supervisor), `daemon.ts` (lifecycle controller), and 4 `package.json` scripts. Specifies locked startup/shutdown order (initBrain->pool->watcher, watcher->pool), no HTTP API, no log rotation, no auto-start, and no SIGTERM forwarding on Windows (handled by `taskkill /T`). Includes 25 acceptance criteria, 25 defensive reflexes, 16 grep audits, and a manual smoke test with 13 happy and 7 crash-recovery steps.
*   **Corrupted File Migration**: A one-shot tool was designed to archive 77 corrupted files to `staged/archive-pre-fix/`, followed by orphan removal via the existing `syncStaged` path.

## Why
This detailed documentation of mccall's ASL project work highlights its capabilities in complex system design, rigorous architectural review, meticulous task planning, and adaptive problem-solving (e.g., changing spawn API). It ensures traceability of key decisions, provides a clear understanding of the system's components and their interactions, and serves as a reference for future development, maintenance, and debugging.

## How to apply
When working on the ASL project, refer to these architectural decisions and task details for implementation guidance. Leverage mccall's review process to identify and address critical gaps early. Understand the rationale behind design choices documented in ADRs, especially concerning LLM interaction and worker pool behavior. Follow the specified task execution order and migration approaches. Be aware of deferred features and platform-specific considerations (e.g., Windows SIGTERM handling).

## Consolidated from
- `20260408-031114-mccall-s-architectural-review-process-is.md`
- `raw-log-20260407-152306-note.md`
- `raw-log-20260407-155611-note.md`
- `raw-log-20260407-183606-decision.md`
- `raw-log-20260407-205159-note.md`
- `raw-log-20260408-002004-note.md`
- `raw-log-20260408-005729-note.md`
- `raw-log-20260408-014832-note.md`
