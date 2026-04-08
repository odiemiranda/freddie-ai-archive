---
type: staged_proposal
agent: sol
date: 2026-04-08
title: "Updated Signature Move for Style Field Usage"
section: "Signature Moves"
change_type: modify
status: rejected
reason: "Judges rejected: gemini"
created: 2026-04-08T12:53:44.961Z
---

# Updated Signature Move for Style Field Usage

## Proposed Change

- Pushes the full 2,000-character Style *field* limit, leveraging a lean Style *Block* for global tone and detailed typed brackets for per-section nuance.

## Evidence

`suno-prompt-engineering-and-references.md`

## Gate Results

Passed: true

| Gate | Pass | Reason |
|------|------|--------|
| safety | true | No changes target protected sections |
| size | true | 1 lines added/modified, 0 removed — within limits |
| drift | true | 30-day drift 6/6 (59-line soul, 10% threshold) |
| regression | true | No high-similarity rule matches |
| constitution | true | PASS |

## Judge Results

Approved: false
Dissents: gemini

| Model | Vote | ErrorType | Reason |
|-------|------|-----------|--------|
| gemini | reject | substantive | Evidence provided is a single file reference, which may not constitute "sufficient evidence from multiple sessions/workflows" as required for a soul change. |
| grok | approve | substantive | Change refines existing "Pushes the full 2,000-character Style limit" trait with precise Suno-specific technique (lean Style Block + typed brackets), supported by evidence, non-contradictory, and utility-enhancing for nuanced prompting without drift. |

## Reason for Staging

Judges rejected: gemini

## Decision

Status: rejected
Decided at: 2026-04-08T13:43:39.185Z
Decided by: user
Rejection reason: Bulk rejected as part of 2026-04-08 review queue clear-out. ASL-0024 (knowledge-delta gate + fail-closed degraded mode) is being shipped to prevent this proposal flood from recurring. Distill fired spuriously due to broken trigger (regenerated content from unchanged knowledge); this and similar proposals will not be re-staged after ASL-0024 ships unless real knowledge changes occur.
