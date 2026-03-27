# Shamisen Jazz Tavern Morning

> Generated from: "Shamisen with Jazz melodies fusion with light harp. Mood: Morning coffee by the tavern window, focus work, focus reading, background music"
> Generated At: 2026-03-26 09:21:14 PM PDT

---

## Original Prompt
> Shamisen with Jazz melodies fusion with light harp. Mood: Morning coffee by the tavern window, focus work, focus reading, background music

## Processed Configuration
| Field | Value |
|-------|-------|
| Genre | Jazz |
| Instruments | Lead: shamisen. Support: harp. Rhythm: brushed drums |
| Key | G Major |
| BPM | 85-90 |
| Mood | warm |
| Texture | None (genre implies warmth) |
| Chord Progression | Dropped — invites piano hallucination |
| Exclude | 14 items (focused instrument hallucination prevention) |
| Music Cards | None matched |

---

## Variations

| # | File | BPM | Angle | W | SI |
|---|------|-----|-------|---|-----|
| 1 | [Steam & Silk](./steam-and-silk.md) | 88 | Shamisen-forward classic | 25% | 85% |
| 2 | [Low-Register Lull](./low-register-lull.md) | 90 | Harp-forward, low arpeggios | 25% | 85% |
| 3 | [Space Between Notes](./space-between-notes.md) | 85 | Minimal zen, sparse everything | 25% | 85% |

---

## Variation Summary

1. **Steam & Silk** — Shamisen leads every verse with harp piped underneath. Bridge is shamisen solo. Classic jazz BGM feel. Tested through 8+ iterations to find the right bracket syntax.
2. **Low-Register Lull** — Harp owns the intro and bridge with solo low arpeggios. In verses, shamisen leads with harp piped underneath. Gives harp dedicated space instead of shared space. Consistent across 5+ Suno generations.
3. **Space Between Notes** — Everything says less. Sparse plucks, sustained chords, minimal drums. Silence/stillness cues in every section. Most background-friendly — designed to disappear into the room.

---

## Production Notes

- **Pipe syntax is critical.** All instruments in one bracket line per section (`[Verse | shamisen jazz phrases | harp underneath | brushed drums]`). Separate bracket lines cause layering/competition. This was the key discovery from V1 iteration.
- **Lean Style blocks win.** 145-161 chars outperformed 291 chars for the same track. Less info = less confusion for Suno.
- **Chord progressions invite piano.** Dropped from all 3 variations after testing showed Gmaj7-Em7-Cmaj9-D7 caused piano hallucination at the 75% mark.
- **14 focused excludes > 30 noisy ones.** Abstract excludes (aggressive, uptempo, crescendo, dramatic build) confuse more than help. Keep instrument-specific only.
- **"Jazz" not "Jazz fusion".** Simpler genre label = cleaner output.
- **Scene cues matter.** "morning air", "warm room", "stillness" give Suno atmosphere context that pure instrument instructions don't.
- **Support instrument gets solo sections, not shared dominance.** V2 works because harp owns intro/bridge outright instead of competing with shamisen in every section.
- **BPM spread (85/88/90):** V3 slowest for zen, V1 middle for classic, V2 fastest for arpeggio movement.
