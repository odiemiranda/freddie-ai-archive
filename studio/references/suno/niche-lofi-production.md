# Lo-Fi Production Theory

<!-- TOC
| Section | Line | Keywords |
|---------|------|----------|
| Lo-Fi Production Fundamentals | 18 | lo-fi, lofi, production, fundamentals, smoothness, rules, core |
| Smoothness Formula | 20 | smoothness, warm, midrange, soft highs, controlled lows |
| Jazz Chords Mandatory | 26 | jazz chords, 7ths, 9ths, 13ths, harmony, chord voicings |
| Swing Drums Mandatory | 33 | swing drums, off-grid, human feel, lazy shuffle, drunken feel |
| Warm Filtered Bass | 40 | warm bass, filtered bass, low-pass, rolled-off |
| Texture Budget | 46 | texture, budget, vinyl crackle, tape warmth, maximum two |
| Frequency Hierarchy | 53 | frequency, hierarchy, mud zone, bass, kick, high-pass |
| Space Through Subtraction | 60 | space, subtraction, sparse, minimal layers, arrangement |
| Roll Off Highs | 66 | roll off, highs, vintage warmth, sparkle, 10khz |
| Lo-Fi Subgenre Production Guide | 73 | subgenre, production, guide, overview |
| Subgenre Overview Table | 75 | subgenre, table, bpm, key elements, texture level |
| Lo-Fi Hip-Hop | 85 | lo-fi hip-hop, lofi hip hop, boom-bap, dusty, vinyl |
| Chillhop | 108 | chillhop, chill hop, melodic, clean, lighter |
| Lo-Fi Jazz / Jazzhop | 131 | lo-fi jazz, jazzhop, jazz hop, live, brushed drums, complex harmony |
| Lo-Fi Ambient | 154 | lo-fi ambient, drone, field recordings, minimal percussion, texture |
| Lo-Fi House | 177 | lo-fi house, distorted kicks, analog synths, saturation, dance |
| Background Music Production Rules | 201 | background music, study, focus, concentration, non-intrusive |
| Quality Keywords for Suno | 227 | quality, keywords, clean mix, analog warmth, tape saturation |
| Common Lo-Fi Chord Progressions | 260 | chord progressions, chords, jazz chords, voicings, ready-to-use |
TOC -->

The production theory behind lo-fi prompts. Use this to understand **why** certain combinations work, then apply that knowledge when building Style blocks. Pairs with `niche-lofi.md` (prompt examples) and `production.md` (general production reference).

---

## Lo-Fi Production Fundamentals

### Smoothness Formula

The signature lo-fi sound comes from three frequency decisions working together:
- **Warm midrange** (300 Hz - 3 kHz) — this is where the emotional weight lives. Rhodes, guitar, and keys should dominate here
- **Controlled lows** (below 150 Hz) — bass and kick only, no competition. Warm but not boomy
- **Soft highs** (above 5 kHz) — rolled off, never bright or crispy. Hi-hats should feel distant, not present

When prompting, use: `warm midrange, soft high-end, controlled low end` or `vintage warmth, mellow tone, gentle presence`

### Jazz Chords Mandatory

Plain triads (C, Am, Dm) sound stiff and amateur in lo-fi. Always specify extended voicings:
- **7ths** (minimum): Cmaj7, Am7, Dm7, Fmaj7 — the bread and butter
- **9ths** (sweet spot): Cmaj9, Dm9, Fmaj9, Am9 — lush and open
- **13ths** (sophisticated): Am13, Dm13 — use sparingly, adds complexity
- **Altered** (tension): G7#5, Dm7b5 — for jazzhop and darker variants

In prompts, specify: `jazz chords, Cmaj7 to Am7 progression` or `extended jazz harmony, 9th chords`. Never just say "piano chords" — Suno defaults to basic triads.

### Swing Drums Mandatory

Quantized drums kill lo-fi instantly. Every lo-fi prompt needs at least one swing indicator:
- `swing drums` — the universal trigger
- `off-grid drums` — human imperfection
- `lazy shuffle` — behind-the-beat feel
- `drunken feel` — extreme swing (use for jazzhop)
- `human feel drums` — subtle timing variation
- `laid-back groove` — slightly behind the beat

