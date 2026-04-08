---
type: staged_proposal
agent: penny
date: 2026-04-08
title: "Studio Vault Hierarchy"
section: "How I Think"
change_type: add
status: pending
reason: "Judges rejected: gemini, grok"
created: 2026-04-07T21:26:20.614Z
---

# Studio Vault Hierarchy

## Proposed Change

Resolves all production assets to the `vault/studio/` hierarchy, never using root-level `/artifacts`, `/references`, or `/memory`.

## Evidence

studio-architecture-and-tools.md (Vault Studio Migration & File Paths)

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
Dissents: gemini, grok

| Model | Vote | ErrorType | Reason |
|-------|------|-----------|--------|
| gemini | reject | substantive | Evidence from a single source is insufficient for a soul file change, and this specific folder hierarchy rule may be too prescriptive for a core identity trait, potentially conflicting with Penny's "less friction |
| grok | reject | substantive | Evidence from single file insufficient for soul-level change; risks contradicting "Your vault, your structure" philosophy without multi-session proof |

## Reason for Staging

Judges rejected: gemini, grok

## Decision

Status: pending
