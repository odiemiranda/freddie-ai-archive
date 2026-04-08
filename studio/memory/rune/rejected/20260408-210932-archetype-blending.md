---
type: staged_proposal
agent: rune
date: 2026-04-08
title: "Archetype Blending"
section: "Creative Philosophy"
change_type: add
status: rejected
reason: "Gates failed: safety, drift"
created: 2026-04-08T13:09:32.695Z
---

# Archetype Blending

## Proposed Change

- For complex styles, explicitly defines a 'primary + secondary' archetype blend with specific vocal or thematic roles (e.g., 'chanting' for primary, 'spoken word' for secondary).

## Evidence

suno-prompt-engineering-and-references.md

## Gate Results

Passed: false

| Gate | Pass | Reason |
|------|------|--------|
| safety | false | Changes target protected sections: "Creative Philosophy" is a protected section |
| size | true | 1 lines added/modified, 0 removed — within limits |
| drift | false | Cumulative 30-day changes (12 lines) exceed 10% threshold (11 lines) of soul file (107 lines) |
| regression | false | skipped — phase 1 failure |
| constitution | false | skipped — phase 1 failure |

## Judge Results

No judge panel run.

## Reason for Staging

Gates failed: safety, drift

## Decision

Status: rejected
Decided at: 2026-04-08T13:43:57.535Z
Decided by: user
Rejection reason: Bulk rejected as part of 2026-04-08 review queue clear-out. ASL-0024 (knowledge-delta gate + fail-closed degraded mode) is being shipped to prevent this proposal flood from recurring. Distill fired spuriously due to broken trigger (regenerated content from unchanged knowledge); this and similar proposals will not be re-staged after ASL-0024 ships unless real knowledge changes occur.
