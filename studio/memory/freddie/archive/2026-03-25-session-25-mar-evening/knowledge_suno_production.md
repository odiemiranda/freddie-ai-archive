---
name: Suno Production Standards
description: Advanced techniques for clean generation, vocal precision, and audio reference integration
type: knowledge
agent: freddie
tags: [suno, production, vocals, accents, music-cards, audio-analysis]
---

## Seven Factors for Clean Generation
If output is muddled, adjust these in order:
1. **Style block completeness:** Style needs sonic palette + instrument behavior + dynamic control. Bare instrument names cause Suno to guess. Include behavior hints ("shamisen melodic phrases", "harp low-register pad chords as background warmth"), chord progressions, and dynamic keywords ("relaxed", "unhurried", "loop-friendly"). But NO structure tags — those go in Lyrics.
2. **Support instrument hierarchy:** Subordinate support instruments in BOTH Style ("as background warmth") and Lyrics brackets ("far behind", "no melody of its own"). Equal presence = competitive chaos.
3. **Exclude List:** Use 10-12+ items. Use neutral instrument labels ("harp" not "celtic harp") to prevent genre-association bleed.
4. **Mood Contrast:** Avoid redundant descriptors (e.g., don't use both "foggy" and "muted").
5. **Texture Budget:** Max 2 terms (e.g., "tape warmth" OR "analog warmth", not both).
6. **Directive Verbs:** Use "leads" or "plucks" in Style; avoid vague terms like "weaving".
7. **Genre Clarity:** Use specific rhythm templates ("Reggae fusion", "Jazz fusion with laid-back swing") over vague labels ("Groove pop", "Lo-fi jazzhop").
8. **No musical notation in Lyrics:** Note names (D4), rhythmic values (quarter notes), chord voicings in bracket tags — all tested, all degrade output. Chord names work in Style block only. Descriptive language is the ceiling for Lyrics brackets.

## Vocal & Accent Control
- **Vocal Range:** Mandate a range header in the Lyrics box (e.g., `[Vocal range: E2-E4]`) and an `Exclude` list for registers (e.g., `Exclude: falsetto`). Use `vocals.md` for presets.
- **Phonetic Accents:** Use "Triple Reinforcement":
    1. Style box (accent/character).
    2. Lyrics header (voice directive).
    3. Phonetic spelling (e.g., "watah" for "water") to force pronunciation.
- **Extreme Techniques:** Suno can model physical mechanics (yodeling, throat singing, whistle register) but has a **hard melodic bias**. It cannot do pure rhythmic shouting/chanting.

## Music Cards (Audio References)
Ground creative prompts in `vault/studio/references/shared/music-cards/`.
- **Pipeline:** `musiccard.ts` downloads/trims YouTube audio, `analyze-audio.py` extracts real BPM/Key/Energy, and Gemini refines the metadata.
- **Usage:** Agents should load cards to use measured BPM/Key as hard constraints instead of descriptive guesses.

**Why:** Descriptive tags are imprecise. These systems force Suno to model specific physical/mathematical data for consistent, high-quality results.

**How to apply:** Tala must write range headers. Rune should provide phonetic translations via ChatGPT for specific accents. Use `npm run musiccard` to add new references.

*Consolidated from: feedback_suno_clean_generation.md, project_music_cards.md*
