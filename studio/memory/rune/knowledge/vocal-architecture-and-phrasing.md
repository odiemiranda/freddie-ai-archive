---
name: Vocal Performance and Phrasing
description: Rules for matching syllable density and word choice to vocal register and phonetic rendering.
type: knowledge
agent: rune
tags: [lyrics, suno, vocal-range, phonetics, phrasing, register, accents]
---

## Overview
Lyrics are the "stage directions" for the vocal engine. To produce professional results, Rune must write lyrics that account for both the physical limitations of different vocal registers and the technical requirements of phonetic translation for accents.

## Vocal Register Awareness
Phrasing rhythm should match the target register's natural delivery:
- **Low Registers (Bass/Baritone, E2-G4):**
  - **Word Choice:** Favor short, common words with long vowels (e.g., "oh", "home", "low", "stay"). 
  - **Avoid:** Complex consonant clusters (e.g., "strengths", "scratched") which can choke low voices.
  - **Phrasing:** Sparse phrasing with fewer words per line and more space between phrases.
- **High Registers (Tenor/Soprano, C3-C6):**
  - **Word Choice:** Open vowels (e.g., "ay", "ee", "oh") sustain well on high notes.
  - **Phrasing:** Can handle faster syllable density and longer melodic flows.

## Phonetic and Accent Preparation
When writing for regional or character voices (Scottish, Jamaican, etc.):
- **Standard First:** Write standard English lyrics first. The phonetic translation is a separate step (handled by Tala or external tools).
- **Vocal Clarity:** Favor words with strong vowels and clear consonants.
- **Cluster Avoidance:** Avoid complex consonant clusters that become unreadable when phonetically respelled.
- **Character Count:** Keep lines slightly shorter than usual, as phonetic spelling increases character count (approaching Suno's limits).

## Agent Interaction
- **Rune's Role:** Focus on the emotional architecture and word choice that supports the target register.
- **Tala's Role:** Handles the vocal production layer, including range headers and the actual phonetic translation.

## How to Apply
1. **Match Density to Register:** When Tala requests a "deep male voice," reduce syllable count and increase line breaks.
2. **Phonetic Efficiency:** Choose words that are "phonetic-friendly"—simple, vowel-forward, and rhythmically clear.
3. **Register-Weight Rule:** Match phrasing weight to voice weight. A bass voice sings heavy; a soprano floats.

**Consolidated from:**
- `2026-03-24-phonetic-lyrics-awareness.md`
- `2026-03-24-vocal-range-awareness.md`
