---
type: staged_proposal
agent: rune
date: 2026-04-08
title: "Brand Avoidance & Dynamic Instruments"
section: "Creative Philosophy"
change_type: add
status: rejected
reason: "Gates failed: safety"
created: 2026-04-07T21:27:05.676Z
---

# Brand Avoidance & Dynamic Instruments

## Proposed Change

- Avoid brand names; generic references keep the focus on emotion and ensure lyrics remain timeless.
- Wield structure tags as dynamic instruments: use directive verbs in `[Instrument:]` tags to define their active roles.

## Evidence

lyrics-authenticity-and-quality.md (Avoid Brands), suno-prompt-engineering-and-references.md (Dynamic Instrument Roles, Directive Verbs)

## Gate Results

Passed: false

| Gate | Pass | Reason |
|------|------|--------|
| safety | false | Changes target protected sections: "Creative Philosophy" is a protected section |
| size | true | 2 lines added/modified, 0 removed — within limits |
| drift | true | 30-day drift 3/9 (89-line soul, 10% threshold) |
| regression | false | skipped — phase 1 failure |
| constitution | false | skipped — phase 1 failure |

## Judge Results

No judge panel run.

## Reason for Staging

Gates failed: safety

## Decision

Status: rejected
Decided at: 2026-04-08T10:38:02.430Z
Decided by: user
Rejection reason: Safety gate: targets protected sections (e.g., 'Creative Philosophy') which are identity, not rules. Protected sections are locked by design. Bulk rejected as part of 2026-04-08 review queue cleanup.
