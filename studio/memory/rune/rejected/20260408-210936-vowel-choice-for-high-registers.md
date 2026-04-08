---
type: staged_proposal
agent: rune
date: 2026-04-08
title: "Vowel Choice for High Registers"
section: "How I Perceive Words"
change_type: modify
status: rejected
reason: "Gates failed: regression"
created: 2026-04-08T13:09:36.507Z
---

# Vowel Choice for High Registers

## Proposed Change

- **Vowel color.** Open vowels (a, o) feel expansive — use them for longing, openness, release. Closed vowels (i, ee) feel tight — use them for anxiety, confinement, urgency. For high registers, open vowels like "ay", "ee", "oh" sustain well on high notes.

## Evidence

vocal-performance-lyrics.md

## Gate Results

Passed: false

| Gate | Pass | Reason |
|------|------|--------|
| safety | true | No changes target protected sections |
| size | true | 1 lines added/modified, 0 removed — within limits |
| drift | true | 30-day drift 11/11 (107-line soul, 10% threshold) |
| regression | false | Change "[modify] - **Vowel color.** Open vowels (a, o) feel expansiv..." contradicts rule "- **Vowel color.** Open vowels (a, o) feel expansive — use t..." (similarity 0.91) |
| constitution | true | PASS |

## Judge Results

No judge panel run.

## Reason for Staging

Gates failed: regression

## Decision

Status: rejected
Decided at: 2026-04-08T13:43:59.305Z
Decided by: user
Rejection reason: Bulk rejected as part of 2026-04-08 review queue clear-out. ASL-0024 (knowledge-delta gate + fail-closed degraded mode) is being shipped to prevent this proposal flood from recurring. Distill fired spuriously due to broken trigger (regenerated content from unchanged knowledge); this and similar proposals will not be re-staged after ASL-0024 ships unless real knowledge changes occur.
