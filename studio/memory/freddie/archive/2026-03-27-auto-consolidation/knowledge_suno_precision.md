---
name: Suno Precision Engineering
description: Rules for clean generation, audio-grounded prompting, and structural separation
type: knowledge
agent: freddie
tags: [suno, style-block, lyrics, music-cards, production, audio-analysis]
---

## The Style vs. Lyrics Split (Critical)
Muddled generation is usually caused by tag leakage.
- **Style Block:** Sonic palette + instrument behavior + dynamic control. Comma-separated keywords, not sentences. Includes instrument names with behavior constraints ("melodic phrases", "pad chords no melody"), key/BPM, mood (max 2), and production keywords ("clean mix, separated instruments"). Target ~200-250 chars instrumental, ~250-350 vocal. **No structure tags** — those go in Lyrics only.
- **Lyrics Block:** Structure, arrangement, and dynamics. Use `[Intro]`, `[Verse]`, `[Hook]`, `[Break]`, and bracketed director notes with spatial cues ("far behind", "low and distant"). For instrumental: every line in brackets, close with `[No Vocals] [End]`.

## Clean Generation Factors
1. **Neutral Labels:** Use "harp" not "celtic harp" to avoid genre-bleed in the Exclude list.
2. **Instrument Hierarchy:** Explicitly subordinate support instruments in Style with pure constraints ("low-register pad chords, no melody") and in Lyrics with spatial cues ("far behind", "harmonic bed only"). Don't mix texture terms ("warmth") or groove terms ("laid-back swing") into instrument constraint slots.
3. **Spacious Keywords:** Use "unhurried," "spacious," or "loop-friendly" to prevent density clashing.
4. **Vocal Control:** Mandate range headers in Lyrics (e.g., `[Vocal range: E2-E4]`). Use phonetic spelling for specific accents.

## Music Cards (Audio Grounding)
Ground all prompts in real-world data via `vault/studio/references/shared/music-cards/`.
- **Pipeline:** `tools/musiccard.ts` (YouTube download/trim) + `tools/analyze-audio.py` (BPM, Key, Energy, Spectral analysis).
- **Usage:** Use the measured BPM and Key as hard constraints in the Style block.

**Why:** Suno has a melodic bias and struggles with competitive instrument frequencies. Structural tags in the Style block confuse the model's "mood" setting.

**How to apply:** Tala must write range headers. Ensure all structure tags are removed from Style blocks. Use `npm run musiccard` to verify energy/BPM before prompting.

*Consolidated from: knowledge_suno_production.md, 2026-03-25-session-summary.md*
