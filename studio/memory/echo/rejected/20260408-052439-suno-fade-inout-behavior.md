---
type: staged_proposal
agent: echo
date: 2026-04-08
title: "Suno Fade In/Out Behavior"
section: "How I Think"
change_type: add
status: rejected
reason: "Gates failed: drift"
created: 2026-04-07T21:24:39.811Z
---

# Suno Fade In/Out Behavior

## Proposed Change

- Recognizes `[Fade In]` as a valid structural tag for intros in Suno, but flags `[Fade Out]` as unreliable due to consistent critic feedback.

## Evidence

`suno-prompt-engineering-and-references.md` (Undocumented but Observed Suno Behaviors)

## Gate Results

Passed: false

| Gate | Pass | Reason |
|------|------|--------|
| safety | true | No changes target protected sections |
| size | true | 1 lines added/modified, 0 removed — within limits |
| drift | false | Cumulative 30-day changes (6 lines) exceed 10% threshold (5 lines) of soul file (48 lines) |
| regression | false | skipped — phase 1 failure |
| constitution | false | skipped — phase 1 failure |

## Judge Results

No judge panel run.

## Reason for Staging

Gates failed: drift

## Decision

Status: rejected
Decided at: 2026-04-08T10:37:50.930Z
Decided by: user
Rejection reason: Drift gate working as designed — soul modified too heavily in recent 30-day window. Not a content problem. Proposal can be regenerated after drift cooldown. Bulk rejected as part of 2026-04-08 review queue cleanup.
