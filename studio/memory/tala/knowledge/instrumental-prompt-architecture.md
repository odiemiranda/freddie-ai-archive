---
name: instrumental-prompt-architecture
description: Master rules and advanced techniques for instrumental and background music (BGM) prompt engineering.
type: knowledge
agent: tala
tags: [suno, prompt-design, instrumental, bgm, instrument-interaction, exclusion-style, syntax, style-block-formula]
---

# Instrumental Prompt Architecture

## 1. Style Block Formula (11-Slot Concept)
The Style block defines *what* it is and *how* it acts. Suno front-loads importance (genre first, instruments second, production last). The slot order is critical. Never include general structure tags (`[Intro]`) or staging ("enters") here.

- **Mandatory Tags for Instrumental/BGM:**
    - `[Instrumental]` (must open the Style block for instrumental tracks).
    - `[Background Music]` (must be included in Style for BGM tracks to ensure restrained arrangement).
- **Critical Production Keyword:** `separated instruments` is the single most important production keyword. Without it, Suno layers instruments on top of each other, leading to chaos.
- **Character Budget:** Target ~200-250 chars for instrumental Style blocks, hard max 350. First 20-30 words carry the most weight — genre and instruments must come first. When over budget, cut chords → textures → mood (keep 1). Never cut genre, lead instrument, key/BPM, or production keywords.
- **Sonic Palette:** Genre, BPM (88-100 BPM is optimal for plucked instrument separation), Key. Texture (e.g., "tape warmth") is LOW priority — skip if budget tight. Suno infers warmth from genre.
- **Instrument Behavior:** Define roles clearly (e.g., "shamisen melodic phrases", "harp low-register pad chords").
- **Dynamic Control:** Use "relaxed", "unhurried", "spacious", or "loop-friendly." Avoid "subtle melodic movement" or "consistent dynamics"—these cause aimless "noodling."

## 2. Instrument Hierarchy & Interaction
Clear roles and explicit constraints are vital for instrument separation and clean mixes.

- **Lead Continuity:** One instrument must be designated as the Lead in every section.
- **Support Subordination (Critical):** Support instruments must be explicitly demoted and constrained in both Style and Lyrics blocks.
    - **Why:** Prevents support instruments from drifting into melodic territory and competing with the lead.
    - **How to apply:** Use phrases like "harmonic bed only," "no melody," "underneath," "low-register pad chords," "no high notes." Keep constraints pure — don't mix in texture terms ("warmth") or groove terms ("laid-back swing"). Those belong in their own slots (8-9).
    - **`rolled-off highs`:** This keyword helps keep support instruments in their designated frequency lane, preventing them from climbing into the lead instrument's register.
- **Interaction Verbs:** Use "trades phrases" or "breathing together." Avoid "duet" or "trading phrases" for instruments in the same register (causes clashing).

## 3. Instrumental Lyrics Syntax
Control the arrangement by using the Lyrics block as a director's script.

- **Mandatory Brackets:** Every line MUST be wrapped in `[brackets]`.
- **Mandatory Anchors:**
    - Start with `[Instrumental]` as the very first line.
    - End with `[No Vocals] [End]` (double reinforcement to prevent vocal elements and ensure proper fade).
- **Tag Length:** 3-10 words. Longer tags risk being sung; shorter tags may be ignored.
- **No Musical Notation:** Never use note names (D4, F#4), rhythmic values (quarter notes), or chord voicings in Lyrics brackets. Suno treats them as noise. Chord names work ONLY in the Style block.
- **BGM Section Limits:** Max 5 sections for BGM tracks (e.g., Intro→Hook→Break→Hook→Sustained). More sections force Suno to differentiate each one by adding layers, leading to chaos. Hook = instrumental melodic loop (not Chorus). Break = breathing room. Sustained = tells Suno to maintain energy for clean loop/extend.
- **Closing Anchors:** Always end with `[No Vocals] [End]` — double reinforcement prevents vocal bleed and forces clean termination. `[Background Music]` goes in the **Style block only**, not in Lyrics closing tags.
- **Avoid Intensity Language:** Do not use "rise" or intensity language (e.g., "arpeggios rise," "phrases slightly longer") in Build sections. Suno interprets these as instructions to add density/layers.
- **Groove Consistency:** "Groove stays even and consistent" helps prevent Suno from evolving the rhythm unexpectedly.
- **Stereo Panning:** Stereo panning instructions (e.g., "harp left channel shamisen right") are ignored. Remove them.

## 4. Exclusions
Aggressive exclusion is critical for clean instrumental tracks.

- **Why:** Suno's default behavior is to add layers. Explicitly blocking unwanted instruments prevents frequency fighting and maintains clarity.
- **How to apply:** Default to **20+ exclude items** for instrumental tracks. Include "vocals, singing" plus any instrument in the same frequency register as the lead that isn't explicitly named in the Style block.

**Consolidated from:** instrumental-prompt-engineering.md, 20260326-instrumental-bgm-rules.md, 20260326-style-block-formula.md
