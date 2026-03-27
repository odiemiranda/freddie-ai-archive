---
title: Syllable, Rhythm & Emotional Arc Reference
tags: [lyrics, syllable, rhythm, tempo, genre-density, emotional-arc, repetition, minimax, cross-platform]
---

# Syllable, Rhythm & Emotional Arc Reference

Genre density targets, tempo-based line lengths, repetition strategy, emotional arc frameworks, and cross-platform tag mapping.

## Emotional Arc Framework

Before writing, establish the journey:

| Arc Type | Movement | Best For |
|----------|----------|----------|
| **Rising** | Quiet → building → peak | Anthems, workout, uplifting |
| **Falling** | Intense → softening → resolve | Ballads, lullabies, meditation |
| **Wave** | Rise → fall → rise → resolve | Pop songs, storytelling |
| **Flat/sustained** | Consistent energy | Lo-fi, ambient, study music |
| **Contrast** | Soft verse → explosive chorus | Rock, EDM, dramatic |
| **Spiral** | Returns to theme with new meaning | Art songs, concept tracks |

### Questions to Ask

1. What's the emotional starting point?
2. Where should the peak be?
3. How should it end?
4. Is there a story or just a feeling?
5. Which platform? (default: Suno AI)

## Genre Quick Reference

| Genre | Lines/Verse | Syl/Line | Density | Rhyme | Key Rule |
|-------|-------------|----------|---------|-------|----------|
| Ambient | 2-3 | 3-5 | Sparse | Assonance | Words are texture, not narrative |
| Folk | 4-6 | 6-8 | Moderate | ABAB/ABCB | Story advances each verse |
| Reggae | 4 | 5-7 | Moderate | Slant/AABB | Offbeat phrasing, open syllables |
| Jazz | 4 | 5-7 | Low-Mod | Loose/slant | Space for melody — NEVER 8+ lines |
| Pop | 4-6 | 7-9 | Moderate | ABAB/AABB | Hook-forward, singable after 2 listens |
| Rock | 4-6 | 7-10 | Mod-High | ABAB/AABB | Power chorus, verse builds tension |
| R&B/Soul | 4-6 | 7-10 | Moderate | Slant+internal | Space for vocal runs and melisma |
| Melodic Hip-Hop | 6-8 | 8-10 | High | End+internal | Sung hooks, rhythmic verses |
| Rap | 8-16 | 8-12 | Very High | Stacked multi | Dense, not singable — rapped |
| Lo-fi | 4 | 4-7 | Low | Slant | Less text = more atmosphere |
| Metal | 4-8 | 6-10 | High | ABAB/none | Short phrases for screams, melodic for cleans |

**Density spectrum** (least → most): Ambient → Folk → Reggae → Lo-fi → Jazz → Indie → Pop → Rock → R&B → Melodic Hip-Hop → Rap

**Fusion rule**: When two genres are specified, at least one variation must lean toward the less-dense genre.

**Suno limit**: Max 10-12 syl/line even for rap. Dense lyrics + sparse chorus = best contrast.

## By Tempo

| BPM Range | Line Length | Character |
|-----------|-------------|-----------|
| 40-60 (sleep) | 3-5 syllables | Sparse, breathing room |
| 60-80 (ballad) | 5-8 syllables | Flowing, emotional weight |
| 80-100 (lo-fi) | 4-7 syllables | Rhythmic but unhurried |
| 100-130 (pop) | 6-10 syllables | Energetic, melodic |
| 130-150+ (EDM) | 4-6 syllables | Punchy, repetitive |

## Repetition Strategy

- **Hooks**: Repeat core phrase 2-3 times in chorus
- **Callbacks**: Echo a verse phrase in the bridge with changed meaning
- **Rhythmic anchors**: Repeat a rhythmic pattern with different words
- **AI benefit**: Repetition helps models lock onto melodic patterns

## Line-Writing Principles

1. **One image per line** — don't cram
2. **End strong** — last word of each line carries weight
3. **Vary line length** — monotonous meter sounds robotic
4. **Use concrete nouns** — "the third cup of coffee" not "the recurring beverage"
5. **Break expected patterns** — surprise in the bridge or final chorus
6. **Read it aloud** — if it's hard to say, it's hard to sing

## MiniMax Music Lyrics

### 14 Structure Tags (all lowercase)
`[intro]`, `[verse]`, `[pre-chorus]`, `[chorus]`, `[hook]`, `[drop]`, `[bridge]`, `[solo]`, `[build-up]`, `[instrumental]`, `[breakdown]`, `[break]`, `[interlude]`, `[outro]`

### Vocal Hum & Chanting
For near-instrumental feel, use phonetic syllables:
- `ah, ah, ah, ah...` — Open, emotional
- `la, la, la, la...` — Light, playful
- `mmm, mmm, mmm...` — Warm, intimate
- `ooh, ooh, ooh...` — Soulful, yearning

### Guidelines
- Structure tags = paragraph-level arc control
- Longer lyrics = longer tracks. Max 3,500 characters
- Empty lyrics = auto-generated from Style
- MiniMax is vocal-focused — write for a singer

## Cross-Platform Tag Mapping

| Concept | Suno | MiniMax |
|---------|------|---------|
| Opening | `[Short Instrumental Intro]` (90%) | `[intro]` |
| Story section | `[Verse]` (90%) | `[verse]` |
| Tension build | `[Pre-Chorus]` (65-70%) | `[pre-chorus]` |
| Main hook | `[Chorus]` (95%) | `[chorus]` |
| Catchy phrase | `[Catchy Hook]` (90%) | `[hook]` |
| Energy release | `[Drop]` | `[drop]` |
| Contrast | `[Bridge]` (85%) | `[bridge]` |
| Rising energy | `[Build]` | `[build-up]` |
| Pure instruments | `[Instrumental]` (75%) | `[instrumental]` |
| Strip back | `[Breakdown]` | `[breakdown]` |
| Breathing room | `[melodic interlude]` (85%) | `[interlude]` |
| Closing | `[Big Finish]` (85%) | `[outro]` |
