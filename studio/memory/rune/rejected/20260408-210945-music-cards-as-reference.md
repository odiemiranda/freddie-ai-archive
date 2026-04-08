---
type: staged_proposal
agent: rune
date: 2026-04-08
title: "Music Cards as Reference"
section: "Signature Moves"
change_type: add
status: rejected
reason: "Gates failed: drift"
created: 2026-04-08T13:09:45.322Z
---

# Music Cards as Reference

## Proposed Change

- Uses Music Cards — playable Markdown files linking real-world audio samples to AI generation constraints — to inform BPM targets, instrument phrasing, and style translations.

## Evidence

studio-architecture-and-tools.md

## Gate Results

Passed: false

| Gate | Pass | Reason |
|------|------|--------|
| safety | true | No changes target protected sections |
| size | true | 1 lines added/modified, 0 removed — within limits |
| drift | false | Cumulative 30-day changes (12 lines) exceed 10% threshold (11 lines) of soul file (107 lines) |
| regression | false | skipped — phase 1 failure |
| constitution | false | skipped — phase 1 failure |

## Judge Results

No judge panel run.

## Reason for Staging

Gates failed: drift

## Decision

Status: rejected
Decided at: 2026-04-08T13:44:05.891Z
Decided by: user
Rejection reason: Bulk rejected as part of 2026-04-08 review queue clear-out. ASL-0024 (knowledge-delta gate + fail-closed degraded mode) is being shipped to prevent this proposal flood from recurring. Distill fired spuriously due to broken trigger (regenerated content from unchanged knowledge); this and similar proposals will not be re-staged after ASL-0024 ships unless real knowledge changes occur.
