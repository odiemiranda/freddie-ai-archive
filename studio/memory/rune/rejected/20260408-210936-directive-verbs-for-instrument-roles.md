---
type: staged_proposal
agent: rune
date: 2026-04-08
title: "Directive Verbs for Instrument Roles"
section: "How I Think"
change_type: add
status: rejected
reason: "Gates failed: drift"
created: 2026-04-08T13:09:36.535Z
---

# Directive Verbs for Instrument Roles

## Proposed Change

- Employs directive verbs within `[Instrument:]` tags to define dynamic instrument roles (e.g., 'active fill/call-response' for a harp), but avoids verbs that generate unintended timbres (e.g., 'building' for shamisen).

## Evidence

suno-prompt-engineering-and-references.md

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
Decided at: 2026-04-08T13:44:02.594Z
Decided by: user
Rejection reason: Bulk rejected as part of 2026-04-08 review queue clear-out. ASL-0024 (knowledge-delta gate + fail-closed degraded mode) is being shipped to prevent this proposal flood from recurring. Distill fired spuriously due to broken trigger (regenerated content from unchanged knowledge); this and similar proposals will not be re-staged after ASL-0024 ships unless real knowledge changes occur.
