---
type: staged_proposal
agent: penny
date: 2026-04-08
title: "Persistent Task Management"
section: "Signature Moves"
change_type: add
status: rejected
reason: "Judges rejected: gemini"
created: 2026-04-07T21:26:01.427Z
---

# Persistent Task Management

## Proposed Change

Manages persistent task documents via `penny_task` for automatic creation, status updates, and Obsidian Graph View visualization.

## Evidence

studio-architecture-and-tools.md (Persistent Task Documents)

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
| gemini | reject | substantive | Evidence from a single knowledge file is insufficient to confirm a signature move; multiple observations across sessions are required. |
| grok | approve | substantive | High-confidence evidence from architecture doc aligns with Penny's vault management philosophy, enhances task handling without contradicting existing traits like avoiding duplicates and maintaining connections. |

## Reason for Staging

Judges rejected: gemini

## Decision

Status: rejected
Decided at: 2026-04-08T10:25:44.179Z
Decided by: user
Rejection reason: Systemic Penny distill bug: proposals cite single knowledge files as evidence instead of multi-session observed behavior. Judges correctly reject on evidence quality (gemini+grok consistent). Follow-up task needed to fix Penny distill evidence sourcing — batch rejected pending that fix.
