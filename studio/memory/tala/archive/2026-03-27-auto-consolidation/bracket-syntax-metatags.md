---
name: bracket-syntax-metatags
description: Complete bracket syntax rules for Suno Lyrics box — structure tags, colon syntax, pipe stacking, emotion exceptions, call-and-response, vocal techniques.
type: knowledge
agent: tala
tags: [suno, lyrics, brackets, metatags, pipe-stacking, colon-syntax, vocal, performance, adlib]
---

# Bracket Syntax & Meta-Tags

## 1. The Golden Rule
- `[ ]` brackets = instructions. Suno does NOT sing these.
- `( )` parentheses = SUNG as background/backing vocal layer. NEVER put instructions in parentheses.

## 2. Bracket Types

**Structure tags (by reliability):**
- **High (85-95%):** `[Short Instrumental Intro]` (90%), `[Verse]` (90%), `[Chorus]` (95%), `[Catchy Hook]` (90%), `[Bridge]` (85%), `[Big Finish]` (85%), `[melodic interlude]` (85%), `[Final Chorus]`
- **Moderate:** `[Pre-Chorus]` (65-70%), `[Instrumental]` (75%), `[Breakdown]`, `[Build]`, `[Sustained]`, `[End]`
- **AVOID:** `[Intro]` (30-40%), `[Outro]` (40%), `[Fade Out]` (35%), `[Break]` (45%)
Use reliable alternatives: `[Intro]`→`[Short Instrumental Intro]`, `[Outro]`→`[Big Finish]`, `[Break]`→`[melodic interlude]`. Max 5-7 structural tags per song.

**Mood tags (V5, 75-85%):** `[Mood: haunting]`, `[Mood: joyful]`, `[Mood: triumphant]`, `[Mood: somber]`, `[Mood: ethereal]`, `[Mood: bittersweet]`. Max 1-2 per song. Section-level mood modifiers, different from vocal style tags.

**Colon syntax:** `[Verse 1: Building tension, haunting melody]`. Attaches mood/instrumentation to a structural tag. More readable than pipe stacking for simple assignments.

**Pipe stacking:** `[Verse | shamisen tone | reverb]`. Multiple independent modifiers. Order: Core element → Genre/Era → Tone → Effects. Max 4-6 modifiers. **Exception:** Emotion delivery tags (`[Crying voice]`, `[Angry tone]`) must NOT be pipe-stacked — go alone on their own line above the target lyric.

**Key override:** `[Key: D minor]` steers tonal center from that point forward. Useful for mid-song modulations.

**Vocal style tags:** `[Whispered]`, `[Belted]`, `[Raspy]`, `[Soulful]`, `[Rapped]`, `[Operatic]`, `[Harmonies]`. Add texture adjectives: "breathy", "gravel", "smooth", "ethereal". `close-mic` triggers proximity effect.

**Ad-lib tags:** `[adlib HEY]`, `[adlib UH]` — own line, ALL CAPS word.

**Category-level tags:** `[Instrumentation: overdriven punk guitar | palm-muted power chords]`, `[Vocal: raspy lead, light grit]`, `[Mix: mono bass, stereo guitar]`.

**Call-and-response:** `[instrumental break saxophone]` placed between lyric lines creates instrument answering the vocal.

## 3. Structural Rules

- **Every section must start with a structural anchor** — vocal and instrumental tracks alike.
- **Bracket tag length:** 3-10 words. Over 12 words risks being sung or ignored.
- **For instrumental tracks:** Every line wrapped in `[brackets]`. Max 3 direction lines per section.
- **Closing anchors:** Always `[No Vocals] [End]`. `[Background Music]` goes in Style block only.
- **Genre consistency:** Bracket tags must match Style block genre. No `[808 sub bass]` in country.

## 4. Lyrics Header Block
Top of Lyrics box, before any section tags:
1. Vocal range header (vocal tracks): `[Voice]` + `[Vocal range: note-note]` + `[Exclude: ...]`
2. Production header (optional): `[High-fidelity stereo sound]` — only when Style block isn't specific enough.

## 5. Vocal Performance Techniques

| Technique | How |
|---|---|
| Tilde vibrato | `free~dom~`, `ho~me` — hold note with pitch bend |
| Delayed/stretched | `I... lo...ve... you` — emotional weight |
| Stuttered entry | `I, I didn't think...` — vulnerability |
| ALL CAPS burst | `WE RISE` — max 1-3 words/section. Add `!`/`?` for extra. For screams: `[Scream]` then `AAAAAH` |
| Whisper drop | `[Whispered] don't go.` — intimacy after loud section |
| Gang vocal | `(Hey! Hey! Hey!)` — parens = sung backing layer |
| Vocal drone | `[Vocal drone]` + `(deep resonant)` + `Whispers.....................` — cinematic/ambient |
| Ellipsis sustain | More dots = longer hold |
