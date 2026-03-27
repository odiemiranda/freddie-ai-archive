# Morning Coffee Jazz

> Generated from: "jazz lazy morning coffee, warm shamisen, single pluck, warm harp, also most silent vinyl cracks, for focus work and reading. mood: lazy, expectant, getting ready"
> Generated At: 2026-03-27 09:39:46 PM UTC+08:00

---

## Original Prompt
> jazz lazy morning coffee, warm shamisen, single pluck, warm harp, also most silent vinyl cracks, for focus work and reading. mood: lazy, expectant, getting ready

## Processed Configuration
| Field | Value |
|-------|-------|
| Genre | Lo-fi jazz |
| Instruments | Lead: Shamisen, single melodic plucks, low-pass filtered. Support: Harp, low-register pad chords, no melody. Rhythm: None (ambient focus) |
| Key | A minor (V1, V3), G major (V2) |
| BPM | 78-88 range |
| Mood | laid-back, unhurried, anticipatory warmth, gentle forward motion, slow awakening |
| Texture | Vinyl crackle (almost silent, very subtle) |
| Chord Progression | Per variation |
| Exclude | vocals, singing, electronic, drums, percussion |
| Music Cards | None matched |

---

## Variations

| # | File | BPM | Chords | W | SI |
|---|------|-----|--------|---|-----|
| 1 | [Shamisen-Forward](./v1-shamisen-forward.md) | 82 | — | 35% | 70% |
| 2 | [Harp-Forward](./v2-harp-forward.md) | 88 | — | 40% | 65% |
| 3 | [Ambient Drift](./v3-ambient-drift.md) | 78 | — | 45% | 60% |

---

## Variation Summary

1. **Shamisen-Forward** -- Shamisen single plucks lead throughout, harp sits low underneath as harmonic bed, minimal vinyl texture. The most "lazy morning" of the three -- unhurried single notes with space between. 82 BPM, A minor.
2. **Harp-Forward** -- Harp takes more melodic presence with arpeggios, shamisen punctuates with sparse plucks. Captures the "expectant, getting ready" mood -- slightly brighter, more forward motion. 88 BPM, G major.
3. **Ambient Drift** -- Both instruments equally sparse, maximum space between notes, most texture-forward with vinyl crackle as a near-equal presence. The slowest and most meditative -- deep reading focus. 78 BPM, A minor.

---

## Production Notes

- **Frequency map**: Shamisen sits high-mid (4-8 kHz), harp low-mid (250 Hz-1 kHz low arpeggios). No collision -- clean separation without EQ needed. Vinyl crackle occupies high/air (8 kHz+), no competition with either instrument.
- **No drums decision**: User requested focus/reading music. All three variations exclude drums and percussion entirely -- rhythm comes from instrument pacing and space between notes. This is the defining production choice of the folder.
- **Vibe translations**: "lazy" translated to "laid-back, unhurried" (Style) and "gentle, unhurried" (Lyrics) -- raw "lazy" causes Suno to meander aimlessly. "expectant" translated to "anticipatory warmth, gentle forward motion" -- raw "expectant" has no production meaning.
- **Frequency darkening**: Shamisen is naturally bright (4-8 kHz). V1 uses "warm low-pass filtered" to darken it for lo-fi context. V3 uses "sparse plucks" to reduce its presence. V2 lets shamisen stay brighter as punctuation against the warmer harp lead.
- **BPM spread**: 78-82-88 gives playlist sequencing options. 78 (V3) for deep focus, 82 (V1) for casual reading, 88 (V2) for getting-ready energy. All within lo-fi sweet spot (75-95).
- **Tri-critic summary**: All 3 variations ran full Gemini + MiniMax + Grok pipeline. Common accepted flags: tighten vinyl texture language, add "sparse" qualifiers, simplify genre label. Common rejected flags: suggestions to add drums, increase instrument count, expand Style block beyond lean target.
- **Tips**: If Suno adds vocals despite triple reinforcement ([Instrumental] + [No Vocals] + [End] + Exclude), lower Style Influence to 40-50%. If shamisen sounds too bright/sharp, add "muffled" or "low-pass" to the [Instrument:] tag. If vinyl crackle is too loud, change "steady" to "barely audible" in the [Texture:] tag.
- **All 3 approved on first presentation** -- no revision rounds needed.