Pick ONE swing descriptor per prompt. Stacking them does not make it more swung — it wastes prompt space.

### Warm Filtered Bass

Lo-fi bass should feel like a warm blanket, not a sharp punch:
- Always: `warm bass`, `mellow bass`, `filtered bass`, `soft bass`
- Never: `clean bass`, `bright bass`, `punchy bass`, `slap bass`
- Best instruments: `upright bass`, `Fender P-bass`, `sub bass`, `Moog bass`
- Filtering: `low-pass filtered`, `rolled-off highs`, `warm analog bass`

In prompts: `warm upright bass, low-pass filtered` or `mellow bass groove, analog warmth`

### Texture Budget

**Maximum 2 texture elements per prompt.** This is the most common lo-fi prompting mistake — stacking every texture kills clarity.

Good combinations (pick ONE row):
| Texture 1 | Texture 2 | Vibe |
|-----------|-----------|------|
| vinyl crackle | tape warmth | Classic dusty |
| tape saturation | soft hiss | Analog warmth |
| vinyl crackle | rain ambience | Rainy study |
| tape flutter | room tone | Bedroom producer |
| bit crush | soft static | Digital lo-fi |

Bad (too many): `vinyl crackle, tape warmth, rain sounds, room noise, bit crush` — Suno cannot render five texture layers cleanly. The result is muddy noise.

### Frequency Hierarchy

Where each element lives in the frequency spectrum:
- **Sub bass / Kick**: below 150 Hz — these own this range exclusively
- **Bass notes**: 80-300 Hz — fundamental only, harmonics roll off
- **Keys / Guitar / Melody**: 300 Hz - 3 kHz — the emotional center
- **Hi-hats / Textures**: 3-10 kHz — soft and distant
- **Air / Sparkle**: above 10 kHz — rolled off in lo-fi, only boost if intentional

**The mud zone (100-500 Hz):** When bass, keys, and guitar all stack here, the mix becomes thick and undefined. Prompt for `clean low end` or `separated instruments` to help Suno keep this zone clear.

### Space Through Subtraction

Lo-fi breathes through what is NOT there:
- Prompt for: `sparse arrangement`, `minimal layers`, `breathing room`, `space between notes`
- Do NOT add reverb/delay to create space — that adds frequency content, not removes it
- Maximum 3-4 instruments per prompt (including drums as one)
- If the prompt has drums + bass + keys + guitar + strings, it is too dense for lo-fi

Rule of thumb: **If you can count more than 4 sound sources in your prompt, remove one.**

### Roll Off Highs

The vintage warmth of lo-fi comes primarily from reduced high frequencies:
- `rolled-off highs` — direct instruction to Suno
- `vintage warmth` — implies high-end reduction
- `mellow tone` — softer than bright
- `soft high-end` — gentle presence above 5 kHz
- Subtle high boost ONLY for: city pop lo-fi (needs some shimmer), morning/uplifting variants

---

## Lo-Fi Subgenre Production Guide

### Subgenre Overview Table

| Subgenre | BPM | Key Elements | Texture Level | Common Keys | Chord Style |
|----------|-----|-------------|---------------|-------------|-------------|
| Lo-Fi Hip-Hop | 70-90 | Swing drums, jazz keys, warm bass, vinyl | Medium (dusty, dry) | C Minor, A Minor | 7ths, 9ths |
| Chillhop | 75-90 | Cleaner production, melodic focus, lighter feel | Low (vinyl sparingly) | G Major, D Major | 7ths, some 9ths |
| Lo-Fi Jazz (Jazzhop) | 70-85 | Live-sounding, complex harmony, brushed drums | Medium | F Major, Bb Major | 9ths, 13ths, altered |
| Lo-Fi Ambient | 60-75 or free | Drone pads, field recordings, minimal percussion | High (texture IS the music) | C Major, F Major | Sustained, modal |
| Lo-Fi House | 110-124 | Distorted kicks, analog synths, heavy saturation | High (grit is the point) | A Minor, D Minor | Simple 7ths, repetitive |

