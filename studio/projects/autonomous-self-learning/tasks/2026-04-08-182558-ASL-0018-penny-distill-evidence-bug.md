---
id: ASL-0018
title: Fix Penny distill: proposals cite knowledge files as evidence instead of observed behavior
project-code: ASL
status: planned
priority: P2
created: 2026-04-08
author: freddie
depends-on: []
blocks: []
tags: [task, asl, distill, penny, evidence-quality]
---

# ASL-0018 — Fix Penny distill evidence sourcing

## Root cause

All 12 Penny proposals staged on 2026-04-08 were rejected in a single batch after walking the review queue. They share one failure mode: Penny's `distill-soul` run produces proposals whose `Evidence` field is a single knowledge file (e.g. `obsidian-vault-conventions.md`, `studio-architecture-and-tools.md`, `workflow-quality-and-knowledge-management.md`), and both alive judges (gemini + grok — minimax was down) reject with variants of:

> "Evidence is a knowledge file, not observed behavior from multiple sessions."
> "Evidence is from a single document, not sufficient for a soul change."
> "Evidence from single file lacks multi-session validation."

The constitution gate passes these because the diff is structurally fine (1-line append, safe section). The gate doesn't check evidence quality. Judges catch it downstream.

## Why this is a Penny-specific problem

Other agents (Echo, Rune, Sol, Tala) have proposals in the same batch that clear judge evidence checks. They're getting evidence from raw session files or multi-document consolidations. Penny's distill is not.

Hypothesis: Penny's knowledge dir (`vault/studio/memory/penny/knowledge/`) contains a small set of dense knowledge files, and her distill prompt is probably grabbing one file per proposal without aggregating across sessions or inbox items.

## Goals

1. Penny distill output should cite **multi-source evidence** — either multiple knowledge files, or knowledge file + raw session references, or inbox item aggregation.
2. Judges should stop rejecting Penny proposals on evidence quality grounds.
3. Existing Penny knowledge files should not need to be rewritten — the fix is in the distill pipeline, not the knowledge layer.

## Investigation path

1. Read Penny's current knowledge dir and count distinct files. If there's only 1-2 files, the problem is upstream in consolidation, not distill.
2. Read `src/tools/distill-soul.ts` — find where evidence is constructed. Does it pull from one file or many? Does it include raw session references?
3. Compare with Echo/Rune distill invocations that produced proposals judges accepted — what did those evidence fields look like?
4. Check whether Penny's consolidation (memory inbox → knowledge) is producing knowledge files that contain session provenance (file timestamps, raw references) that distill could surface as evidence.

## Not in scope

- Rewriting Penny's existing knowledge files.
- Changing the judge prompts to be more lenient on evidence.
- Fixing minimax judge downtime (separate issue — minimax is down for all Penny proposals, suggesting a stability issue worth tracking separately).

## Context

- The 12 rejected proposals are archived at `vault/studio/memory/penny/rejected/` with rejection reasons.
- See `ASL-0017` for the migration that made judge errorType visible in the review flow — Penny's rejections were substantive (not api_error), so this is a real content problem.
- See Freddie's handling of the review queue on 2026-04-08 (session log for the day).
