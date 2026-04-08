---
type: staged_proposal
agent: sol
date: 2026-04-08
title: "Refined Exclusion Strategy"
section: "How I Think"
change_type: modify
status: rejected
reason: "Gates failed: constitution"
created: 2026-04-08T12:53:47.499Z
---

# Refined Exclusion Strategy

## Proposed Change

While I rely on positive `[Instrument:]` tags, I strategically use `Exclude` for targeted control, preventing unwanted instrument bleed or specific genre-associated 'hallucinations' (e.g., excluding 'violin' to emphasize 'fiddle').

## Evidence

`suno-prompt-engineering-and-references.md`

## Gate Results

Passed: false

| Gate | Pass | Reason |
|------|------|--------|
| safety | true | No changes target protected sections |
| size | true | 1 lines added/modified, 0 removed — within limits |
| drift | true | 30-day drift 6/6 (59-line soul, 10% threshold) |
| regression | true | No high-similarity rule matches |
| constitution | false | FAIL.\n\nThe proposed change, "While I rely |

## Judge Results

No judge panel run.

## Reason for Staging

Gates failed: constitution

## Decision

Status: rejected
Decided at: 2026-04-08T13:07:49.535Z
Decided by: user
Rejection reason: Constitution gate caught a real contradiction with existing soul content. Bulk rejected as part of 2026-04-08 post-sync cleanup.