---

### Lo-Fi Hip-Hop

**What defines it:** The classic "lofi girl" sound. Dusty boom-bap drums with jazz instrument samples. Nostalgic, slightly melancholic, study-friendly.

**Essential instruments (pick 2-3):**
- Rhodes / Wurlitzer electric piano (the default lead)
- Upright bass or warm sub bass
- Boom-bap drums with swing
- Optional: muted trumpet, soft guitar sample

**Mood descriptors that work:** `nostalgic`, `dusty`, `melancholic but peaceful`, `warm`, `cozy`, `late afternoon`, `rainy window`

**What to AVOID:**
- Bright or clean production (`crisp`, `hi-fi`, `polished`)
- Acoustic drums or live-sounding kit (too present)
- More than one melodic instrument besides keys
- Aggressive or punchy bass

**Style block formula:**
`[Instrumental] Lo-fi hip hop, [KEY INSTRUMENT] with jazz chords, swing boom-bap drums, warm bass, [ONE TEXTURE], [KEY], [BPM] BPM, [MOOD] [No Vocals]`

**Recommended settings:** Weirdness 30-50%, Style Influence 50-70%

---

### Chillhop

**What defines it:** Lo-fi hip-hop's cleaner, more melodic cousin. Less dust, more groove. Positive energy rather than melancholy.

**Essential instruments (pick 2-3):**
- Clean electric piano or bright Rhodes
- Acoustic guitar (fingerpicked)
- Tight drums with lighter swing
- Optional: flute, vibraphone, melodica

**Mood descriptors that work:** `breezy`, `sunny`, `positive`, `laid-back`, `afternoon`, `spring`, `light`, `groovy`

**What to AVOID:**
- Heavy vinyl crackle or tape saturation (defeats the clean sound)
- Dark or melancholic moods
- Dense textures — chillhop is about clarity
- Sub-heavy bass (keep it melodic and mid-range)

**Style block formula:**
`[Instrumental] Chillhop, melodic [KEY INSTRUMENT], light swing drums, warm melodic bass, [KEY], [BPM] BPM, [POSITIVE MOOD], clean production [No Vocals]`

**Recommended settings:** Weirdness 25-45%, Style Influence 55-75%

---

### Lo-Fi Jazz / Jazzhop

**What defines it:** Jazz performance aesthetics with lo-fi texture. Sounds like a late-night jazz club recorded on tape. More complex harmony, more "live" feel than standard lo-fi hip-hop.

**Essential instruments (pick 2-3):**
- Jazz piano (not Rhodes — acoustic piano with warmth)
- Upright bass (plucked, walking)
- Brushed drums or soft jazz kit
- Optional: muted trumpet, tenor sax, jazz guitar

**Mood descriptors that work:** `smoky`, `late-night club`, `intimate`, `sophisticated`, `warm`, `after hours`, `candlelit`

**What to AVOID:**
- Electronic drums or programmed beats (need organic/live feel)
- Simple chord voicings (this subgenre demands complexity)
- Bright, clean production (needs tape warmth)
- Heavy boom-bap patterns (too hip-hop, not enough jazz)

**Style block formula:**
`[Instrumental] Lo-fi jazz, [JAZZ INSTRUMENT] with extended harmony, brushed drums with swing, walking upright bass, [ONE TEXTURE], [KEY], [BPM] BPM, [SMOKY/INTIMATE MOOD] [No Vocals]`

**Recommended settings:** Weirdness 40-60%, Style Influence 45-65%

---

### Lo-Fi Ambient

**What defines it:** Texture IS the music. Minimal or no drums. Drones, pads, and field recordings create an immersive sound environment. Closest to soundscape art.

**Essential instruments (pick 2-3):**
- Warm synth pads or drone
- Field recordings (rain, wind, forest, ocean)
- Soft piano (sparse, widely spaced notes)
- Optional: bowed strings, singing bowls, soft guitar harmonics

