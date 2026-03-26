---
name: Suno clean generation patterns
description: Six factors for clean Suno output, vocal range control, and phonetic accent technique — learned from Morning Market comparison and AI with TechNap research
type: feedback
agent: freddie
tags: [suno, clean, generation, muddled, quality, vocal, range, phonetic, accent, six-factor]
---

When Suno generates muddled output, check these six factors (ranked by impact):

1. **Structure tags in Lyrics block** — put [Intro] [Build] [Hook] in the Lyrics block, NOT the Style block. Style is sonic palette only (genre, instruments, mood, BPM, key, texture)
2. **Broad Exclude list** — 10-12 items, ban every instrument not explicitly wanted
3. **Mood contrast** — don't stack same-bucket descriptors (foggy+intimate+muted). Use contrast pairs from different categories
4. **Texture budget** — max 2 terms, no overlaps (tape warmth + analog warmth = redundant)
5. **Directive verbs** — "leads", "plucks" in Style block, not "floating", "weaving"
6. **Genre clarity** — "Reggae fusion" > "Lo-fi jazzhop" (give Suno a rhythm template)

**Why:** Echo/Gemini analysis comparing the clean "Morning Market" track against muddled "tavern morning" tracks identified these as the compound cause. Fixing all six pushed generation quality from ~60% clean to consistently clean.

**How to apply:** When reviewing any Suno track that generates muddled output, run through this checklist before adjusting W/SI or rebuilding from scratch.

## Vocal range control (added 2026-03-24)

For vocal tracks, adjectives like "deep" are opinions — note ranges are facts. Use a three-line header at the top of the Lyrics box:
```
[Voice description + texture]
[Vocal range: E2-E4]
[Exclude: falsetto, soprano register, head voice, bright tenor]
```

Key presets: Bass-baritone E2-E4, Standard baritone G2-G4, Tenor C3-C5, Contralto E3-E5, Soprano C4-C6. Without the range header + exclusion list, Suno drifts to tenor-pop on choruses (register drift).

**Why:** YouTube research (AI with TechNap) demonstrated that note-range headers + exclusion lists produce consistent vocal register where adjectives fail. Added full range map (C2-C6), 13 presets, advanced techniques (proximity effect, drone, sub-bass narrator) to `vocals.md`. Tala's agent updated to mandate range headers on vocal tracks.

**How to apply:** Tala checks vocals.md Range Presets during step 8b (vocal track production rules) and writes the header in step 8e before any section tags.

## Phonetic accent control (added 2026-03-24)

Suno's dictionary normalizes all words to clean American English. For authentic regional/character voices, use triple reinforcement:
1. Style box — genre + accent + vocal character
2. Metadata header — voice directive in Lyrics box
3. Phonetic lyrics — misspelled words forcing target pronunciation

Spelling also controls note duration (stretched vowels = longer notes) and rhythmic attack (hard consonants = punch). Use ChatGPT as phonetic translator. Accent recipes for Scottish, Jamaican, Southern, Irish, Cockney in `vocals.md → Accent Recipes`.

**Why:** YouTube research (AI with TechNap) showed that Suno ignores accent adjectives in Style box alone. Phonetic misspelling forces manual sound rendering instead of dictionary lookup. Combined with range headers, this gives full vocal director control (pitch + pronunciation).

**How to apply:** Tala checks for accent/regional requests in vocal track production rules and applies triple reinforcement. Rune writes standard lyrics first; phonetic translation is a separate step via ChatGPT. Both agents have memories.

## Extreme vocal capability map (added 2026-03-24)

Suno can simulate physical vocal mechanics (microtonal slides, vocal cord flips, two-tone overtones, whistle register). All tested at W:10% SI:85%.

**Works:** Carnatic gamaka, opera/bel canto, yodeling, jazz scat, whistle register, overtone/throat singing.
**Fails:** Non-melodic vocalization (haka, war chants, drill sergeant). Suno has a hard melodic bias — it will always add a tune.

**Why:** AI with TechNap tested 7 extreme techniques. Suno models the physical behavior (cord flip, diaphragm push, overtone harmonics), not just the sound. But it cannot suppress its melodic training for pure rhythmic shouting.

**How to apply:** Check `vocals.md → Extreme Vocal Techniques` before attempting non-standard vocal styles. Warn user about melodic bias if they request pure non-melodic vocals. Combine with range headers for precision (throat singing → C2-E3, whistle → C5-C6).
