---
name: instrumental-prompt-engineering
description: Master rules for clean, high-fidelity instrumental prompts and instrument hierarchy.
type: knowledge
agent: tala
tags: [suno, prompt-design, instrument-interaction, exclusion-style, syntax]
---

# Instrumental Prompt Engineering

## 1. Complexity Limits
Fewer ingredients = clearer signal. 
- **Max 2 genres:** Use rhythm-clear genres (e.g., "Jazz fusion", "Reggae fusion"). Avoid vague labels (e.g., "Groove pop").
- **Max 3 instruments:** Overlapping registers cause frequency fighting.
- **Max 1 rhythm source:** Usually drums or a driving bassline.

## 2. Style Block = Palette + Behavior + Dynamics
The Style block defines *what* it is and *how* it acts. Never include structure tags (`[Intro]`) or staging ("enters") here.
- **Sonic Palette:** Genre, BPM (88-100 BPM is optimal for plucked instrument separation), Key, Texture (e.g., "tape warmth", "rolled-off highs").
- **Instrument Behavior:** Define roles clearly (e.g., "shamisen melodic phrases", "harp low-register pad chords").
- **Dynamic Control:** Use "relaxed", "unhurried", "spacious", or "loop-friendly". 
- **Anti-pattern:** Avoid "subtle melodic movement" or "consistent dynamics"—these cause aimless "noodling."

## 3. Instrument Hierarchy & Neutral Labels
- **Neutral Labels (Critical):** Use "harp" instead of "Celtic harp." Cultural labels trigger unrequested genre-associated instruments (violin, fiddle) even with excludes. Describe sound with adjectives instead.
- **Lead Continuity:** One instrument must be designated as the Lead in every section. 
- **Support Subordination:** Support instruments must be explicitly demoted in both Style and Lyrics blocks (e.g., "harmonic bed only", "no melody of its own", "underneath").
- **Interaction Verbs:** Use "trades phrases" or "breathing together." Avoid "duet" or "trading phrases" for instruments in the same register (causes clashing).

## 4. Instrumental Lyrics Syntax
Control the arrangement by using the Lyrics block as a director's script.
- **Mandatory Brackets:** Every line MUST be wrapped in `[brackets]`.
- **Mandatory Anchors:** Start with `[Instrumental]` and end with `[No Vocals] [End]`.
- **Tag Length:** 3-10 words. Longer tags risk being sung; shorter tags may be ignored.
- **No Musical Notation:** Never use note names (D4, F#4), rhythmic values (quarter notes), or chord voicings in Lyrics brackets. Suno treats them as noise. Chord names work ONLY in the Style block.

## 5. Exclusions
Default to **12-18 exclude items** for instrumental tracks. Include "vocals, singing" plus any instrument in the same frequency register as the lead that isn't named in the Style block.

**Consolidated from:** prompt-architecture-instrumental.md, 2026-03-25-morning-window-groove-testing.md, 2026-03-25-musical-notation-in-brackets.md, 2026-03-25-validated-formula-confirmed.md
