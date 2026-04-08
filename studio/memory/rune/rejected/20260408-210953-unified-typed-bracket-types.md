---
type: staged_proposal
agent: rune
date: 2026-04-08
title: "Unified Typed Bracket Types"
section: "Signature Moves"
change_type: modify
status: rejected
reason: "Judges rejected: gemini"
created: 2026-04-08T13:09:53.849Z
---

# Unified Typed Bracket Types

## Proposed Change

- Leverages a full palette of typed brackets for granular control: `[Energy: Low/Rising/High/Maximum]`, `[Mood: descriptor]`, `[Texture: descriptor]`, `[Vocal Style: descriptor]`, and `[Instrument: lead | support | rhythm]`.

## Evidence

suno-prompt-engineering-and-references.md

## Gate Results

Passed: true

| Gate | Pass | Reason |
|------|------|--------|
| safety | true | No changes target protected sections |
| size | true | 1 lines added/modified, 0 removed — within limits |
| drift | true | 30-day drift 11/11 (107-line soul, 10% threshold) |
| regression | true | No high-similarity rule matches |
| constitution | true | PASS |

## Judge Results

Approved: false
Dissents: gemini

| Model | Vote | ErrorType | Reason |
|-------|------|-----------|--------|
| gemini | reject | substantive | Evidence is from a single knowledge file, not multiple observations/workflows, and the new tags could conflict with existing limits on tag usage. |
| grok | approve | substantive | Evidence from suno-prompt-engineering aligns with existing tag expertise, enhances granular control without contradicting positive constraints or limits, improving Suno output utility proportionally |

## Reason for Staging

Judges rejected: gemini

## Decision

Status: rejected
Decided at: 2026-04-08T13:44:07.570Z
Decided by: user
Rejection reason: Bulk rejected as part of 2026-04-08 review queue clear-out. ASL-0024 (knowledge-delta gate + fail-closed degraded mode) is being shipped to prevent this proposal flood from recurring. Distill fired spuriously due to broken trigger (regenerated content from unchanged knowledge); this and similar proposals will not be re-staged after ASL-0024 ships unless real knowledge changes occur.
