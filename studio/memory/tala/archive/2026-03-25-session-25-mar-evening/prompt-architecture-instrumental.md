---
name: prompt-architecture-instrumental
description: Hard limits and syntax rules for clean, high-fidelity instrumental prompts.
type: knowledge
agent: tala
tags: [suno, prompt-design, instrument-interaction, exclusion-style, syntax]
---

# Instrumental Prompt Architecture

## 1. Prompt Simplicity Limits
Fewer ingredients = clearer signal. 
- **Max 2 genres** (multiple labels pull Suno in competing directions).
- **Max 3 instruments** (overlapping registers cause frequency fighting).
- **Max 1 rhythm source**.
- **Max 5 mood descriptors**.
- **Style blocks** should read like a scene, not a shopping list.

## 2. Style Block = Palette + Behavior + Dynamics (No Structure Tags)
The Style block needs three layers to produce clean output:
- **Sonic palette:** Genre with rhythm description, mood, BPM, key, texture
- **Instrument behavior:** What each instrument does ("shamisen melodic phrases", "harp low-register pad chords as background warmth")
- **Dynamic control:** "relaxed", "unhurried", "loop-friendly", specific chord progressions. Avoid "subtle melodic movement" and "consistent dynamics" — tested, causes noodling/flat jam feel.

**What stays OUT:** Structure tags (`[Intro]`, `[Build]`, `[Hook]`), staging ("enters", "fades in"), and musical notation (note names, rhythmic values — tested, degrades output). Those go in Lyrics (except notation, which goes nowhere).

**Why:** Stripping Style to bare instrument names causes Suno to guess behavior — leading to chaotic, unfocused generation. Suno needs to know what each instrument does (Style) and when it enters (Lyrics). Validated by Morning Market (instrument behavior in Style = clean) vs Morning Window Groove (bare names = chaotic).

**How to apply:** Always include instrument behavior hints and dynamic control keywords. Never include structure tags or staging directions.

## 3. Instrument Hierarchy & Interaction
Describe how instruments relate to each other rather than listing roles in isolation.

**Interaction verbs:**
- **Use Dynamic Interaction Verbs:** "Trades phrases," "weaves through," "responds to," "answers," "breathing together."
- **Avoid Static Isolation Verbs:** "Holds," "sustains," "sits under."
- **Lead Instrument Continuity:** If an instrument (e.g., Shamisen) is designated as "Lead," it must remain lead across all variations. Vary the support levels, not the lead role.

**Support instrument subordination (critical):**
Support instruments MUST have explicit positional/hierarchy cues in bracket tags. Without these, Suno gives all instruments equal presence and the mix fights.
- **Positional language:** "far behind," "low and distant," "underneath," "harmonic bed only," "no melody of its own"
- **More bracket lines for lead, fewer for support.** Lead gets described in every section. Support gets described in 2-3 sections max.
- **Anti-pattern:** Giving lead and support equal bracket presence — Suno treats them as co-leads and they compete.

**Validated by Morning Market track:** Harp described as "background warmth", "no melody of its own", "harmonic bed only" — produced clean separation. Morning Window Groove described harp as equal partner ("duet", "trading phrases") — produced strumming contest.

## 4. Instrumental Lyrics Formatting
To prevent "vocal leakage" on instrumental tracks:
- **Mandatory Brackets:** Every descriptive line in the Lyrics block MUST be wrapped in `[brackets]`.
- **Mandatory Anchors:** Start with `[Instrumental]` and end with `[No Vocals]`.
- **Bracket tag length: 3-10 words.** Short (3-5) for structure cues. Longer (6-10) when carrying positional or hierarchy info. Over 12 words risks being sung.

## 5. Broad Exclusion Lists
Default to **10-12 exclude items** for instrumental/background tracks.
- **Baseline Exclusions:** vocals, singing, plus any instrument in the same frequency register as the lead that isn't named in the Style block.
- **Cultural Safety:** Cultural labels (e.g., "Celtic harp") trigger genre-associated instruments even with excludes. **Prefer neutral labels:** use "harp" instead of "celtic harp", describe the sound with adjectives instead of cultural tags. Fall back to cultural labels only if neutral labels don't produce the right timbre, and always exclude associated instruments (tin whistle, fiddle, violin, strings).

## 6. Genre Clarity
Use genres that give Suno a clear rhythm template. "Jazz fusion" and "Reggae fusion" work because they define a groove. "Groove pop" fails because it's vague — Suno doesn't know what rhythm to lock into.
- **Good:** "Jazz fusion with laid-back swing", "Reggae fusion, offbeat groove"
- **Bad:** "Groove pop", "Lo-fi jazzhop", "Chillpop"

## 7. No Musical Notation in Lyrics Brackets (Tested, Rejected)
Never use note names (D4, F#4), rhythmic values (quarter notes, eighth notes), or chord voicings (Dm7 arpeggio) in Lyrics bracket tags. Tested A/B/C comparison:
- A (Descriptive): 100% clean
- B (Register + Rhythm): 80% clean, weird solo plucking
- C (Full Notation): 70% clean, plucking, strumming, drone, bad layering

Chord names work in the **Style block** (Dm7 to Gmaj7) but NOT in Lyrics brackets. Suno is a language model, not a MIDI sequencer.

**Consolidated from:** 2026-03-21 through 2026-03-25 testing sessions.
