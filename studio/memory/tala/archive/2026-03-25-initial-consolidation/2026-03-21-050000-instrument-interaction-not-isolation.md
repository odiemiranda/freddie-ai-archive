---
id: mem-tala-20260321-050000-01
title: Instrument Interaction Not Isolation
created: 2026-03-21 05:00:00
updated: 2026-03-25
tags: [suno, lyrics, instrument-interaction, prompt-design, shamisen, harp]
type: feedback
agent: tala
confirmed: true
connected: []
---

Describe instrument INTERACTIONS in lyrics blocks, not isolated roles. "Shamisen leads, harp sustains underneath" produces a shamisen solo loop where harp vanishes. "Shamisen and harp trading phrases" produces a musical conversation.

**Why:** The track "two-tabs-left-open" worked because lyrics said "trading phrases", "weaves through chords", "just Rhodes and shamisen, breathing" — instruments related to each other. The track "corner-store-run" failed because lyrics described each instrument in isolation: "shamisen enters", "harp arpeggios climb", "shamisen holds the melody" — producing a repetitive shamisen loop with no harmonic movement.

**How to apply:** In lyrics blocks, use dynamic interaction verbs (trades, weaves, responds, answers) not static isolation verbs (holds, sustains, sits under). At least one line per section should describe how instruments relate. Structure tags (`[Intro]`, `[Build]`, `[Hook]`) go in the Lyrics block to give Suno the arrangement journey — not in the Style block.
