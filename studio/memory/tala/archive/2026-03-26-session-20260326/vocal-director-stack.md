---
name: vocal-director-stack
description: Advanced techniques for vocal range control, phonetic accents, and extreme mechanics.
type: knowledge
agent: tala
tags: [suno, vocals, range-headers, phonetics, extreme-vocals]
---

# The Vocal Director Stack

To achieve consistent, authentic vocals, use **Triple Reinforcement**. All three components must agree or Suno will default to generic American pop-tenor.

## 1. Range Headers (Mandatory)
Place a three-line header at the very top of the Lyrics box:
```text
[Voice description + texture]
[Vocal range: E2-E4]
[Exclude: falsetto, soprano register, head voice, bright tenor]
```
- **Why:** Prevents "register drift" where Suno moves to a higher, thinner voice during choruses.
- **Common Presets:** Bass-baritone (E2-E4), Tenor (C3-C5), Contralto (E3-E5), Soprano (C4-C6).

## 2. Phonetic Control (Accent & Character)
Break Suno's dictionary by deliberately misspelling words.
- **Spelling as Notation:** `na-ame` (stretched vowels/longer notes), `Da wata` (hard consonants/rhythmic punch), `walkin'` (dropped endings/regional feel).
- **Workflow:** Write standard lyrics -> Translate phonetically for target accent (Scottish, Jamaican, Cockney, etc.) -> Paste with corresponding Style box tags.

## 3. Extreme Vocal Mechanics
Suno can handle microtonal slides and complex ornaments but has a hard melodic bias (it cannot do pure non-melodic chanting/Haka).
- **Tested Techniques:** Carnatic (gamaka), Opera (wide vibrato), Yodeling (chest-to-falsetto flip), Overtone/Throat singing.
- **Settings:** Use **Low Weirdness (W:5-15%)** and **High Style Influence (SI:80-90%)**. High weirdness breaks the technical authenticity of these styles.

**Consolidated from:** 2026-03-24-extreme-vocal-capability.md, 2026-03-24-phonetic-vocal-control.md, 2026-03-24-vocal-range-control.md
