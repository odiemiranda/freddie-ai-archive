---
id: mem-tala-20260325-notation-01
title: Test musical notation in Lyrics bracket tags
created: 2026-03-25
tags: [suno, lyrics, notation, testing, register, rhythm, chords]
type: idea
agent: tala
connected: []
status: tested-rejected
---

Idea: use actual musical notation (note names, register numbers, rhythmic values, chord voicings) inside Lyrics bracket tags instead of vague descriptive language.

## Why this could work

Suno already responds to:
- Key signatures in Style blocks (D Major)
- Chord names in Style blocks (Dm7 to Gmaj7)
- Vocal range headers ([Vocal range: E2-E4])
- BPM values

If Suno understands these in Style and vocal headers, it may also understand them in Lyrics brackets — giving us precise instrument control instead of hoping "smooth melodic phrases" means what we want.

## Three levels to test

1. **Register control:** `[Shamisen melody in D4 register]`, `[Harp chords in D2-D3 range]`
   - Most likely to work — similar to vocal range headers
   - Would solve the "harp going high" problem permanently

2. **Rhythmic values:** `[Shamisen eighth note phrases, swung]`, `[Harp whole note chords, sustained]`
   - Would solve the "weird plucking" problem by specifying note duration
   - "Whole note chords" = sustained pad behavior, "eighth notes" = melodic movement

3. **Note names:** `[Shamisen plays D4-F#4-A4 ascending]`, `[Harp holds Dm7 voicing]`
   - Least likely to work as exact pitches, but may nudge intervals
   - Worth testing even if Suno only partially follows

## Test plan

Take the working First Sip Style block (validated clean). Create 3 Lyrics variants:
- A: Current descriptive approach (control)
- B: Register + rhythmic values (Level 1+2)
- C: Full notation (Level 1+2+3)

Same Style, same Exclude, different Lyrics. A/B/C comparison.

## Test Results (2026-03-25)

Same Style block, same Exclude, three Lyrics variants:

- **A (Descriptive):** 100% clean, smooth, production quality
- **B (Register + Rhythm):** 80% clean, weird solo plucking
- **C (Full Notation):** 70% clean, weird plucking, strumming, drone, bad instrument layering

**Conclusion: Musical notation in Lyrics brackets DOES NOT WORK.** More notation = worse output. Suno is a language model, not a MIDI sequencer. It understands descriptive language ("smooth melodic phrase", "low register pad chords") but treats note names and rhythmic values as noise.

**Rule:** Never use note names (D4, F#4), rhythmic values (quarter notes, eighth notes), or chord voicings (Dm7 arpeggio) in Lyrics bracket tags. Chord names work in the **Style block** (Dm7 to Gmaj7) but NOT in Lyrics brackets.

**What works:** Descriptive language with positional cues — the same approach validated through Morning Market testing.
