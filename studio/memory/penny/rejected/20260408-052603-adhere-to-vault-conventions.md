---
type: staged_proposal
agent: penny
date: 2026-04-08
title: "Adhere to Vault Conventions"
section: "How I Think"
change_type: add
status: rejected
reason: "Judges rejected: gemini"
created: 2026-04-07T21:26:03.544Z
---

# Adhere to Vault Conventions

## Proposed Change

Adheres to established vault structures and templates, always checking `vault/templates/` before creating new notes.

## Evidence

obsidian-vault-conventions.md (Vault Structure and Templates)

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
| gemini | reject | substantive | Evidence is a single knowledge file, not explicitly from multiple sessions/workflows as required for soul changes. |
| grok | approve | substantive | Aligns seamlessly with existing "checks what exists before creating" trait, enhances template consistency using vault-specific evidence, and improves utility without contradiction. |

## Reason for Staging

Judges rejected: gemini

## Decision

Status: rejected
Decided at: 2026-04-08T10:25:45.163Z
Decided by: user
Rejection reason: Systemic Penny distill bug: proposals cite single knowledge files as evidence instead of multi-session observed behavior. Judges correctly reject on evidence quality (gemini+grok consistent). Follow-up task needed to fix Penny distill evidence sourcing — batch rejected pending that fix.
