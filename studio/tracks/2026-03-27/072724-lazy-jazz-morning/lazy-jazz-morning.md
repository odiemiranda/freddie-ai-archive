# Lazy Jazz Morning

> Generated from: "lazy jazz morning with shamisen and harp, brushed drums, instrumental"
> Generated At: 2026-03-27 07:27:24 AM UTC+08:00

---

## Original Prompt
> Lazy jazz morning instrumental — shamisen and harp over brushed drums. Warm, unhurried, background-safe.

## Processed Configuration
| Field | Value |
|-------|-------|
| Genre | Jazz (instrumental) |
| Instruments | Lead: shamisen bright plucks / harp melodic arpeggios (varies by variation). Support: harp low-register chords / shamisen low plucks (inverted per variation). Rhythm: brushed drums |
| Key | D Major (V1, V3), G Major (V2) |
| BPM | 85-95 (V1: 90, V2: 85, V3: 95) |
| Mood | warm, unhurried, hopeful, peaceful |
| Texture | clean mix, separated instruments |
| Exclude | vocals, singing |

---

## Variations

| # | File | BPM | Key | W | SI |
|---|------|-----|-----|---|-----|
| 1 | [Still Warm](./still-warm.md) | 90 | D Major | 35% | 75% |
| 2 | [Golden Warmth](./golden-warmth.md) | 85 | G Major | 35% | 75% |
| 3 | [First Light Stride](./first-light-stride.md) | 95 | D Major | 35% | 75% |

---

## Variation Summary

1. **Still Warm** — Shamisen-forward. Shamisen leads every phrase with harp sitting low and distant as harmonic bed. 90 BPM, D Major. The quietest of the three — a morning where nothing needs to happen yet.
2. **Golden Warmth** — Harp-forward. Harp leads with gentle arpeggios while shamisen provides low rhythmic plucks underneath. 85 BPM, G Major. Slower, more golden-hour afternoon energy despite the morning framing.
3. **First Light Stride** — Groovy hopeful. Both instruments trade phrases and share the spotlight. 95 BPM, D Major. The most energetic variation — a morning where you're actually looking forward to the day.

---

## Production Notes

- All three variations use the **typed bracket pattern** ([Energy:], [Mood:], [Instrument:] per section) to keep Style blocks ultra-lean (~185-194 chars) while controlling arrangement from Lyrics.
- **Tri-critic pipeline** ran on all 3 variations (Gemini + MiniMax + Grok). Common flags across the set: Gemini consistently wanted to strip instrument behavior from Style (rejected — Suno needs behavior in Style, typed brackets are additive). Grok consistently flagged Jazz genre flood risk (noted — user chose minimal excludes intentionally).
- **Key accepted changes across set:** "drifting" → "sparse" (V1), "no melody" → positive instruction "low rhythmic plucks" (V2), "warm stride" → "confident bounce" (V3), "morning" → "bright" to avoid birdsong trigger (V3).
- **Shamisen + harp fusion** is unusual for Suno. Expect 1-2 rerolls per variation for clean instrument separation. The "separated instruments" and "clean mix" production keywords are critical — do not remove them.
- **W:35% SI:75%** across all three keeps Suno tight to Jazz genre conventions while allowing slight creative breathing room. Good for a consistent set where all tracks should feel like they belong together.
- Track names generated via tri-model pipeline (MiniMax + Gemini + Grok, 18 per variation). Curated to 8 per variation using 6+ naming strategies, avoiding AI-slop fatigue words.
