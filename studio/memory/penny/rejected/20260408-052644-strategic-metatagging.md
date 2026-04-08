---
type: staged_proposal
agent: penny
date: 2026-04-08
title: "Strategic Metatagging"
section: "How I Think"
change_type: add
status: rejected
reason: "Judges rejected: gemini"
created: 2026-04-07T21:26:44.147Z
---

# Strategic Metatagging

## Proposed Change

Applies metatags strategically, prioritizing higher-tier, weighted tags and aiming for 3-5 per section or prompt.

## Evidence

workflow-quality-and-knowledge-management.md (Metatag Management)

## Gate Results

Passed: true

| Gate | Pass | Reason |
|------|------|--------|
| safety | true | No changes target protected sections |
| size | true | 1 lines added/modified, 0 removed — within limits |
| drift | true | 30-day drift 1/5 (50-line soul, 10% threshold) |
| regression | true | No high-similarity rule matches |
| constitution | true | PASS |

## Judge Results

Approved: false
Dissents: gemini

| Model | Vote | ErrorType | Reason |
|-------|------|-----------|--------|
| gemini | reject | substantive | Evidence is from a single source document, not explicitly stated to be from multiple sessions/workflows, which is insufficient for a soul file change. |
| grok | approve | substantive | Evidence aligns with Penny's philosophy of clean, consistent frontmatter and tag management; enhances utility without contradicting existing traits like keeping frontmatter ordered. |

## Reason for Staging

Judges rejected: gemini

## Decision

Status: rejected
Decided at: 2026-04-08T10:25:51.937Z
Decided by: user
Rejection reason: Systemic Penny distill bug: proposals cite single knowledge files as evidence instead of multi-session observed behavior. Judges correctly reject on evidence quality (gemini+grok consistent). Follow-up task needed to fix Penny distill evidence sourcing — batch rejected pending that fix.
