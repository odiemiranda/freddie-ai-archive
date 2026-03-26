---
id: mem-tala-20260323-060000-01
title: Structure Tags Belong in Lyrics Block Only
created: 2026-03-23 06:00:00
updated: 2026-03-25
tags: [suno, style-block, structure-tags, generation-quality]
type: discovery
agent: tala
connected: []
---

Structure tags (`[Intro]`, `[Build]`, `[Hook]`) belong in the **Lyrics block only**, not in the Style block. The Style block is a sonic palette — genre, instrument names, mood, BPM, key, texture keywords. Putting structure tags in Style overloads it and causes Suno to produce muddled, chaotic output.

**Why:** Testing the "morning window groove" track showed that embedding `[Intro]...[Build]...[Hook]` in the Style block alongside instrument descriptions and arrangement directions creates competing instructions — Suno tries to satisfy both the sonic palette and the arrangement roadmap from the same field and fails. Moving structure tags to the Lyrics block and keeping Style as pure palette produced clean, separated output.

**How to apply:** Style block format: `[Instrumental] genre, instrument names, mood keywords, BPM, key, quality keywords [No Vocals]`. All structure tags and arrangement flow go in the Lyrics block. Bracket tags in Lyrics should be 3-5 words max.

**Previous finding (superseded):** Earlier testing suggested inline structure tags in Style blocks improved quality. This was a correlation — the real improvement came from having structure at all. Structure in the Lyrics block achieves the same benefit without overloading Style.
