---
type: staged_proposal
agent: sol
date: 2026-04-08
title: "Refined Descriptors for Artifact Prevention"
section: "How I Think"
change_type: add
status: rejected
reason: "Gates failed: drift"
created: 2026-04-08T12:53:36.102Z
---

# Refined Descriptors for Artifact Prevention

## Proposed Change

I use precise descriptors like 'harmonic beds' instead of generic 'pads' to prevent unwanted sonic artifacts such as 'synth drift', ensuring cleaner textures.

## Evidence

`suno-prompt-engineering-and-references.md`

## Gate Results

Passed: false

| Gate | Pass | Reason |
|------|------|--------|
| safety | true | No changes target protected sections |
| size | true | 1 lines added/modified, 0 removed — within limits |
| drift | false | Cumulative 30-day changes (7 lines) exceed 10% threshold (6 lines) of soul file (59 lines) |
| regression | false | skipped — phase 1 failure |
| constitution | false | skipped — phase 1 failure |

## Judge Results

No judge panel run.

## Reason for Staging

Gates failed: drift

## Decision

Status: rejected
Decided at: 2026-04-08T13:07:23.557Z
Decided by: user
Rejection reason: Drift gate: Sol soul has accumulated too many recent changes (from prior auto-apply batch). New proposals can't land until drift window cools. Not a content problem. Bulk rejected as part of 2026-04-08 post-sync cleanup.
