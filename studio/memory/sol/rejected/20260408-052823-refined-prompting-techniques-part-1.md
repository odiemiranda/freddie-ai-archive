---
type: staged_proposal
agent: sol
date: 2026-04-08
title: "Refined Prompting Techniques (Part 1)"
section: "How I Think"
change_type: add
status: rejected
reason: "Judges rejected: gemini"
created: 2026-04-07T21:28:23.615Z
---

# Refined Prompting Techniques (Part 1)

## Proposed Change

- Applies a multi-layered approach for nuanced sonic control, using precise vocal descriptors like 'Projected vocals'. Employs timbre anchors ('twangy') and refined terms ('harmonic beds') for instruments, preferring 'steady' over 'building' for delicate sounds like shamisen to avoid unintended timbres.

## Evidence

`suno-prompt-engineering-and-references.md` (Multi-Layered Prompt Control, Vocal Style Specificity, Descriptor Refinement, Dynamic Instrument Roles)

## Gate Results

Passed: true

| Gate | Pass | Reason |
|------|------|--------|
| safety | true | No changes target protected sections |
| size | true | 1 lines added/modified, 0 removed — within limits |
| drift | true | 30-day drift 5/6 (55-line soul, 10% threshold) |
| regression | true | No high-similarity rule matches |
| constitution | true | PASS |

## Judge Results

Approved: false
Dissents: gemini

| Model | Vote | ErrorType | Reason |
|-------|------|-----------|--------|
| gemini | reject | substantive | The proposed change largely duplicates existing traits in the 'How I Think' section, offering minimal new utility. |
| grok | approve | substantive | Change reinforces existing vocal-first philosophy and instrument refinement practices with high-confidence evidence, enhancing nuance without contradiction or overreach. |

## Reason for Staging

Judges rejected: gemini

## Decision

Status: rejected
Decided at: 2026-04-08T10:40:06.553Z
Decided by: user
Rejection reason: Duplicate of the other 'Refined Prompting Techniques (Part 1)' proposal — gemini redundancy objection applies identically. Sol distill producing dup proposals ~6s apart.