**Mood descriptors that work:** `ethereal`, `floating`, `meditative`, `dream-like`, `expansive`, `still`, `vast`, `weightless`

**What to AVOID:**
- Drums or any rhythmic pulse (kills the ambient quality)
- Melodic hooks or catchy phrases (this is texture music)
- Fast tempo or BPM markings (use `free tempo` or `no fixed tempo`)
- More than one active melodic element at a time

**Style block formula:**
`[Instrumental] Lo-fi ambient, warm drone pads, [FIELD RECORDING], sparse piano notes, [KEY], slow and free, [EXPANSIVE MOOD], deep reverb, analog warmth [No Vocals]`

**Recommended settings:** Weirdness 50-75%, Style Influence 35-55% — higher weirdness creates more interesting textures

---

### Lo-Fi House

**What defines it:** Dance music with lo-fi aesthetics. Distorted, saturated, raw. Repetitive grooves with analog grit. Sounds like a house track recorded on a broken cassette player.

**Essential instruments (pick 2-3):**
- Distorted kick drum (the anchor)
- Analog synth chords or stabs
- Filtered vocal samples or chopped vocals
- Optional: acid bassline, tape-delayed keys

**Mood descriptors that work:** `gritty`, `raw`, `hypnotic`, `warehouse`, `underground`, `late night`, `sweaty`, `analog`

**What to AVOID:**
- Clean or polished production (needs grit and saturation)
- Complex chord progressions (keep it simple and repetitive)
- Soft or ambient moods (this is dance music with attitude)
- Slow BPM (below 110 loses the house energy)

**Style block formula:**
`[Instrumental] Lo-fi house, distorted kick, [ANALOG SYNTH ELEMENT], heavy tape saturation, [KEY], [BPM] BPM, [GRITTY MOOD], raw analog production [No Vocals]`

**Recommended settings:** Weirdness 45-65%, Style Influence 50-70%

---

## Background Music Production Rules

When the target is background, study, or focus music, apply these additional constraints on top of the subgenre rules:

### What Makes Music "Background"

Background music stays below conscious attention. The moment a listener notices the music, it has failed as background.

**Rules for background-friendly prompts:**

1. **No bright memorable melodies** — if someone could hum it after one listen, it is foreground music. Prompt for `ambient melody`, `subtle melodic movement`, or `gentle harmonic motion` instead of `catchy melody` or `memorable hook`

2. **Consistent dynamics** — no sudden volume changes, builds, or drops. Prompt for `even dynamics`, `consistent volume`, `no dynamic shifts`, `steady energy level`

3. **Loop-friendly structures** — the track should feel like it could repeat forever. Prompt for `seamless loop`, `repeating structure`, `cyclical arrangement`, `no clear ending`

4. **Ambient textures over melodic hooks** — texture creates atmosphere without demanding attention. Prefer `warm pad`, `soft texture`, `ambient wash` over `lead melody`, `hook`, `riff`

5. **Pulsating, concise patterns** — short repeating musical phrases that create gentle momentum without development. Prompt for `repeating motif`, `ostinato pattern`, `pulsating rhythm`

6. **No dramatic transitions** — no big drops, breakdowns, or build-ups. Prompt for `smooth transitions`, `gradual changes`, `no drops`, `seamless flow`

### Background Music Exclude Style Suggestions

Use these in the Exclude Style (negative prompt) field:
- `dramatic build, drop, breakdown, sudden silence, crescendo`
- `catchy hook, memorable melody, singalong, anthem`
- `aggressive drums, loud percussion, cymbal crash`

---

## Quality Keywords for Suno

Specific keywords and phrases that measurably improve Suno's lo-fi output quality. These act as production directives that Suno's model responds well to.

### Separation and Clarity

| Keyword | What It Does | When to Use |
|---------|-------------|-------------|
| `clean mix` | Improves instrument separation, reduces frequency overlap | Always useful, especially with 3+ instruments |
| `separated instruments` | Each instrument occupies its own frequency space | When the prompt has multiple melodic elements |
| `warm but defined` | The lo-fi sweet spot: soft but not muddy | Default for most lo-fi prompts |
| `clear low end` | Keeps bass tight and controlled, prevents boominess | When bass and kick are both present |

