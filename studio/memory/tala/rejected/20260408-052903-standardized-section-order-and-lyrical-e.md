---
type: staged_proposal
agent: tala
date: 2026-04-08
title: "Standardized Section Order and Lyrical Energy Arcs"
section: "Signature Moves"
change_type: add
status: rejected
reason: "Gates failed: constitution"
created: 2026-04-07T21:29:03.264Z
---

# Standardized Section Order and Lyrical Energy Arcs

## Proposed Change

Follows a strict section order: Structure tag → Energy → Mood → Vocal Style → Instrument → Texture → Lyrics.
Designs BGM lyrics with 'energy arcs' to control dynamic flow and intensity.
Uses `[Short Fade In]` for subtle track intros, an undocumented but reliable technique.

## Evidence

suno-prompt-engineering-and-references.md

## Gate Results

Passed: false

| Gate | Pass | Reason |
|------|------|--------|
| safety | true | No changes target protected sections |
| size | true | 3 lines added/modified, 0 removed — within limits |
| drift | true | 30-day drift 4/6 (58-line soul, 10% threshold) |
| regression | true | No high-similarity rule matches |
| constitution | false | FAIL.

The proposed "Signature Moves" section, |

## Judge Results

No judge panel run.

## Reason for Staging

Gates failed: constitution

## Decision

Status: rejected
Decided at: 2026-04-08T10:38:21.511Z
Decided by: user
Rejection reason: Constitution gate caught a real contradiction with existing soul content. Legitimate content-level rejection by the gate pipeline. Bulk rejected as part of 2026-04-08 review queue cleanup.
