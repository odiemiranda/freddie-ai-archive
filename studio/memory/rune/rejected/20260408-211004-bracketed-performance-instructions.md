---
type: staged_proposal
agent: rune
date: 2026-04-08
title: "Bracketed Performance Instructions"
section: "Signature Moves"
change_type: modify
status: rejected
reason: "Judges rejected: gemini"
created: 2026-04-08T13:10:04.229Z
---

# Bracketed Performance Instructions

## Proposed Change

- Uses punctuation and bracketed instructions as performance direction: ellipses for pauses (`...`), ALL CAPS for screaming, dashes for stretched syllables (`lo-o-o-ove`), parentheses for backing vocals (`(I'm still here)`), and specific brackets for instrumental breaks (`[instrumental break saxophone]`) or vocalizations (`[Whispered]`).

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
| gemini | reject | substantive | Evidence from a single knowledge file is insufficient for a soul file change, as multiple observations/workflows are required. |
| grok | approve | substantive | Evidence from Suno engineering doc supports expanded, precise performance directions that enhance compatibility with existing punctuation rules and improve lyric rendering without contradiction. |

## Reason for Staging

Judges rejected: gemini

## Decision

Status: rejected
Decided at: 2026-04-08T13:44:09.172Z
Decided by: user
Rejection reason: Bulk rejected as part of 2026-04-08 review queue clear-out. ASL-0024 (knowledge-delta gate + fail-closed degraded mode) is being shipped to prevent this proposal flood from recurring. Distill fired spuriously due to broken trigger (regenerated content from unchanged knowledge); this and similar proposals will not be re-staged after ASL-0024 ships unless real knowledge changes occur.
