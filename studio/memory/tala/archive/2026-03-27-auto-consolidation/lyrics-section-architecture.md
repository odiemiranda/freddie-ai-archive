---
name: lyrics-section-architecture
description: Section ordering rules by genre and track type, metatag reliability tiers, Hook vs Chorus distinction, syllable density limits, duration templates.
type: knowledge
agent: tala
tags: [suno, lyrics, structure, sections, hook, chorus, verse, bgm, vocal, genre-structure, syllable-density, metatag-reliability]
---

# Lyrics Section Architecture

## 1. Metatag Reliability Tiers
Not all section tags are equally reliable. Use high-reliability tags, avoid unreliable ones.

| Tier | Tags | Reliability |
|---|---|---|
| **High (85-95%)** | `[Chorus]` (95%), `[Verse]` (90%), `[Short Instrumental Intro]` (90%), `[Catchy Hook]` (90%), `[Big Finish]` (85%), `[Bridge]` (85%), `[melodic interlude]` (85%) | Prefer these |
| **Moderate (65-75%)** | `[Instrumental]` (75%), `[Pre-Chorus]` (65-70%), `[Post-Chorus]` (65%) | Use when genre requires |
| **Unreliable (20-45%)** | `[Intro]` (30-40%), `[Outro]` (40%), `[Fade Out]` (35%), `[Break]` (45%) | AVOID — use alternatives |

**Reliable replacements:**
- `[Intro]` → `[Short Instrumental Intro]` or `[Short Intro]`
- `[Outro]` → `[Big Finish]` or `[Final Chorus]`
- `[Fade Out]` → `[Big Finish]` or natural end
- `[Break]` → `[melodic interlude]`

**Mood tags (V5):** `[Mood: haunting]`, `[Mood: joyful]`, `[Mood: triumphant]`, `[Mood: somber]`, `[Mood: ethereal]` — 75-85% reliable. Max 1-2 per song.

## 2. Genre Determines Section Structure
- **Hook ≠ Chorus.** Hook = short repeated instrumental phrase (hip-hop, lo-fi). Chorus = full sung emotional peak (pop, rock, R&B, country, jazz, folk).
- **Pre-Chorus is genre-specific.** Pop, rock, R&B need it. Jazz, folk, hip-hop, lo-fi, ambient skip it.
- **Bridge vs Breakdown.** Bridge = perspective shift. Breakdown = energy drop before rebuild.
- **Max 5-7 structural tags per song.** More reduces reliability across all tags.
- **Piano ballad:** Do NOT add drums unless explicitly requested. Drums destroy intimate atmosphere.

## 3. Section Patterns by Track Type (Reliable Tags)

| Track Type | Pattern | Total Sections |
|---|---|---|
| Vocal — Pop/Rock/R&B | Short Intro → V → PC → C → V → PC → C → Bridge → Final Chorus → Big Finish | 7-10 |
| Vocal — Jazz | Short Intro → V → C → V → C → Bridge → C | 5-7 |
| Vocal — Hip-Hop/Rap | V → Catchy Hook → V → Catchy Hook → V → Catchy Hook → Big Finish | 7 |
| Vocal — Lo-fi | V → Catchy Hook → V → Catchy Hook | 4-5 |
| Vocal — Piano Ballad | Short Intro → V → C → V → C → Bridge → Final Chorus | 5-7 |
| Instrumental — BGM/Lo-fi | Short Instrumental Intro → Catchy Hook → melodic interlude → Catchy Hook → Sustained | max 5 |
| Instrumental — Ambient | Short Instrumental Intro → Sustained → melodic interlude → Sustained | 3-4 |

## 4. Audio Quality vs Length
Less information = cleaner audio. 500-1500 chars = 9.5/10 clarity. 3500-5000 chars = 8.0/10.

## 5. Syllable Density Per Genre
Suno interprets dense text as rap even with non-rap Style tags.

| Genre | Syl/Line | Lines/Verse |
|---|---|---|
| Ambient | 3-5 | 2-3 |
| Lo-fi | 4-7 | 4 |
| Jazz | 5-7 (max 10) | 4 (NEVER 8+) |
| Pop/Rock | 7-10 | 4-6 |
| Rap/Hip-Hop | 8-12 | 8-16 |

**Reference:** `genre-structure.md` for full per-genre guidance.
