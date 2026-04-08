---
type: staged_proposal
agent: tala
date: 2026-04-08
title: "New Signature Move: Subtle Intros"
section: "Signature Moves"
change_type: add
status: pending
reason: "Judges rejected: grok"
created: 2026-04-07T21:29:44.981Z
---

# New Signature Move: Subtle Intros

## Proposed Change

- Uses `[Short Fade In]` for subtle track intros, an undocumented trick that just works.

## Evidence

`suno-prompt-design.md` (section 2), `suno-prompt-engineering-and-references.md` (section "Undocumented but Observed Suno Behaviors")

## Gate Results

Passed: true

| Gate | Pass | Reason |
|------|------|--------|
| safety | true | No changes target protected sections |
| size | true | 1 lines added/modified, 0 removed — within limits |
| drift | true | 30-day drift 5/8 (71-line soul, 10% threshold) |
| regression | true | No high-similarity rule matches |
| constitution | true | PASS |

## Judge Results

Approved: false
Dissents: grok

| Model | Vote | ErrorType | Reason |
|-------|------|-----------|--------|
| gemini | approve | substantive | The change is well-supported by evidence from multiple sources, aligns with Tala's pragmatic approach to prompt tags and subtle sonic control, and adds a useful, specific technique to the agent's toolkit. |
| grok | reject | substantive | Contradicts "Pet Peeves" aversion to unreliable fade tags like `[Fade Out]`, despite evidence; risks semantic inconsistency without stronger proof of universal reliability. |

## Reason for Staging

Judges rejected: grok

## Decision

Status: pending
