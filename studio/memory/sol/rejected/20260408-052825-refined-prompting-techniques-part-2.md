---
type: staged_proposal
agent: sol
date: 2026-04-08
title: "Refined Prompting Techniques (Part 2)"
section: "How I Think"
change_type: add
status: rejected
reason: "Judges rejected: gemini"
created: 2026-04-07T21:28:25.589Z
---

# Refined Prompting Techniques (Part 2)

## Proposed Change

- Uses `[Fade In]` for intros but avoids `[Fade Out]` due to its unreliability. Fine-tunes Music Card influence with `W`/`SI` parameters (e.g., `W:30 / SI:70`) to anchor warmth and tempo while allowing fusion elements. Prioritizes genres with stronger 'anchoring' effects and strategically excludes instruments to prevent genre-associated 'hallucinations'.

## Evidence

`suno-prompt-engineering-and-references.md` (Undocumented but Observed Suno Behaviors, W/SI for Music Cards, Genre Anchoring Strength, Strategic Exclusion)

## Gate Results

Passed: true

| Gate | Pass | Reason |
|------|------|--------|
| safety | true | No changes target protected sections |
| size | true | 1 lines added/modified, 0 removed — within limits |
| drift | true | 30-day drift 6/6 (57-line soul, 10% threshold) |
| regression | true | No high-similarity rule matches |
| constitution | true | PASS |

## Judge Results

Approved: false
Dissents: gemini

| Model | Vote | ErrorType | Reason |
|-------|------|-----------|--------|
| gemini | reject | substantive | The proposed change contains information already present in the soul file (W/SI for Music Cards) and the new fade/genre anchoring details are minor tactical refinements that don't significantly alter |
| grok | approve | substantive | Evidence from documented Suno behaviors aligns with existing Music Card and MiniMax optimization traits, enhancing prompt precision without contradiction or overreach. |

## Reason for Staging

Judges rejected: gemini

## Decision

Status: rejected
Decided at: 2026-04-08T10:40:07.566Z
Decided by: user
Rejection reason: Gemini: content already present in soul. Same Sol distill redundancy pattern — distill is regenerating existing traits as new proposals instead of identifying truly novel observed behavior.
