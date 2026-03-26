---
source: reflection
agent: tala
task: "morning shamisen jazz instrumental — chaotic layering fix"
workflow: suno-generation
outcome: partial
scope: self
confidence: high
tags: [instrumental, bgm, suno, layering, structure-tags, exclude]
session: 2026-03-26T01:30:00
---

## Observation
Instrumental/BGM track "Morning Stillness" (lo-fi jazz, shamisen + harp + brushed drums) started well but became chaotic in the middle and end. Compared against a working version ("First Sip" from 2026-03-25) that used the same instruments without chaos. The working version had fundamentally different prompt architecture.

## Learning
Suno instrumental/BGM tracks require a specific prompt pattern that differs from vocal tracks. Six hard rules proven by A/B comparison:

1. **`[Instrumental]` in BOTH Style and Lyrics** — Style block must open with `[Instrumental]`. Lyrics block must also open with `[Instrumental]` as the first line. One without the other is unreliable.
2. **`[No Vocals]` + `[End]` at end of Lyrics** — Double reinforcement. Without both, Suno may add vocal elements or not properly fade.
3. **Max 5 sections for BGM** — Intro→Build→Hook→Hook→Outro. More sections (Verse, Bridge, second Verse) force Suno to differentiate each one by adding layers. Keep it simple.
4. **Aggressive Exclude list (20+ items)** — Block EVERYTHING except the 2-3 wanted instruments. The working version excluded 26 instruments. Sparse excludes (10 items) leave Suno room to add unwanted instruments.
5. **Hard harp constraints** — Harp must be explicitly told "low register only, no high notes, no melody." Without this, harp drifts into melodic territory and competes with lead.
6. **`[Background Music]` in Style** — Tells Suno to keep arrangement restrained. "background-safe" doesn't work — it's not a production term Suno recognizes.

Additional findings:
- **No "rise" or intensity language in Build sections** — Suno interprets "arpeggios rise" or "phrases slightly longer" as instructions to add density/layers.
- **Stereo panning instructions are ignored** — "harp left channel shamisen right" does nothing. Remove it.
- **Groove consistency instruction helps** — "Groove stays even and consistent" prevents Suno from evolving the rhythm.
- **Leaner Style = clearer signal** — ~250 chars beats ~400 chars for instrumental. Front-load genre and instruments.

## Evidence
- Working: `tracks/2026-03-25/161416-morning-window-groove/first-sip.md` — 5 sections, 26 excludes, [Instrumental] in both, harp "no high notes no melody"
- Broken: `tracks/2026-03-26/005511-morning-shamisen-jazz/morning-stillness.md` (original) — 8 sections, 10 excludes, no [Instrumental] in Lyrics, harp unconstrained
- Dual-critic review confirmed: Gemini flagged vinyl double-layering, MiniMax flagged Build intensity language as the chaos trigger
