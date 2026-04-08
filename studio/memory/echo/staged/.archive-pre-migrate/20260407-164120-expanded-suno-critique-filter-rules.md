---
type: staged_proposal
agent: echo
date: 2026-04-07
title: "Expanded Suno Critique Filter Rules"
section: "How I Think"
change_type: modify
status: pending
reason: "Judges rejected: minimax"
created: 2026-04-07T08:41:20.427Z
---

# Expanded Suno Critique Filter Rules

## Proposed Change

- For Suno, Gemini's structural observations (frequency collisions, missing tags, redundancy, tag counts) are high-confidence. Its per-instrument mixing, production effects, wholesale rewrites, or atmospheric stripping suggestions are low-confidence and often contradict Suno's behavior; I mine individual fixes from these rather than applying them wholesale.

## Evidence

gemini-suno-critique-filter-rules.md

## Gate Results

Passed: true

| Gate | Pass | Reason |
|------|------|--------|
| safety | true | No changes target protected sections |
| size | true | 1 lines added/modified, 0 removed — within limits |
| drift | true | 30-day drift 3/5 (46-line soul, 10% threshold) |
| regression | true | 1 high-similarity pair(s) checked, no contradictions |
| constitution | true | PASS |

## Judge Results

Approved: false
Dissents: minimax

| Model | Vote | Reason |
|-------|------|--------|
| gemini | approve | The change refines an existing operational rule, adding useful detail and a practical strategy for handling low-confidence suggestions, aligning with Echo's core philosophy of providing data and options. |
| minimax | reject | API error: Failed to parse JSON |
| grok | approve | High-quality evidence from knowledge files refines existing Suno-specific confidence rules without contradiction, enhancing critique precision proportionally. |

## Reason for Staging

Judges rejected: minimax

## Decision

Status: pending
