---
type: staged_proposal
agent: echo
date: 2026-04-08
title: "Consistent Attribution"
section: "How I Talk"
change_type: add
status: rejected
reason: "Gates failed: drift"
created: 2026-04-08T13:07:14.857Z
---

# Consistent Attribution

## Proposed Change

- Prefixes all commentary with `[Echo]:` for clear attribution in multi-agent conversations.

## Evidence

agent-communication-and-identity.md

## Gate Results

Passed: false

| Gate | Pass | Reason |
|------|------|--------|
| safety | true | No changes target protected sections |
| size | true | 1 lines added/modified, 0 removed — within limits |
| drift | false | Cumulative 30-day changes (6 lines) exceed 10% threshold (5 lines) of soul file (48 lines) |
| regression | false | skipped — phase 1 failure |
| constitution | false | skipped — phase 1 failure |

## Judge Results

No judge panel run.

## Reason for Staging

Gates failed: drift

## Decision

Status: rejected
Decided at: 2026-04-08T13:43:42.554Z
Decided by: user
Rejection reason: Bulk rejected as part of 2026-04-08 review queue clear-out. ASL-0024 (knowledge-delta gate + fail-closed degraded mode) is being shipped to prevent this proposal flood from recurring. Distill fired spuriously due to broken trigger (regenerated content from unchanged knowledge); this and similar proposals will not be re-staged after ASL-0024 ships unless real knowledge changes occur.
