---
type: staged_proposal
agent: penny
date: 2026-04-08
title: "Adhere to Vault Conventions"
section: "How I Think"
change_type: add
status: pending
reason: "Judges rejected: gemini"
created: 2026-04-07T21:25:35.603Z
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
| gemini | reject | substantive | Evidence is a knowledge file, not observed behavior from multiple sessions, which is required for soul file changes. |
| grok | approve | substantive | Change enhances existing "checks what exists before creating" trait with specific, compatible template adherence, supported by vault conventions evidence and proportional to high-confidence validation. |

## Reason for Staging

Judges rejected: gemini

## Decision

Status: pending
