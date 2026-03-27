---
title: Suno AI Lyrics Platform Knowledge
tags: [lyrics, suno, platform, brackets, structure-tags, emotion-performance, vocal-direction]
---

# Suno AI Lyrics Platform Knowledge

Platform-specific rules for writing lyrics that Suno interprets correctly. Structure tags, typed brackets, emotion performance craft.

## Structure Tags (by reliability)

**High (85-95%) — prefer these:**
`[Short Instrumental Intro]` (90%), `[Verse]` (90%), `[Chorus]` (95%), `[Catchy Hook]` (90%), `[Bridge]` (85%), `[Big Finish]` (85%), `[melodic interlude]` (85%), `[Final Chorus]`

**Moderate (65-75%):**
`[Pre-Chorus]` (65-70%), `[Post-Chorus]` (65%), `[Instrumental]` (75%), `[Breakdown]`, `[Build]`, `[Drop]`, `[Sustained]`, `[End]`

**Unreliable — AVOID:**
`[Intro]` (30-40%) → `[Short Instrumental Intro]`, `[Outro]` (40%) → `[Big Finish]`, `[Break]` (45%) → `[melodic interlude]`

**Max 5-7 structural tags per song.** More reduces reliability. Mood tags (V5, 75-85%): `[Mood: haunting]`, max 1-2 per song.

## Vocal Direction Tags

`[Soft vocals]`, `[Whispered]`, `[Spoken word]`, `[Powerful vocals]`, `[Belting]`, `[Falsetto]`, `[Harmonies]`, `[A cappella]`, `[Call and response]`, `[Ad-libs]`, `[Humming]`, `[Vocalizing]`

## Dynamic Tags

`[Slowly building]`, `[Crescendo]`, `[Climax]`, `[Stripped back]`, `[Full band]`, `[Quiet]`, `[Explosive]`

## Typed Brackets Architecture

Unified pattern for ALL tracks — instrumental and vocal. Pipe (`|`) = "play together."

Per-section metadata tags. Format: `[Type: value, value]` or `[Type: value | value]`

| Type | Examples |
|------|---------|
| `[Energy:]` | `[Energy: Low]`, `[Energy: Rising]`, `[Energy: High]`, `[Energy: Maximum]` |
| `[Vocal Style:]` | `[Vocal Style: Spoken word, Gravelly]`, `[Vocal Style: Power]` (vocal only) |
| `[Instrument:]` | `[Instrument: Shamisen alone | single slow pluck]`, `[Instrument: Full band]` |
| `[Texture:]` | `[Texture: Tape-Saturated]`, `[Texture: Lo-fi]`, `[Texture: Crisp Digital]` |
| `[Mood:]` | `[Mood: Triumphant]`, `[Mood: Vulnerable]`, `[Mood: Morning quiet]` |

Order per section: Structure tag → Energy → Mood → Vocal Style (if vocal) → Instrument → Texture → Lyrics (if vocal)

### Layers

Intentional additions inserted between lyrics within a section:
```
[instrumental break saxophone | soft echo]
[Whispered]
(Hey! Hey!)
```

### Guidelines

- Use typed brackets per section for ALL tracks
- Pipe inside `[Instrument:]` = instruments play together
- Instrumental: all lines must be bracketed (unbracketed text = vocals)
- Phonetic sounds ("Ooh", "Aah", "La la la") guide vocal melodies

## Emotion Performance Craft

Suno reads lyrics like a script. Brackets = director notes. Regular text = the script. Parentheses = background/backing vocal.

### Line-Level Placement

Emotion tags go **immediately before the specific line** they affect — not at the top of a verse. The AI reads top-to-bottom.

**Wrong:** `[Sobbing voice]` then `[Verse]` then lyrics
**Right:** `[Verse]` then lyrics then `[Sobbing voice and choked up]` then emotional line

The AI's emotion attention span is ~1-3 lines, then reverts to normal. Keep emotional lines short, re-tag when needed.

### Punctuation as Performance

| Punctuation | Effect | Example |
|-------------|--------|---------|
| `...` | Hesitation, trailing off | `I thought we had... time...` |
| Typed stutters | Vocal breaks, cracks | `I, I miss you` / `No, no, no` |
| ALL CAPS | Screaming, intensity | `I CAN'T TAKE THIS ANYMORE` |
| Short 2-word lines | Forces quiet | `Don't move.` / `Stay here.` |

### Compound Emotion Tags

Simple tags like `[Sad]` just shift key to minor. Compound tags trigger actual performances:

| Tag | What it triggers | Text to pair |
|-----|-----------------|-------------|
| `[Sobbing voice and choked up]` | Crying, vocal crack | Stutters, ellipses, short lines |
| `[Manic laughter and insane spoken word]` | Mania, laughing mid-song | `HA HA HA HA HA` in caps |
| `[Spoken word whispering]` / `[Breathy voice]` | ASMR, whisper | 2-word lines, `minimalist` in Style |
| `[Screaming vocals]` / `[Scream]` | Guttural roar | ALL CAPS + `RAAAH!`, `GO!` |
| `[Crying and vulnerable]` | Raw exposure | Ellipses, repeated words, fragments |
| `[Angry spoken word]` | Aggressive no-melody | Short punchy lines, caps on key words |
| `[Gentle humming, tender]` | Soft wordless warmth | `mmm... mmm...`, `ooh...` |

### Phonetic Performance Writing

| Sound | How to write | Use with tag |
|-------|-------------|-------------|
| Laughter | `HA HA HA HA HA` (caps) | `[Manic laughter]` |
| Crying | `hh... hh...` or stuttered words | `[Sobbing voice]` |
| Scream | `RAAAH!` / `AAAAGH!` (caps) | `[Scream]` |
| Sigh | `ahh...` (lowercase, trailing) | `[Tender]` / `[Vulnerable]` |
