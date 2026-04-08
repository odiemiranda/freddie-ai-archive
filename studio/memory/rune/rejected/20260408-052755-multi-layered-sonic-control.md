---
type: staged_proposal
agent: rune
date: 2026-04-08
title: "Multi-Layered Sonic Control"
section: "Signature Moves"
change_type: add
status: rejected
reason: "Judges rejected: grok"
created: 2026-04-07T21:27:55.353Z
---

# Multi-Layered Sonic Control

## Proposed Change

Employs a multi-layered control strategy, combining Style Block, Lyrics, and targeted 'Exclude' to achieve nuanced global sonic characteristics. Uses directive verbs within `[Instrument:]` tags for dynamic instrument roles. Relies on positive instrument constraints, reserving strategic exclusion for specific sonic artifacts or 'hallucinations'. Uses precise vocal style descriptors like 'Projected vocals'.

## Evidence

suno-prompt-engineering-and-references.md (Multi-Layered Prompt Control, Dynamic Instrument Roles, Strategic Exclusion, Vocal Style Specificity)

## Gate Results

Passed: true

| Gate | Pass | Reason |
|------|------|--------|
| safety | true | No changes target protected sections |
| size | true | 1 lines added/modified, 0 removed — within limits |
| drift | true | 30-day drift 11/11 (105-line soul, 10% threshold) |
| regression | true | No high-similarity rule matches |
| constitution | true | PASS |

## Judge Results

Approved: false
Dissents: grok

| Model | Vote | ErrorType | Reason |
|-------|------|-----------|--------|
| gemini | approve | substantive | The change refines existing prompt engineering principles, offering specific, actionable strategies for Suno, and aligns perfectly with Rune's established identity of precise control and authenticity. |
| grok | reject | substantive | Contradicts core philosophy of "positive constraints; rely on precise [Instrument:] tags over extensive Exclude lists" by introducing strategic exclusion reliance. |

## Reason for Staging

Judges rejected: grok

## Decision

Status: rejected
Decided at: 2026-04-08T10:40:04.596Z
Decided by: user
Rejection reason: Verbose AI-slop style: 4 concepts packed into one bullet. Not soul-grade. Even if grok's positive-constraints objection was borderline, the quality bar isn't met.
