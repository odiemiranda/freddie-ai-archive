---
id: ASL-0019
title: Fix Sol distill: proposals regenerate traits already present in the soul
project-code: ASL
status: planned
priority: P2
created: 2026-04-08
author: freddie
depends-on: []
blocks: []
related: [ASL-0018]
tags: [task, asl, distill, sol, redundancy]
---

# ASL-0019 — Fix Sol distill redundancy

## Root cause

All 3 Sol proposals in the 2026-04-08 review queue were rejected on the same ground: **gemini repeatedly flagged them as duplicating traits already present in Sol's soul file**.

- "The proposed change is largely redundant, duplicating existing traits..."
- "The proposed change largely duplicates existing traits in the 'How I Think' section..."
- "The proposed change contains information already present in the soul..."

Grok approved all three (finding them well-evidenced reinforcements), so this isn't a judge disagreement about content quality in the abstract — it's specifically that Sol's distill is **regenerating** content from the current soul as if it were novel observed behavior.

## Relationship to ASL-0018

ASL-0018 is about Penny: her distill cites single knowledge files as evidence.
ASL-0019 is about Sol: her distill proposes changes already embedded in the current soul.

Both are distill-quality bugs, both produce judge rejections, but the failure modes differ:
- Penny: evidence layer broken (sources don't meet the multi-session bar)
- Sol: deduplication layer broken (doesn't compare proposals against current soul state)

They may share a root cause in the distill prompt (insufficient context about the *current* soul file, or missing self-comparison step), or they may be independent.

## Goals

1. Sol distill should compare candidate proposals against the current soul before staging.
2. Proposals whose content overlaps meaningfully with existing soul traits should be skipped at distill time, not surfaced to judges.
3. Ideally: a "redundancy check" step in the distill pipeline — embedding similarity between proposed change and each section of the current soul, with a threshold that triggers skip.

## Investigation path

1. Read `src/tools/distill-soul.ts` — find where the proposal content is constructed. Does it load the current soul file and compare? If yes, where does the comparison break? If no, that's the gap.
2. Check the distill prompt template — does it instruct the LLM to identify *new* behavior relative to existing soul, or just "what has the agent learned"?
3. Compare with the regression gate at `src/libs/gates/regression.ts` — that gate runs AFTER distill and catches some contradictions. The regression gate PASSED these Sol proposals (because they weren't contradictions, they were duplications). A redundancy check is a different check entirely — similarity, not contradiction.
4. Consider: should redundancy be a new GATE (at pipeline level, catching all agents) or a distill-level filter (agent-specific)? Gate is cheaper but catches later; distill filter is cleaner but requires per-agent instrumentation.

## Not in scope

- Rewriting Sol's existing soul content.
- Changing Sol's knowledge files.
- Judge prompt changes (gemini is correctly catching the problem — the fix is upstream).

## Context

- Rejected proposals archived at `vault/studio/memory/sol/rejected/` with rejection reasons.
- Same day as ASL-0018 was filed for the Penny equivalent.
- Minimax was down for all Sol proposals (degraded mode) — see ASL-0020 for the judge stability investigation.
