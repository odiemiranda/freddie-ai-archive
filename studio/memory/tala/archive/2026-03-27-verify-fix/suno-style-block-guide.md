---
name: suno-style-block-guide
description: Tested Style block construction — lean char targets, slot formula, chord/texture rules, production keywords
type: knowledge
agent: tala
tags: [suno, style-block, char-budget, production, lean]
---

# Suno Style Block Guide

## Character Budgets (tested 2026-03-26)

| Track Type | Target | Hard Max | Evidence |
|---|---|---|---|
| Instrumental/BGM | **~140-170 chars** | 250 | 148 chars outperformed 291 chars on same track |
| Vocal (simple) | **~250-350 chars** | 450 | Vocals need genre + instrument + vocal character |
| Vocal (complex/fusion) | **~350-450 chars** | 500 | Fusion needs both genre anchors defined |

**Leaner = cleaner audio.** Every element you add dilutes the core signal.

## What to Include (never cut)

- Genre (single word: "Jazz" not "Jazz fusion" — tested, simpler = cleaner)
- Lead instrument + one behavior word ("shamisen melodic plucks")
- Support instrument + one role word ("harp background pad chords")
- Rhythm instrument + one character word ("brushed drums sparse")
- Key, BPM
- "clean mix, separated instruments" (mandatory — slot 10)
- `[Instrumental]` and `[No Vocals]` tags

## What to Drop (for instrumental)

- **Chord progressions** — invite piano hallucination (tested: Gmaj7-Em7-Cmaj9-D7 caused piano at 75% mark)
- **Textures** (tape warmth, rolled-off highs) — genre implies these
- **`[Background Music]` tag** — arrangement already implies it
- **Negative constraints** ("no melody", "no fills") — confuse. Use positive: "pad chords", "sparse"

## Reference Pattern (~148 chars)

```
[Instrumental] Jazz, shamisen melodic plucks, harp low arpeggios, brushed drums sparse, G Major, 90 BPM, warm, clean mix, separated instruments [No Vocals]
```
