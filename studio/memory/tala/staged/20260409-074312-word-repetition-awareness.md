---
type: staged_proposal
agent: tala
date: 2026-04-09
title: "Word Repetition Awareness"
section: "How I Think"
change_type: add
status: pending
degraded_quorum: "{\"alive\":2,\"dead\":[\"minimax\"]}"
reason: "degraded quorum: only 2 of 3 judges alive (dead: minimax (api))"
created: 2026-04-08T23:43:12.423Z
---

# Word Repetition Awareness

## Proposed Change

- Avoids repeating content words in Style Blocks or closely related typed brackets to prevent unintended sonic characteristics like monotone delivery.

## Evidence

suno-prompt-design.md (3.2. Word Repetition and Weighting)

## Gate Results

Passed: true

| Gate | Pass | Reason |
|------|------|--------|
| safety | true | No changes target protected sections |
| size | true | 1 lines added/modified, 0 removed — within limits |
| drift | true | 30-day drift 6/9 (84-line soul, 10% threshold) |
| regression | true | No high-similarity rule matches |
| constitution | true | PASS |

## Judge Results

Approved: true

| Model | Vote | ErrorType | Reason |
|-------|------|-----------|--------|
| gemini | approve | substantive | The change is well-supported by a canonical prompt design document, aligns perfectly with existing traits emphasizing precise language and avoiding repetition, and directly improves output quality by preventing monotone delivery. |
| grok | approve | substantive | High-confidence evidence from documented prompt design principles; semantically compatible with existing style block discipline and improves mix clarity by preventing sonic artifacts. |

## Reason for Staging

degraded quorum: only 2 of 3 judges alive (dead: minimax (api))

## Decision

Status: pending
