# Shamisen Jazz Morning

> Generated from: Jazz fusion instrumental BGM with shamisen lead, harp support, brushed drums. Relaxed morning tavern feel, loop-safe for background listening.
> Generated At: 2026-03-26 08:41:45 PM UTC+08:00

---

## Original Prompt
> Jazz fusion instrumental BGM — shamisen lead, harp low-register pad chords, brushed drums. Relaxed, laid-back, contemplative, cozy, morning warmth. D Major, 85-92 BPM. Loop-safe, even dynamics, no bright hooks. Background music for focus/atmosphere.

## Processed Configuration
| Field | Value |
|-------|-------|
| Genre | Jazz fusion |
| Instruments | Lead: shamisen melodic phrases (70%). Support: harp low-register pad chords, no melody (30%). Rhythm: brushed drums sparse, no fills |
| Key | D Major |
| BPM | 85-92 |
| Mood | relaxed, laid-back, contemplative, cozy, morning warmth |
| Texture | V1: none. V2: rolled-off highs. V3: tape warmth |
| Chord Progression | Per variation |
| Exclude | vocals, singing, piano, synth, guitar, bass guitar, violin, cello, strings, flute, brass, horn, trumpet, saxophone, organ, electric, bells, chimes, whistling, humming |
| Music Cards | None matched |

---

## Variations

| # | File | BPM | Chords | W | SI |
|---|------|-----|--------|---|-----|
| 1 | [Morning Tavern](./morning-tavern.md) | 88 | — | 30% | 75% |
| 2 | [Window Reading](./window-reading.md) | 86 | — | 30% | 75% |
| 3 | [Cozy Tavern](./cozy-tavern.md) | 90 | — | 30% | 75% |

---

## Variation Summary

1. **Morning Tavern** — Classic jazz swing, no texture additions. Lean prompt (~220 chars). Shamisen carries everything, harp barely there.
2. **Window Reading** — Warm and muffled for deep focus. Rolled-off highs darken the shamisen's brightness. Study/reading mode.
3. **Cozy Tavern** — Harp slightly more present with mid-low arpeggios. Tape warmth texture adds vintage character. The warmest of the three.

---

## Production Notes

- All three variations use identical exclude lists (20+ items) to prevent instrument hallucination
- Shamisen sits 4-8 kHz, harp constrained to 250 Hz-1 kHz (V3: mid-low 250 Hz-1 kHz arpeggios), brushed drums fill 2-8 kHz texture — no frequency collision
- [Sustained] ending on all variations for loop safety
- W:30% SI:75% across the board — tight to genre, minimal creative risk for consistent BGM output
- BPM spread: 86-90 range keeps all three compatible for playlist sequencing
- Tri-critic ran on V2 and V3 (Gemini + MiniMax + Grok). Key accepted fixes:
  - V2: "muffled" replaced with "soft" in lyrics (dangerous vibe word), added directive verb "leads with" to shamisen in Style
  - V3: "cozy" replaced with "warm intimate" in Style (vibe risk), "bright phrase" changed to "quick melodic phrase", "drift in" changed to "enters softly" (passive verb), interlude clarified for shamisen dominance over harp
- If V2's rolled-off highs over-darken the mix, try "gentle high-end taper" instead
- If V3's tape warmth introduces hiss, try "analog warmth" instead
- All critics praised the behavior-focused bracket tags and comprehensive exclude list. Gemini consistently wanted bare structure tags in Lyrics — rejected per agent rules (instrumental tracks need bracketed direction on every line)
