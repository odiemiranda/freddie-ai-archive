---
name: suno-style-block-guide
description: Tested Style block construction for ALL tracks — lean character targets, essential elements, what to avoid, and minimal exclude strategy.
type: knowledge
agent: tala
tags: [suno, style-block, char-budget, production, lean, unified, exclude, instruments, hallucination]
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

## Minimal Exclude Strategy

### Why
When every section has `[Instrument:]` typed brackets in the Lyrics, Suno has positive per-section instructions and doesn't hallucinate unwanted instruments. This positive guidance replaces the need for extensive negative exclusions. Adding abstract terms confuses Suno, as it struggles to interpret "what NOT to feel" compared to "what instrument NOT to use."

### How to apply
Only add dealbreakers (e.g., auto-tune for raw vocal, electronic for acoustic genres). Otherwise, leave the Exclude Style block empty. Do not add abstract terms like: `aggressive, uptempo, crescendo, dramatic build, drop, bright melody, distortion`.

---
*Consolidated from: `suno-style-block-guide.md`, `suno-exclude-strategy.md`, `20260327-040817-specific-vocal-styles-e-g-deep-chanting--1.md`, `unified-typed-brackets.md`*
