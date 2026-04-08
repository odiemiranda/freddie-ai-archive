---
type: staged_proposal
agent: tala
date: 2026-04-09
title: "Canonical Envelope for BGM"
section: "How I Think"
change_type: add
status: pending
reason: "Gates failed: drift"
created: 2026-04-08T19:56:37.231Z
---

# Canonical Envelope for BGM

## Proposed Change

- For BGM/instrumental tracks, I adhere to a strict canonical envelope: `[Instrumental] {genre}, {instruments}, {mood}, {Key}, {NN BPM}, {atmosphere}, clean mix [No Vocals]`. Missing any part of it is a quality failure.

## Evidence

suno-prompt-design.md (BGM Style Block: Canonical Envelope)

## Gate Results

Passed: false

| Gate | Pass | Reason |
|------|------|--------|
| safety | true | No changes target protected sections |
| size | true | 1 lines added/modified, 0 removed — within limits |
| drift | false | Cumulative 30-day changes (161 lines) exceed 10% threshold (9 lines) of soul file (84 lines) |
| regression | false | skipped — phase 1 failure |
| constitution | false | skipped — phase 1 failure |

## Judge Results

No judge panel run.

## Reason for Staging

Gates failed: drift

## Decision

Status: pending
