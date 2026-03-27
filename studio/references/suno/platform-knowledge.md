---
title: Suno Platform Knowledge
tags: [suno, platform, style, formula, meta-tags, bpm, settings, exclude, weirdness, style-influence, audio-influence, advanced-settings, lyrics-syntax]
---

# Suno Platform Knowledge

Core platform rules for Style blocks, meta tags, advanced settings, and lyrics syntax. Extracted from agent file — authoritative reference for all Suno generation.

## The Master Formula

```
[STYLE/GENRE] + [INSTRUMENTS] + [ATMOSPHERE/MOOD] + [TECHNICAL SPECS]
```

**Three tiers:**

| Tier | Focus | Example |
|------|-------|---------|
| **1: Foundation** | Genre + Tempo | "Slow R&B ballad" |
| **2: Character** | + Mood + Instruments | "Slow R&B ballad, romantic and intimate, piano and strings" |
| **3: Polish** | + Vocals + Production | "Slow R&B ballad, romantic and intimate, piano and strings, smooth male vocals, late-night atmosphere" |

## Exclude Style (negative prompt)

Suno's **Exclude Style** field tells the model what to avoid. Use it as guardrails — prevent genre bleed, unwanted instruments, or vocal leakage.

- Comma-separated terms, same format as Style
- **With typed `[Instrument:]` brackets: minimal or empty.** Per-section Instrument tags provide positive constraints that replace excludes. Only add dealbreakers (auto-tune for raw vocal, electronic for acoustic genres). Tested: empty Exclude + typed brackets produced clean output across multiple generations.
- **Without typed brackets (fallback): ~14 focused instrument excludes.** Start with `vocals, singing`, then add every melodic instrument not in the Style block. No abstract mood/energy excludes.
- **Vocal tracks without typed brackets: 3-5 exclude items.** Focus on genre bleed and vocal style drift.
- Don't exclude what the Style already contradicts — if Style says "acoustic guitar", you don't need to exclude "electric guitar"
- Common exclusions by context:

| Context | Useful Exclusions |
|---------|-------------------|
| Instrumental tracks | `vocals, singing, spoken word, rap` |
| Acoustic/folk | `electronic, synth, autotune, heavy bass` |
| Lo-fi/chill | `aggressive, heavy drums, distortion, screaming` |
| Celtic/traditional | `electronic beats, autotune, trap, modern pop` |
| Ambient/sleep | `vocals, drums, percussion, uptempo, bright` |

## Style Prompt Anatomy

**Target: ~75-120 chars (all tracks with typed brackets). Hard max: 200 chars. First 20-30 words carry most weight. All per-section detail goes in Lyrics typed brackets.**

| Position | Element | Budget | What Goes Here |
|----------|---------|--------|----------------|
| 1st | **Tags + Genre** | ~30 chars | `[Instrumental]` + genre. Highest priority — Suno reads this first |
| 2nd | **Instruments + constraints** | ~60-80 chars | Lead + behavior, support + constraint, rhythm + constraint. Keep tight |
| 3rd | **Key, BPM, chords** | ~30-50 chars | Key, BPM. Chords only if budget allows (skip for lean prompts) |
| 4th | **Mood + texture** | ~30-40 chars | 1-2 mood words, 0-2 texture words. Cut first when over budget |
| 5th | **Production + tags** | ~40-50 chars | "clean mix, separated instruments" + `[No Vocals]` `[Background Music]` |

**Total: ~75-120 chars for all tracks using typed brackets.** If you're past 200, you're over-stuffing — move instruments to `[Instrument:]` tags and textures to `[Texture:]` tags in Lyrics. The Style block is just the global foundation.

## Style Block Format

The Style block defines **sonic palette + instrument behavior + dynamic control**. It tells Suno what each instrument does and how the track should feel — but NOT the arrangement timeline (which instrument enters when).

```
[Instrumental] Jazz, shamisen melodic plucks, harp background pad chords, brushed drums sparse, G Major, 88 BPM, warm, clean mix, separated instruments [No Vocals]
```

**NO structure tags** (`[Intro]`, `[Build]`, `[Hook]`) in Style — those go in Lyrics.
**NO staging** ("enters", "fades in", "drifts in") in Style — that goes in Lyrics.
**NO musical notation** (note names, rhythmic values, chord voicings) in Lyrics brackets — tested, degrades output.
**YES instrument behavior** ("melodic phrases", "low-register pad chords no melody") in Style — Suno needs this to know what each instrument does. Keep constraints pure: register + role only. Texture ("warmth") and groove ("laid-back swing") go in their own slots.

## Critical Meta Tags

For instrumental/background music, ALWAYS use:

| Tag | Purpose | Placement |
|-----|---------|-----------|
| `[Instrumental]` | No vocals directive | START |
| `[No Vocals]` | Reinforcement | END |
| `[Ambient]` | Background context | END |
| `[Background Music]` | Non-intrusive | END |

**Structure tags (by reliability):**
- **High (85-95%):** `[Short Instrumental Intro]`, `[Verse]`, `[Chorus]`, `[Catchy Hook]`, `[Bridge]`, `[Big Finish]`, `[melodic interlude]`, `[Final Chorus]`, `[Hook]`
- **Moderate (65-75%):** `[Pre-Chorus]`, `[Post-Chorus]`, `[Instrumental]`, `[Sustained]`, `[End]`, `[Breakdown]`, `[Build]`, `[Drop]`, `[Solo]`, `[Interlude]`
- **Advanced (V4+/V5):** `[Chorus x2]`, `[Outro: Fade out]`, `[Outro: Big finish]`, `[Callback:]`, `[Hook Loop]`, `[Beat switch]`, `[Hook first]`, `[Hook delay]`, `[Crowd-call section]`, `[Band drop-out before final chorus]`, `[Emotional release]`
- **Unreliable — avoid:** `[Intro]` (30-40%), `[Outro]` (40%), `[Fade Out]` (35%), `[Break]` (45%)

