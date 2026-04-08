---
type: staged_proposal
agent: sol
date: 2026-04-08
title: "Clarified Style Field and Style Block Philosophy"
section: "Creative Philosophy"
change_type: modify
status: rejected
reason: "Gates failed: safety"
created: 2026-04-08T12:53:36.137Z
---

# Clarified Style Field and Style Block Philosophy

## Proposed Change

- The Style *field* is a 2,000-character opportunity; I use a lean Style *Block* for global tone and rich typed brackets for granular detail. Sparse prompts waste potential.

## Evidence

`suno-prompt-engineering-and-references.md`

## Gate Results

Passed: false

| Gate | Pass | Reason |
|------|------|--------|
| safety | false | Changes target protected sections: "Creative Philosophy" is a protected section |
| size | true | 1 lines added/modified, 0 removed — within limits |
| drift | true | 30-day drift 6/6 (59-line soul, 10% threshold) |
| regression | false | skipped — phase 1 failure |
| constitution | false | skipped — phase 1 failure |

## Judge Results

No judge panel run.

## Reason for Staging

Gates failed: safety

## Decision

Status: rejected
Decided at: 2026-04-08T13:07:46.192Z
Decided by: user
Rejection reason: Safety gate: targets protected sections (Creative Philosophy or similar identity-locked section). Hard bulk reject as part of 2026-04-08 post-sync cleanup.
