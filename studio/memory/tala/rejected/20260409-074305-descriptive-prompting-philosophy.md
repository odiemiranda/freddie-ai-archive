---
type: staged_proposal
agent: tala
date: 2026-04-09
title: "Descriptive Prompting Philosophy"
section: "Creative Philosophy"
change_type: add
status: rejected
reason: "Gates failed: safety"
created: 2026-04-08T23:43:05.212Z
---

# Descriptive Prompting Philosophy

## Proposed Change

- Prompts should be descriptive, setting a scene, mood, or theme, rather than prescriptive, to give Suno creative latitude.

## Evidence

suno-prompt-design.md (3.1. Descriptive vs. Prescriptive Prompting), suno-prompt-engineering-and-references.md (Prompt Design Philosophy)

## Gate Results

Passed: false

| Gate | Pass | Reason |
|------|------|--------|
| safety | false | Changes target protected sections: "Creative Philosophy" is a protected section |
| size | true | 1 lines added/modified, 0 removed — within limits |
| drift | true | 30-day drift 6/9 (84-line soul, 10% threshold) |
| regression | false | skipped — phase 1 failure |
| constitution | false | skipped — phase 1 failure |

## Judge Results

No judge panel run.

## Reason for Staging

Gates failed: safety

## Decision

Status: rejected
Decided at: 2026-04-09T07:25:57.970Z
Decided by: user
Rejection reason: Protected section (safety gate). Also content is a reference-file rule (suno-prompt-design.md §3.1), not soul material. And the rule is genre-dependent — descriptive works for ambient/lo-fi but contradicts the tight BGM envelope. Codifying it would create internal conflict.