### Character and Warmth

| Keyword | What It Does | When to Use |
|---------|-------------|-------------|
| `analog warmth` | Triggers lo-fi processing, soft saturation, rounded transients | Core lo-fi descriptor, use freely |
| `tape saturation` | Adds harmonic warmth and gentle compression | When you want audible analog character |
| `soft high-end` | Prevents harsh frequencies, maintains warmth | When cymbals or bright instruments are present |
| `vintage warmth` | Implies era-appropriate EQ and processing | For retro-leaning variants |

### Space and Depth

| Keyword | What It Does | When to Use |
|---------|-------------|-------------|
| `intimate room` | Small space reverb, close and personal | Bedroom producer aesthetic |
| `deep reverb` | Larger space, instruments float | Ambient and dreamy variants |
| `dry and close` | Minimal reverb, upfront sound | Boom-bap and hip-hop variants |
| `lo-fi room tone` | Subtle room ambience as a texture | When you want "recorded in a room" feel |

### Combining Keywords

Layer ONE keyword from each category for best results:

- Study lo-fi: `clean mix, analog warmth, intimate room`
- Jazzhop: `separated instruments, tape saturation, dry and close`
- Lo-fi ambient: `warm but defined, vintage warmth, deep reverb`
- Lo-fi house: `clear low end, tape saturation, lo-fi room tone`

Do NOT stack multiple keywords from the same category — diminishing returns and wasted prompt characters.

---

## Common Lo-Fi Chord Progressions

Ready-to-reference progressions for lo-fi prompts. Specify these directly in Style blocks when you want harmonic control.

### By Mood

**Classic (warm, familiar, study-friendly):**
`Am7 - Dm7 - G7 - Cmaj7`
A satisfying cycle that resolves gently. The default study lo-fi progression. Works in any lo-fi subgenre.

**Sophisticated (lush, open, jazzy):**
`Cmaj9 - G7#5 - Am11 - Fmaj9`
Extended voicings with an altered dominant for tension. Best for jazzhop and late-night variants.

**Melancholic (sad, wistful, introspective):**
`Dm9 - Am7 - Em7 - Gmaj7`
Descending emotional arc that never fully resolves. Perfect for rainy day and late night drive moods.

**Chill (easy, breezy, positive):**
`Fmaj7 - Em7 - Dm7 - Cmaj7`
Smooth stepwise descent in major. Ideal for chillhop and morning/afternoon vibes.

**Dark (moody, noir, tension):**
`Cm7 - Fm7 - Bb7 - Ebmaj7`
Minor key ii-V-I with weight. Use for dark academia, cyberpunk, and midnight sessions.

### Usage in Prompts

When specifying progressions in Style blocks, write them naturally:
- Good: `jazz chords, Am7 to Dm7 to G7 to Cmaj7 progression`
- Good: `Cmaj9 and Fmaj9 jazz harmony`
- Bad: `Am7 - Dm7 - G7 - Cmaj7` (the dashes may confuse Suno)

For simpler control, just specify the chord quality:
- `extended jazz chords, 9th voicings` — lets Suno choose the progression
- `minor 7th chords, melancholic harmony` — constrains mood without dictating notes
- `major 9th chords, warm open voicings` — lush and positive

### Progression Complexity by Subgenre

| Subgenre | Complexity | Specify in Prompt? |
|----------|------------|-------------------|
| Lo-Fi Hip-Hop | Medium — 7ths and 9ths | Optional, chord quality is enough |
| Chillhop | Low-Medium — 7ths mostly | Usually not needed, mood descriptors suffice |
| Lo-Fi Jazz / Jazzhop | High — 9ths, 13ths, altered | Yes, specify for best results |
| Lo-Fi Ambient | Minimal — sustained chords, modal | No, just specify `modal harmony` or `sustained chords` |
| Lo-Fi House | Low — simple 7ths, repetitive | No, the groove matters more than harmony |
