---
name: suno-style-block-guide
description: Tested Style block construction for ALL tracks — lean character targets, essential elements, and what to avoid.
type: knowledge
agent: tala
tags: [suno, style-block, char-budget, production, lean, unified]
---

# Suno Style Block Guide

## Character Budgets (Unified for ALL Tracks)

| Track Type | Target | Hard Max |
|---|---|---|
| All tracks (with typed brackets in Lyrics) | **~75-120 chars** | 200 |

### Why
Leaner prompts result in cleaner audio. Every element you add dilutes the core signal, making it harder for Suno to interpret. The Style block sets the global genre, BPM, and foundational mood, while detailed per-section control is handled by typed brackets in the Lyrics.

### How to apply
Adhere to the character budget above. Prioritize essential global elements and remove anything that can be implied by genre or controlled more precisely in the Lyrics section.

## What to Include (Never Cut)

### How to apply
Ensure these essential elements are always present in your Style block:
-   Genre (single word: "Jazz" not "Jazz fusion")
-   Key, BPM
-   "clean mix, separated instruments" (mandatory — helps audio clarity)
-   `[Instrumental]` and `[No Vocals]` tags (if applicable)

## What to Drop (for ALL Tracks)

### Why
These elements frequently confuse Suno, lead to hallucinations, are redundant, or are better controlled via typed brackets in the Lyrics section.

### How to apply
Avoid including these in your Style block:
-   **Chord progressions** — invite piano hallucination (tested: Gmaj7-Em7-Cmaj9-D7 caused piano at 75% mark).
-   **Textures** (e.g., tape warmth, rolled-off highs) — genre often implies these, or they can be specified per-section in Lyrics using `[Texture:]`.
-   **`[Background Music]` tag** — arrangement already implies it.
-   **Negative constraints** ("no melody", "no fills") — confuse Suno. Use positive descriptors instead, e.g., "pad chords", "sparse" in Lyrics `[Instrument:]` tags.
-   **Specific vocal styles** (e.g., 'deep chanting', 'whispered') — these belong in Lyrics using `[Vocal Style:]` tags.
-   **Specific instrument roles or behaviors** (e.g., "shamisen melodic plucks", "harp background pad chords", "brushed drums sparse") — these are better placed in Lyrics using `[Instrument:]` tags for per-section control.

## Reference Pattern (~79 chars)

```
[Instrumental] Jazz, G Major, 90 BPM, warm, clean mix, separated instruments [No Vocals]
```

---
*Consolidated from: `suno-style-block-guide.md` (original), `20260327-040817-specific-vocal-styles-e-g-deep-chanting--1.md`, `unified-typed-brackets.md`*
