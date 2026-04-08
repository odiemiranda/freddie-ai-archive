---
type: staged_proposal
agent: sol
date: 2026-04-08
title: "Refined Prompting Techniques (Part 1)"
section: "How I Think"
change_type: add
status: rejected
reason: "Judges rejected: gemini"
created: 2026-04-07T21:28:17.393Z
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
| drift | true | 30-day drift 4/6 (53-line soul, 10% threshold) |
| regression | true | No high-similarity rule matches |
| constitution | true | PASS |

## Judge Results

Approved: false
Dissents: gemini

| Model | Vote | ErrorType | Reason |
|-------|------|-----------|--------|
| gemini | reject | substantive | The proposed change is largely redundant, duplicating existing traits in the "How I Think" section almost verbatim, offering minimal new utility. |
| grok | approve | substantive | Change reinforces existing practices on precise descriptors, timbre anchors, and 'steady' preference with strong evidence from knowledge files, enhancing vocal/instrumental control without contradiction. |

## Reason for Staging

Judges rejected: gemini

## Decision

Status: rejected
Decided at: 2026-04-08T10:40:05.586Z
Decided by: user
Rejection reason: Gemini correctly identified as redundant with existing soul traits. Sol distill is producing duplication — same systemic issue as ASL-0018 (Penny) but different failure mode (redundancy vs evidence quality).
