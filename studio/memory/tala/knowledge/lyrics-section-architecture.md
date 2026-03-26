---
name: lyrics-section-architecture
description: Section ordering rules by genre and track type, Hook vs Chorus distinction, syllable density limits, duration templates.
type: knowledge
agent: tala
tags: [suno, lyrics, structure, sections, hook, chorus, verse, bgm, vocal, genre-structure, syllable-density]
---

# Lyrics Section Architecture

## 1. Genre Determines Section Structure
Never default to Intro→Build→Hook for everything. Different track types use fundamentally different section patterns.

- **Hook ≠ Chorus.** Hook = short repeated instrumental phrase (2-4 bars), used in hip-hop, lo-fi, melodic hip-hop. Chorus = full sung emotional peak section, used in pop, rock, R&B, country, jazz, folk. Using the wrong one confuses Suno.
- **Pre-Chorus is genre-specific.** Pop, rock, R&B need it. Jazz, folk, hip-hop, lo-fi, ambient skip it.
- **Bridge vs Breakdown.** Bridge = perspective shift/volta (pop, rock, country). Breakdown = energy drop before rebuild (metal, EDM, rock).

## 2. Section Patterns by Track Type

| Track Type | Pattern | Total Sections |
|---|---|---|
| Vocal — Pop/Rock/R&B | Intro → V → PC → C → V → PC → C → Bridge → Final C → Outro | 8-10 |
| Vocal — Jazz | Intro → V → C → V → C → Bridge → C → Outro | 5-7 |
| Vocal — Folk/Country | Intro → V → V → C → V → C → Bridge → C → Outro | 6-8 |
| Vocal — Hip-Hop/Rap | V → Hook → V → Hook → V → Hook | 7 |
| Vocal — Melodic Hip-Hop | V → Hook → V → Hook → Bridge → Hook | 6-8 |
| Vocal — Lo-fi (with vocals) | V → Hook → V → Hook → Outro | 4-5 |
| Vocal — Metal | Intro → V → C → V → C → Breakdown → C → Outro | 7-9 |
| Instrumental — BGM/Lo-fi | Intro → Hook → Break → Hook → Sustained | max 5 |
| Instrumental — Ambient/Sleep | Intro → Ambient → Sustained → Outro → Fade Out | 3-5 |
| Instrumental — Cinematic | Theme → Var 1 → Var 2 → Theme Reprise | 4-6 |

## 3. Duration Controls Section Count

| Duration | Strategy |
|---|---|
| Short (~1-2 min) | Intro → Verse → Hook → Outro (3-4 sections) |
| Standard (~2-3 min) | Intro → Verse → Chorus → Verse → Chorus → Outro |
| Extended (~3-4 min) | Add Bridge, second Hook, Instrumental Break |
| Long (~4+ min) | Multiple verses, Bridge, Solo, `[Sustained]`, or Extend |

## 4. Syllable Density Per Genre
Suno interprets dense text as rap even with non-rap Style tags.

| Genre | Syl/Line | Lines/Verse | Why |
|---|---|---|---|
| Ambient | 3-5 | 2-3 | Space > words |
| Lo-fi | 4-7 | 4 | Sparse, murmured |
| Jazz | 5-7 (max 10) | 4 (NEVER 8+) | Dense text → rap |
| Folk/Reggae | 5-8 | 4-6 | Conversational |
| Pop/Rock | 7-10 | 4-6 | Hook-forward |
| R&B/Soul | 7-10 | 4-6 | Melisma needs vowel space |
| Melodic Hip-Hop | 8-10 | 6-8 | Rhythmic verses |
| Rap/Hip-Hop | 8-12 | 8-16 | Stumbles at 14-16 |
| Metal | 6-10 | 4-8 | Consonant attacks |

**Reference:** `genre-structure.md` has full per-genre guidance with rhyme approach and structure details.