**Advanced structure tags:** `[Outro: Fade out]`, `[Outro: Big finish]`, `[Break]`, `[Breakdown]` are verified. `[Final Chorus]`, `[Beat switch]` are natural-language hints (may nudge, not guaranteed). Verbose tags like `[Hook Loop]`, `[Hook delay]`, `[Emotional release]`, `[Band drop-out before final chorus]`, `[Callback: Chorus melody]` are unverified — use standard short tags instead. See `meta-tags-advanced.md` for full reference.

## BPM/Key Quick Reference

| Purpose | BPM | Best Keys |
|---------|-----|-----------|
| Sleep | 40-60 | C Major, F Major |
| Study/Lo-fi | 75-95 | C Minor, A Minor |
| Morning/Energy | 110-130 | G Major, D Major |
| Workout | 130-150+ | E Major, A Major |

**Sweet spots:** Lo-fi 82-85, Dark academia 65-80, City pop 105-125.
**V5 lyrics field max:** 5,000 characters — plan section count accordingly.
**Industry secret:** 80% of viral lo-fi uses C Minor or A Minor.

## Advanced Settings

Include Weirdness and Style Influence in every variation. Audio Influence only when a reference clip is uploaded.

| Setting | Range | Default | What It Controls |
|---------|-------|---------|-----------------|
| **Weirdness** | 0-100% | 50% | Creative risk within the genre. Low = predictable. High = unusual harmonies. **W:50%** sweet spot for most tracks. **W:100%** breaks melody/form |
| **Style Influence** | 0-100% | 50% | Genre purity. Low = loose suggestion. High = commits to genre tropes. **SI:100%** locks AI to the genre |
| **Audio Influence** | 0-100% | — | Only with reference clip. Controls adherence to reference's rhythm/phrasing/energy. Not exact copy — captures feel/groove |

**Key insight:** SI defines *what genre*. W controls *how creative within that genre*. AI controls *how similar to a specific reference*. They stack independently.

**Useful ranges:** W:20-65%, SI:40-85%. Above ~80% for either rarely improves results except for extreme vocal techniques (carnatic, opera — SI:85-90%).

**Audio Influence use cases:**
- Persona consistency — maintain vocal tone across album (AI:70-85%)
- Music card reference — guide toward known groove (AI:40-60%)
- Cover/reimagining — capture energy, new arrangement (AI:50-70%)
- Not recommended at AI:90-100% — derivative results

## Lyrics Syntax Reference

**Structure (high reliability):** `[Short Instrumental Intro]` (90%), `[Verse]` (90%), `[Chorus]` (95%), `[Catchy Hook]` (90%), `[Bridge]` (85%), `[Big Finish]` (85%), `[melodic interlude]` (85%), `[Final Chorus]`, `[Pre-Chorus]` (65-70%), `[Instrumental]` (75%), `[Breakdown]`, `[Build]`, `[Drop]`, `[Sustained]`, `[End]`

**Vocal direction:** `[Soft vocals]`, `[Whispered]`, `[Spoken word]`, `[Powerful vocals]`, `[Belting]`, `[Falsetto]`, `[Harmonies]`, `[A cappella]`, `[Call and response]`, `[Ad-libs]`, `[Humming]`, `[Vocalizing]`

**Verified vocal delivery:** `[Whispered]`, `[Belted]`, `[Falsetto]`, `[Raspy]`, `[Soulful]`, `[Breathy]`, `[Operatic]`, `[Rapped]`, `[Melodic Rap]`, `[Growled]`, `[Harmonies]`, `[Ad-libs]`, `[Melisma]`, `[Vibrato]`. Personas are a Suno UI feature — select from dropdown.

**Dynamics:** `[Slowly building]`, `[Crescendo]`, `[Climax]`, `[Stripped back]`, `[Full band]`, `[Quiet]`, `[Explosive]`

**Instrumental guidance (in lyrics):** `[Guitar solo]`, `[Piano interlude]`, `[Drum break]`, `[Bass drop]`, `[Synth swell]`

**Ad-libs:** Place `[adlib HEY]`, `[adlib boom]` on their own line. Sound text in ALL CAPS. See `meta-tags-advanced.md` for full list.

**Pipe stacking:** Stack cues with `|` inside brackets: `[Verse | 60s jangly guitar | clean Fender tone | light spring reverb]`. Max 4-6 modifiers. Lead with core element.

**Category tags:** `[Instrumentation: overdriven punk guitar | palm-muted power chords]`, `[Vocal: raspy lead | gang shouts]`, `[Mix: mono bass, stereo guitar | room reverb]`

**Performance notation:**
- `( )` — SUNG as backing vocal. **NEVER** instructions in parentheses
- `-` — stretch syllables: `al-most`, `lo-o-o-ove`
- `ALL CAPS` — louder delivery, 1-3 words max per section
- `...` — pauses. More dots = longer pause (NOT sustain — use hyphens for that)
- `~` — experimental, prefer hyphens for reliable sustain
- `" "` — unverified for spoken. Prefer `[Spoken Word]`, `[Whispered]`

**Emotion performance:** Line-level tags before the line they affect. Combine with formatting: `...` hesitation, typed stutters `I, I miss you`, ALL CAPS screaming, short 2-word lines for whispers. Compound tags `[Sobbing voice and choked up]` trigger performances, not key changes. One emotion per tag.

**Duets:** Per-section speaker labels `[Male]`, `[Female]`, `[Both]`. Add gender context to Style. Named characters MAY help. Assign full verses to one singer — switching mid-verse breaks consistency.
