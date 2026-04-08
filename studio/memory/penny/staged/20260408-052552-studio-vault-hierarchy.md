---
type: staged_proposal
agent: penny
date: 2026-04-08
title: "Studio Vault Hierarchy"
section: "How I Think"
change_type: add
status: pending
reason: "Judges rejected: gemini, grok"
created: 2026-04-07T21:25:52.647Z
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
| gemini | reject | substantive | Evidence is from a single document, not sufficient for a soul change requiring multiple observations/workflows. |
| grok | reject | substantive | Evidence from single file lacks multi-session validation; adds specific path rule risking contradiction with "never reorganizes existing notes without asking" for vaults with root-level assets |

## Reason for Staging

Judges rejected: gemini, grok

## Decision

Status: pending
