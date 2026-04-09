---
title: "Autonomous Self-Learning — ARCHIVED"
type: archive-notice
project: autonomous-self-learning
project-code: ASL
status: archived
archived: 2026-04-09
---

# ARCHIVED — 2026-04-09

The Autonomous Self-Learning (ASL) subsystem has been archived. All code has been moved to `src/archive/asl/` and is no longer wired into the live pipeline.

## Why

Session 2026-04-09 surfaced the cost-benefit pattern explicitly:

- **6/6 Tala proposals rejected** in a single morning review
- **4/6 were TECHNIQUE-class content** (structural generation rules) mislabeled as IDENTITY — these belonged in `memory/tala/knowledge/`, not the soul file
- **4/8 recent "approved" soul changes** were borderline or outright TECHNIQUE under strict `memory-boundaries.md` application — meaning the pipeline had been quietly corrupting soul files
- The proposed fix (ASL-0027 content-class classifier) would only eliminate ~60% of noise while adding 2 days of engineering, a live-Gemini eval fixture, and a new runtime LLM dependency
- Each historical layer of the pipeline (gates → judges → rejection blacklist → degraded-mode fail-closed → classifier) had been added to fix the previous layer's mistakes. The aggregate had become a pain point instead of a productivity multiplier.

## What replaced it

Nothing automated. The working half of the original pipeline — **reflect → consolidate → knowledge** — remains active and continues to provide compound learning across sessions:

- `reflect.ts` captures end-of-workflow surprises
- `consolidate-memory.ts` dedupes inbox → knowledge
- `discover.ts` loads relevant knowledge when agents run
- Agents become smarter across sessions because knowledge files persist

**Soul file updates are now manual.** When a session produces a genuine identity insight, Freddie or the user edits the soul file directly at task completion. When an agent appears to be lacking a capability, the response is to build a skill or tool — not to distill a rule into its soul.

## What this directory still contains

This directory is **preserved as engineering history**, not deleted. It contains:

- `PROJECT.md` — original project descriptor
- `BRIEF.md` — original architecture brief (daemon, pool, worker, file-watcher, task queue)
- `BRIEF-knowledge-classifier.md` — the revised brief from the 04-09 session that led to the archival decision
- `PRINCIPLES.md` — design principles and 2026-04-08 incident post-mortems (still valuable reading for anyone considering a similar system)
- `tasks/` — ~28 task docs from ASL-0001 through ASL-0027 (ASL-0027 was never shipped — see its status field)
- `research/` — daemon architecture findings, local LLM research

The lessons in `PRINCIPLES.md` and the task post-mortems remain valuable. Anyone considering building a similar autonomous-learning pipeline should read them first.

## Do not revive without

1. A new brief that accounts for the failure modes documented here and in `PRINCIPLES.md`
2. An honest cost-benefit assessment that compares against manual soul editing
3. User's explicit approval
4. A rollback plan

## Related code

- Archive location: `src/archive/asl/` (README there has the code inventory)
- brain.db tables `soul_changes`, `staged_proposals`, and `agent_lifecycle` remain in the schema (non-destructive). Their write paths are dead code inside the archive.
- Decision logged: brain.db `decisions` table, agent `freddie`, session 2026-04-09
