---
type: staged_proposal
agent: rune
date: 2026-04-08
title: "Strategic Exclusion"
section: "How I Think"
change_type: modify
status: rejected
reason: "Judges rejected: grok"
created: 2026-04-08T13:09:45.311Z
---

# Strategic Exclusion

## Proposed Change

- Build with positive constraints; rely on precise `[Instrument:]` tags over extensive `Exclude` lists. Strategic `Exclude` is reserved for targeted control, preventing unwanted instrument bleed, ensuring unique instrument rendering, or blocking specific genre-associated 'hallucinations'.

## Evidence

suno-prompt-engineering-and-references.md

## Gate Results

Passed: true

| Gate | Pass | Reason |
|------|------|--------|
| safety | true | No changes target protected sections |
| size | true | 1 lines added/modified, 0 removed — within limits |
| drift | true | 30-day drift 11/11 (107-line soul, 10% threshold) |
| regression | true | No high-similarity rule matches |
| constitution | true | PASS |

## Judge Results

Approved: false
Dissents: grok

| Model | Vote | ErrorType | Reason |
|-------|------|-----------|--------|
| gemini | approve | substantive | The change refines an existing guideline, providing crucial nuance for strategic `Exclude` tag use to prevent known issues like 'instrument bleed' and 'hallucinations', aligning with Rune' |
| grok | reject | substantive | Evidence from single file insufficient for core philosophy shift; contradicts existing "rely on precise [Instrument:] tags over extensive Exclude lists" without multi-session proof of utility |

## Reason for Staging

Judges rejected: grok

## Decision

Status: rejected
Decided at: 2026-04-08T13:44:04.260Z
Decided by: user
Rejection reason: Bulk rejected as part of 2026-04-08 review queue clear-out. ASL-0024 (knowledge-delta gate + fail-closed degraded mode) is being shipped to prevent this proposal flood from recurring. Distill fired spuriously due to broken trigger (regenerated content from unchanged knowledge); this and similar proposals will not be re-staged after ASL-0024 ships unless real knowledge changes occur.
