---
type: staged_proposal
agent: tala
date: 2026-04-09
title: "Refined Weirdness/Style Influence with W/SI"
section: "Signature Moves"
change_type: modify
status: pending
reason: "Gates failed: drift"
created: 2026-04-08T19:56:37.252Z
---

# Refined Weirdness/Style Influence with W/SI

## Proposed Change

Sets Weirdness/Style Influence (W/SI) by asking what each variation *needs*, using a deliberate gradient of parameters to control the output spectrum. Tight groove gets low weirdness, experimental fusion gets high. Never a mechanical staircase.

## Evidence

suno-agent-workflow-and-control.md (Parameter Tuning: Weight (W) and Suno Interpretation (SI)), suno-prompt-engineering-and-references.md (Variation Control: Weight (W) and Suno Interpretation (SI))

## Gate Results

Passed: false

| Gate | Pass | Reason |
|------|------|--------|
| safety | true | No changes target protected sections |
| size | true | 1 lines added/modified, 0 removed — within limits |
| drift | false | Cumulative 30-day changes (160 lines) exceed 10% threshold (9 lines) of soul file (84 lines) |
| regression | false | skipped — phase 1 failure |
| constitution | false | skipped — phase 1 failure |

## Judge Results

No judge panel run.

## Reason for Staging

Gates failed: drift

## Decision

Status: pending
