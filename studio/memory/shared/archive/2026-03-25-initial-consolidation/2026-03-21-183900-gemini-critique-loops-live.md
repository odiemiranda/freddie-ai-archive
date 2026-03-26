---
id: mem-shared-20260321-183900-01
title: Gemini Critique Loops Live
created: 2026-03-21 18:39:00
tags: [gemini, critique-loop, quality-gate, style-block, lyrics]
type: decision
agent: shared
connected: []
---

Gemini critique loops are now built into Tala (step 8b: Style Critique Loop) and Rune (step 8b: Lyrics Critique Loop). Up to 3 passes per variation, early exit when no high-confidence issues remain. Tala uses `/sound`, Rune uses `/lyrics`.

**Why:** Style blocks and lyrics had no structured quality gate between generation and finalization. Gemini's sound_engineer and lyrics_critic agents catch instrument overloads, frequency clashes, clichés, and density issues that self-checks miss.

**How to apply:** Both loops run automatically during generation — no manual invocation needed. Gemini suggestions are classified by confidence and filtered against reference hard rules. Gemini rewrites are mined for individual fixes, never applied wholesale. Results logged in production notes (Tala) or Style Block Notes (Rune).
