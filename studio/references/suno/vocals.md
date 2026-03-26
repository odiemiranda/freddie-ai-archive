# Vocals Reference

<!-- TOC
| Section | Line | Keywords |
|---------|------|----------|
| Vocal Specification Formula | 42 | formula, specification, gender, delivery, technique, processing |
| Vocal Range Control | 52 | range, pitch, notes, E2, G2, C2, bass, baritone, tenor, soprano, header, exclusion, register, Hz, frequency, drift, constraint |
| Human Vocal Range Map | 74 | range map, C2, C6, bass, baritone, tenor, alto, soprano, mezzo, countertenor, Hz |
| Range Presets for Suno | 108 | presets, sub-bass, bass, baritone, tenor, contralto, alto, soprano, coloratura |
| Exclusion Lists | 133 | exclude, falsetto, register drift, soprano, head voice, negative prompting |
| Advanced Techniques | 157 | proximity effect, close mic, drone, sub-bass narrator, texture stacking, rasp |
| Suno Vocal Range Tips | 203 | tips, range, header, exclusion, drift, chorus, floor |
| Phonetic Vocal Control | 216 | phonetic, accent, misspelling, dictionary, pronunciation, rendering, mouth shape, spelling, notation, regional |
| Phonetic Translation Workflow | 240 | phonetic, chatgpt, translator, rewrite, accent, standard |
| Spelling as Musical Notation | 267 | spelling, vowel, consonant, duration, note length, sustain, breath, phrasing |
| Accent Recipes | 282 | accent, scottish, jamaican, southern, irish, cockney, regional, patois, highland, drawl |
| Combining with Vocal Range Control | 319 | combine, range, phonetic, stack, accent, header |
| Phonetic Control Tips | 345 | tips, phonetic, accent, triple reinforcement, vowel, consonant |
| Extreme Vocal Techniques | 358 | extreme, capability, carnatic, opera, yodeling, scat, whistle register, overtone, throat singing, haka, melodic bias, tested |
| Gender + Age Descriptors | 395 | gender, age, female, male, soprano, alto, tenor, baritone |
| Female Vocals | 397 | female, soprano, alto, mezzo-soprano |
| Male Vocals | 404 | male, baritone, tenor, bass, deep, raspy |
| Other | 411 | mixed, choir, harmonies, androgynous, ensemble |
| Delivery Styles | 420 | delivery, style, soft, powerful, textured, smooth |
| Soft/Gentle | 422 | soft, gentle, breathy, whispered, tender, vulnerable, airy |
| Powerful/Strong | 432 | powerful, strong, belting, soaring, commanding, fiery |
| Textured | 442 | textured, raspy, smoky, gravelly, gritty, husky, growling |
| Smooth | 452 | smooth, silky, velvety, sultry, honeyed, crooning, mellow |
| Expressive | 464 | expressive, dramatic, playful, confident, robotic |
| Vocal Techniques | 474 | techniques, classical, contemporary, experimental |
| Classical | 476 | classical, operatic, bel canto, vibrato, head voice, falsetto |
| Contemporary | 486 | contemporary, belting, runs, melisma, ad-libs, vocal fry, scatting |
| Extended/Experimental | 496 | extended, experimental, spoken word, chanting, overtone, throat singing |
| Vocal Processing | 508 | processing, reverb, effects, auto-tune, vocoder, distorted |
| Reverb/Space | 510 | reverb, space, dry, plate, hall, room |
| Effects | 519 | effects, auto-tune, vocoder, distorted, echo, chorus, bitcrush |
| Character | 540 | character, vintage, modern, lo-fi, pristine, natural |
| Vocal Direction Tags | 552 | direction tags, lyrics tags, soft, whispered, spoken word |
| Vocal Arrangements | 573 | arrangements, solo, harmony, ensemble, choir |
| Solo | 575 | solo, lead vocals |
| Harmony | 581 | harmony, two-part, three-part, stacked, tight, wide |
| Ensemble | 591 | ensemble, choir, gospel, backing, gang vocals, unison |
| Genre-Specific Vocal Styles | 603 | genre, pop, r&b, rock, hip-hop, jazz, folk, electronic, country, metal |
| Quick Reference: Mood to Vocal Style | 652 | quick reference, mood, vocal style, cheat sheet |
| Vocal Track Workflow | 667 | workflow, vocal track, style prompt, lyrics, tips |
| Instrumental-Only Prompts | 776 | instrumental, no vocals, background music |
TOC -->

## Vocal Specification Formula

```
[Gender] + [Delivery Style] + [Technique] + [Processing]
```

Example: "Soft female vocals, breathy delivery, intimate whisper, reverb-heavy"

---

## Vocal Range Control

Adjectives like "deep" or "low" are opinions — Suno interprets them inconsistently. Note ranges are facts. Use a **header block** at the top of the lyrics box to constrain the vocal register with specific note values and an exclusion list.

### The Header Format

Place bracketed instructions at the very top of the lyrics, before any section tags:

```
[Deep bass-baritone male vocal, gravelly texture, chest voice only]
[Vocal range: E2-E4]
[Exclude: falsetto, soprano register, head voice, bright tenor, high notes]

[Verse 1]
(lyrics here...)
```

**Three parts:**
1. **Voice description** — character + texture (gravelly, smooth, breathy, etc.)
2. **Range constraint** — specific notes (e.g., `E2-E4`)
3. **Exclusion list** — what the AI must NOT do (prevents octave jumps on choruses)

### Human Vocal Range Map

```
              C2    E2    G2    C3    E3    G3    C4    E4    G4    C5    E5    G5    C6
              |     |     |     |     |     |     |     |     |     |     |     |     |
Bass        : ████████████████████████████
              C2--------------------E4
              65 Hz               330 Hz

Baritone    :       ██████████████████████████████
                    G2------------------------G4
                    98 Hz                   392 Hz

Tenor       :             ████████████████████████████████
                          C3----------------------------C5
                          131 Hz                      523 Hz

Countertenor:                         ████████████████████████████
                                      E3--------------------------E5
                                      165 Hz                    659 Hz

Alto        :                   ██████████████████████████████
                                E3------------------------E5
                                165 Hz                  659 Hz

Mezzo-Sop.  :                         ████████████████████████████████
                                      A3------------------------------A5
                                      220 Hz                        880 Hz

Soprano     :                               ████████████████████████████████
                                            C4------------------------------C6
                                            262 Hz                       1047 Hz
```

### Range Presets for Suno

#### Male Voice Presets

| Preset | Range | Hz (low) | Character | Use For |
|--------|-------|----------|-----------|---------|
| **Sub-bass narrator** | C2-C3 | 65 Hz | Deepest human voice, movie trailer, rumble | Intros, outros, spoken narration |
| **Deep bass** | C2-E4 | 65 Hz | Heavy floor, church bass | Gospel, doom, dark ambient |
| **Bass-baritone** | E2-E4 | 82 Hz | Reliable low register, consistent weight | Country bass, blues, folk |
| **Standard baritone** | G2-G4 | 98 Hz | Natural male speaking range, versatile | Rock, jazz, most genres |
| **Textured baritone** | G2-G4 | 98 Hz | Same range + gravel/rasp tags | Swamp rock, blues rock, whiskey country |
| **Low tenor** | B2-A4 | 123 Hz | Warm middle ground, can push up | Pop-rock, soul, R&B |
| **Tenor** | C3-C5 | 131 Hz | Bright, open, melodic | Pop, musical theatre, rock |
| **High tenor** | E3-E5 | 165 Hz | Light, soaring | Falsetto pop, indie, art rock |

#### Female Voice Presets

| Preset | Range | Hz (low) | Character | Use For |
|--------|-------|----------|-----------|---------|
| **Contralto** | E3-E5 | 165 Hz | Deepest female, rich, dark | Jazz, blues, torch songs |
| **Alto** | F3-F5 | 175 Hz | Warm, grounded, earthy | Folk, soul, indie |
| **Mezzo-soprano** | A3-A5 | 220 Hz | Versatile, natural center | Pop, R&B, musical theatre |
| **Soprano** | C4-C6 | 262 Hz | Bright, light, soaring | Opera, pop ballads, ethereal |
| **Coloratura** | C4-E6 | 262 Hz | Agile, ornamented, highest | Classical, dramatic |

### Exclusion Lists

The exclusion list prevents the most common Suno problem: **register drift** — the voice starts low but jumps to a high glossy tenor on the chorus.

**For deep male vocals (bass/baritone):**
```
[Exclude: falsetto, soprano register, head voice, alto, high notes, bright tenor, light vocals, airy]
```

**For tenor:**
```
[Exclude: bass, baritone, deep rumble, sub-bass, growling, chest-only]
```

**For female alto/contralto:**
```
[Exclude: soprano, high notes, bright, light, airy, coloratura, operatic high]
```

**For soprano:**
```
[Exclude: bass, baritone, deep, chest voice only, dark, heavy, low register]
```

### Advanced Techniques

#### Proximity Effect (bass boost via mic technique)
When a singer gets very close to a microphone, physics boosts the bass frequencies. Trigger this in Suno with production tags:

```
[Close mic, sub-bass male vocal, intimate whisper, no reverb]
[Vocal range: C2-C4]
[Exclude: falsetto, soprano, bright, reverb, distant]
```

Combine with `dry vocals` in the Style box. The result is heavy, intimate, and present — the voice fills the low end without needing to be loud.

#### Single-Note Drone (goth/doom)
Restrict the range to a single note to force a monotone drone:

```
[Dark bass vocal drone, single sustained note]
[Vocal range: E2-E2]
[Exclude: melody, falsetto, soprano, rising pitch, variation]
```

Style: `gothic rock, post-punk, doom metal, slow tempo, reverb, cold`

#### Sub-Bass Narrator (trailers/intros)
Strip instruments via exclusion to let the voice own all low frequencies:

```
[Sub-bass male narrator, cinematic, authoritative]
[Vocal range: C2-C3]
[Exclude: singing, melody, instruments, drums, falsetto, soprano]
```

The voice becomes the entire track. Use for album intros, outros, spoken interludes.

#### Texture Stacking (rasp + range)
Range controls pitch. Texture tags control timbre. Stack them:

```
[Gravelly bass-baritone, whiskey-soaked, rough vocal cords]
[Vocal range: G2-G4]
[Exclude: clean, polished, falsetto, soprano, smooth, silky]
```

The AI responds to words like "gravelly" and "rough" by adding harmonic noise to the vocal signal — character that a plain "low voice" tag never produces.

### Suno Vocal Range Tips

1. **"Deep" is an opinion, E2 is a fact** — always use note ranges over adjectives for pitch control
2. **The exclusion list is mandatory** — without it, Suno will drift to its comfortable tenor-pop default on choruses
3. **Range + texture = full control** — range sets the pitch floor/ceiling, texture tags set the timbre
4. **Narrower range = more consistent** — E2-E4 (two octaves) is more reliable than E2-G5 (three+ octaves)
5. **Header goes in the lyrics box, not the Style box** — Style box handles genre and production; the lyrics header handles vocal performance
6. **Proximity effect stacks with range** — `close mic` + low range + `dry` = maximum bass presence
7. **Test the floor** — C2 (65 Hz) is the absolute limit. Most Suno generations can't reliably hit below D2. E2 is the safe floor for consistent output.
8. **Choruses are where drift happens** — Suno naturally wants to push higher on choruses. The exclusion list + range constraint fights this.

---

## Phonetic Vocal Control

Suno has a hard-coded dictionary trained on clean, standardized data. When it recognizes a word, it pulls a preset pronunciation — polished, generic, American English. The result: every voice sounds like a "Disney prince" regardless of what you wrote in the Style box.

**The fix:** misspell words phonetically. When Suno hits a word that doesn't exist in its dictionary, it stops reading and starts *rendering* — forced to interpret the literal sounds of the letters. You're not just changing lyrics. You're rewiring vocal cords.

### How It Works

Suno's brain is split in two:
1. **Style box** → handles the vibe (genre, mood, production)
2. **Lyrics box** → a text-to-speech engine that converts words to vocal sounds

When you write `home`, Suno recognizes it, pulls the preset, hands you a polished result. When you write `ho-ome`, Suno finds nothing in the dictionary and is forced to render the sounds manually — on the fly, with character.

### The Repetition Rule (Triple Reinforcement)

One layer isn't enough. Suno will try to correct back to standard delivery. Stack three layers to lock the voice:

1. **Style box** — genre + accent + vocal character (e.g., `Scottish folk, thick Highland accent, gritty raw vocals`)
2. **Metadata header** — voice directive at top of Lyrics box (e.g., `[Voice: deep, raw, regional Scottish accent]`)
3. **Phonetic lyrics** — misspelled words that force the pronunciation

All three must agree. If the Style says Scottish but the lyrics are clean American English, Suno compromises with a generic accent.

### Phonetic Translation Workflow

Use ChatGPT (or any LLM) as a phonetic translator:

1. Write the lyrics in standard English first
2. Prompt ChatGPT: "Rewrite these lyrics phonetically for a [Scottish/Jamaican/Southern] accent. Spell words as they would be pronounced in that accent."
3. Review the output — adjust for singability (the phonetic version must still flow when sung)
4. Paste into Suno's Lyrics box with the metadata header

**Example — Standard → Scottish phonetic:**
```
Standard: "I am walking away from the hills of my youth"
Phonetic: "Ah'm walkin' awaa frae the hills o' ma yooth"
```

**Example — Standard → Jamaican phonetic:**
```
Standard: "The water is cold"
Phonetic: "Da wata be cole"
```

**Example — Standard → Southern Gothic:**
```
Standard: "The ghosts in the valley are calling my name"
Phonetic: "The gho-osts in tha valley ah callin' mah na-ame"
```

### Spelling as Musical Notation

Phonetic spelling doesn't just control accent — it controls **note duration and mouth shape**:

| Technique | What It Does | Example |
|-----------|-------------|---------|
| **Stretched vowels** | Holds the note longer | `na-ame` instead of `name` — forces a long A |
| **Hard consonants** | Punches the beat | `Da wata` — the D forces a hard attack |
| **Open vowels** | Keeps the mouth open, warmer tone | `awaa` instead of `away` — stays open |
| **Dropped endings** | Creates casual/regional feel | `walkin'` instead of `walking` |
| **Doubled vowels** | Extends sustain | `coole` instead of `cool` — lingers |
| **Hyphenated syllables** | Controls exact phrasing rhythm | `gho-osts` — two distinct beats |

**Key insight:** Standard spelling gives Suno permission to clip words short. Phonetic spelling tells Suno exactly how long each sound lasts. You're using spelling as breath control.

### Accent Recipes

#### Scottish Highland
```
Style: Scottish folk, thick Highland accent, gritty raw vocals, acoustic guitar
Header: [Voice: deep, raw, regional Scottish accent, rolling Rs]
Phonetic patterns: "oo" → "oo" (keep), "away" → "awaa", "my" → "ma", "from" → "frae", drop -ing → -in'
```

#### Jamaican / West Indies
```
Style: reggae, Jamaican patois accent, soulful deep bass, 75 BPM
Header: [Voice: Jamaican patois, warm bass, island rhythm]
Phonetic patterns: "the" → "da", "water" → "wata", hard D sounds, "cold" → "cole", open vowels
```

#### Southern Gothic / Deep South
```
Style: gothic Americana, deep south drawl, dark atmospheric, slide guitar
Header: [Voice: deep south drawl, slow and heavy, dark]
Phonetic patterns: "name" → "na-ame" (stretched), drop final consonants, "I'm" → "Ah'm", long vowels
```

#### Irish / Celtic
```
Style: Irish folk, Celtic accent, warm and lilting, acoustic
Header: [Voice: Irish accent, lilting, warm storytelling]
Phonetic patterns: "th" → "t" or "d", "my" → "me", soft consonants, rhythmic lilt
```

#### Cockney / London
```
Style: British punk, Cockney accent, raw and sharp
Header: [Voice: Cockney accent, sharp, street-level]
Phonetic patterns: drop H's ("'ello"), "th" → "f" ("fink"), glottal stops ("bo'le"), "my" → "moi"
```

### Combining with Vocal Range Control

Phonetic control and range headers are complementary layers — stack them:

```
[Deep bass-baritone male vocal, gravelly texture, Scottish Highland accent]
[Vocal range: E2-E4]
[Exclude: falsetto, soprano register, head voice, American accent, clean diction]

[Verse 1]
[Raspy, rolling Rs]
Ah'm walkin' awaa frae the hills o' ma yooth
The wind on ma face like a whisper o' truth
```

- **Range header** → controls pitch (what notes)
- **Phonetic spelling** → controls pronunciation (what mouth shape)
- **Style box** → controls character (what vibe)
- **Metadata header** → reinforces all three

### Phonetic Control Tips

1. **The AI will fight you** — it wants to correct back to standard English. Triple reinforcement (Style + header + phonetic lyrics) overcomes this
2. **Don't phonetically rewrite everything** — key emotional words and accent markers are enough. Over-phonetizing makes lyrics unreadable
3. **Test with one verse first** — phonetic spelling is experimental. Generate, listen, adjust the spelling
4. **Vowel length = note length** — `na-ame` holds longer than `name`. Use this to control phrasing
5. **Hard consonants = rhythmic attacks** — D, T, K, P create punch. Use them on downbeats
6. **ChatGPT is your translator** — don't guess phonetics. Use an LLM to convert, then adjust for singability
7. **Accent + range + phonetics = full vocal director stack** — each layer controls a different dimension of the performance
8. **This works best for character voices** — gritty pirates, regional folk, Southern blues, island vibes. For clean pop, standard spelling is fine

---

## Extreme Vocal Techniques — Suno Capability Map

Suno can simulate advanced vocal mechanics that take humans years to master — but not all techniques work equally well. This map shows what succeeds, what needs workarounds, and what fails. Use it to set expectations before attempting non-standard vocal styles.

### Tested Techniques

| Technique | Works? | Style Prompt Keys | Settings | Notes |
|-----------|--------|-------------------|----------|-------|
| **Carnatic / Indian classical** | Yes | `carnatic classical vocal, Indian classical singing, heavy gamaka ornamentation` | W:10% SI:85% | Microtonal slides (gamaka) and wavy note ornaments render well. Tambura drone appears in background. High style influence keeps it authentic. |
| **Opera / Bel canto** | Yes | `dramatic operatic vocal, powerful projection, strong vibrato` | W:10% SI:85% | Wide vibrato and diaphragm power translate. Sounds theatrical and physically pushed, not just "loud." |
| **Yodeling** | Yes | `traditional yodelling vocal, alpine folk style` | W:10% SI:85% | Suno simulates the chest-to-falsetto crack — the actual vocal cord flip. Use traditional yodeling syllables in lyrics. |
| **Jazz scat** | Yes | `jazz scat singing, bebop style, swing rhythm` | W:10% SI:85% | Nonsense syllables treated as instrumental phrases, not glitches. Swings with the rhythm section. Works especially well with female voice. |
| **Whistle register** | Yes | `powerful pop diva vocal, whistle register singing, ultra high pitch` | W:10% SI:85% | Piercing high-frequency tone achievable. Best at the end of phrases/tracks. The airy, zipped-cord quality comes through. |
| **Overtone / throat singing** | Yes | `overtone singing, Tuvan throat singing style, low drone` | W:10% SI:85% | Two simultaneous tones: deep false-chord drone + high whistling harmonic. Sounds organic and ancient, not synthesized. |
| **Haka (Māori war chant)** | No | `haka chanting style, loud aggressive group vocals, rhythmic stomps` | Any | **Fundamental limitation:** Suno is biased toward melody. It cannot produce pure rhythmic shouting without adding a cinematic tune. The result sounds like a movie trailer anthem, not a raw war dance. |

### Key Insights

1. **Suno understands physical vocal mechanics** — it can simulate the chest-falsetto flip (yodeling), the zipped-cord whistle register, the two-tone throat singing overtone. These aren't just "effects" — the AI models the physical behavior.

2. **Low weirdness + high style influence for authenticity** — every successful extreme technique test used ~W:10% SI:85%. High style influence locks the AI to the cultural tradition. High weirdness would add "creative" deviations that break the technique.

3. **Suno has a melodic bias** — it will always try to make things sound like a song. Pure rhythmic speech, shouting, or non-melodic vocalization gets "musicalized." This is a hard limitation, not a prompting problem.

4. **Nonsense syllables work** — scat syllables, random numbers, gibberish — Suno treats them as musical material and sings them with full emotional commitment. Meaning doesn't matter; syllable shape does.

5. **Use with vocal range headers** — combine extreme techniques with range control for precision. Throat singing benefits from `[Vocal range: C2-E3]`. Whistle register from `[Vocal range: C5-C6]`. Opera from the full range but with `[Exclude: pop, breathy, intimate]`.

### When to Use This Map

- User requests a non-standard vocal style → check the map before promising results
- If the technique is in the "Yes" column → use the documented style prompt and settings as a starting point
- If the technique involves pure non-melodic vocalization (haka, drill sergeant, auction caller) → warn the user about the melodic bias
- For unlisted techniques → test with low weirdness / high style influence first, then adjust

---

## Gender + Age Descriptors

### Female Vocals
```
female vocals, soft female vocals, powerful female vocals
young female voice, mature female vocals
soprano female, alto female, mezzo-soprano
```

### Male Vocals
```
male vocals, smooth male vocals, raspy male vocals
deep male voice, young male vocals, baritone
tenor male vocals, bass male vocals
```

### Other
```
mixed vocals, choir, harmonies
androgynous vocals, gender-neutral voice
call and response, vocal ensemble
```

---

## Delivery Styles

### Soft/Gentle
| Style | Description | Best For |
|-------|-------------|----------|
| **Breathy** | Airy, intimate, ASMR-like | Ballads, ambient |
| **Whispered** | Hushed, secretive, close | Intimate, emotional |
| **Soft** | Gentle, restrained, warm | Lullabies, peaceful |
| **Tender** | Caring, delicate, loving | Romantic, emotional |
| **Vulnerable** | Exposed, honest, fragile | Confessional, sad |
| **Airy** | Light, floating, ethereal | Dreamy, ambient |

### Powerful/Strong
| Style | Description | Best For |
|-------|-------------|----------|
| **Powerful** | Full voice, commanding | Anthems, rock |
| **Belting** | Sustained high notes, chest | Power ballads |
| **Soaring** | Rising, expansive, epic | Climactic moments |
| **Commanding** | Authoritative, strong | Anthems |
| **Passionate** | Intense emotion, conviction | Emotional peaks |
| **Fiery** | Aggressive passion | Rock, soul |

### Textured
| Style | Description | Best For |
|-------|-------------|----------|
| **Raspy** | Rough, textured, gritty | Rock, blues |
| **Smoky** | Dark, mysterious, jazz-club | Jazz, noir |
| **Gravelly** | Very rough, deep | Rock, soul |
| **Gritty** | Raw, unpolished | Garage, punk |
| **Husky** | Low, warm roughness | Blues, soul |
| **Growling** | Aggressive texture | Metal, rock |

### Smooth
| Style | Description | Best For |
|-------|-------------|----------|
| **Smooth** | Polished, silky, flowing | R&B, jazz |
| **Silky** | Extra smooth, luxurious | Soul, quiet storm |
| **Velvety** | Rich, dark smooth | Jazz, R&B |
| **Sultry** | Seductive, warm | R&B, romantic |
| **Honeyed** | Sweet, warm, golden | Soul, folk |
| **Crooning** | Classic smooth style | Jazz standards |
| **Mellow** | Relaxed, warm, unhurried | Lo-fi, chill, acoustic |
| **Laid-back** | Easy, effortless, casual | Reggae, surf rock, bossa |

### Expressive
| Style | Description | Best For |
|-------|-------------|----------|
| **Dramatic** | Theatrical, intense dynamics | Cinematic, orchestral pop |
| **Playful** | Cheeky, bouncy, lighthearted | Pop, novelty, kids |
| **Confident** | Assured, bold, self-possessed | Hip-hop, anthems, pop |
| **Robotic** | Mechanical, synthetic, inhuman | Electronic, industrial |

---

## Vocal Techniques

### Classical
| Technique | Description |
|-----------|-------------|
| **Operatic** | Classical trained, powerful vibrato |
| **Bel canto** | Beautiful singing, Italian style |
| **Vibrato** | Controlled pitch oscillation |
| **Head voice** | Upper register, lighter |
| **Chest voice** | Lower register, fuller |
| **Falsetto** | High, airy male vocals |

### Contemporary
| Technique | Description |
|-----------|-------------|
| **Belting** | Powerful sustained notes |
| **Runs/Melisma** | Many notes per syllable |
| **Ad-libs** | Improvised vocal flourishes |
| **Vocal fry** | Creaky low register |
| **Scatting** | Jazz improvisation, nonsense syllables |
| **Yodeling** | Rapid register switches |

### Extended/Experimental
| Technique | Description |
|-----------|-------------|
| **Spoken word** | Rhythmic speech, not singing |
| **Chanting** | Repetitive, ritualistic |
| **Overtone singing** | Multiple tones simultaneously |
| **Throat singing** | Tuvan/Mongolian style |
| **Wordless vocals** | Vocalizing without lyrics |
| **Humming** | Closed-mouth melody |

---

## Vocal Processing

### Reverb/Space
```
dry vocals - No reverb, intimate and close
reverb-heavy - Spacious, cathedral-like
room reverb - Natural space, live feel
plate reverb - Vintage, smooth
hall reverb - Concert hall ambience
```

### Effects
```
Auto-Tuned - Pitch corrected, modern
vocoder - Robotic, synthesized
talk box - Funky, electronic
distorted - Gritty, aggressive
telephone effect - Lo-fi, filtered
echo/delay - Repeating, spacious
chorus - Thickened, doubled
harmonizer - Parallel harmony, lush
double-tracked - Two takes layered, wide
slapback - Short echo, rockabilly
bitcrush - Digital lo-fi, crunchy
formant shift - Altered vowel character
pitch shift - Raised or lowered pitch
stutter - Rhythmic glitch, choppy
gate - Chopped sustain, rhythmic
phaser - Sweeping phase, psychedelic
flanger - Jet swoosh, metallic
```

### Character
```
vintage - Old recording character
modern - Clean, contemporary
lo-fi - Degraded, tape-like
pristine - Crystal clear
processed - Obviously effected
natural - Unprocessed, organic
```

---

## Vocal Direction Tags (use in lyrics)

```
[Soft vocals]
[Whispered]
[Spoken word]
[Powerful vocals]
[Belting]
[Falsetto]
[Harmonies]
[A cappella]
[Call and response]
[Ad-libs]
[Humming]
[Vocalizing]
```

For structure tags (`[Intro]`, `[Verse]`, `[Chorus]`, etc.) and dynamic tags (`[Crescendo]`, `[Climax]`, etc.), see `production.md` and `track-template.md`.

---

## Vocal Arrangements

### Solo
```
solo vocals - Single voice
lead vocals - Primary singer
```

### Harmony
```
harmonies - Multiple vocal lines
two-part harmony - Simple harmony
three-part harmony - Rich harmony
stacked harmonies - Layered voices
tight harmonies - Close intervals
wide harmonies - Spread intervals
```

### Ensemble
```
choir - Large vocal group
gospel choir - Churchy, powerful
backing vocals - Support vocals
gang vocals - Shouted group
call and response - Leader and group
unison - Everyone same note
```

---

## Genre-Specific Vocal Styles

### Pop
```
Bright, polished, catchy delivery, clear diction, hook-focused, radio-ready
```

### R&B/Soul
```
Smooth runs, melismatic, sensual delivery, gospel influence, emotional depth
```

### Rock
```
Powerful, raspy, energetic, raw emotion, stadium-ready, anthemic
```

### Hip-Hop/Rap
```
Rhythmic flow, confident delivery, varied cadence, Auto-Tune melodic
```

### Jazz
```
Scatting, sophisticated phrasing, behind-the-beat, smoky, intimate
```

### Folk/Indie
```
Natural, unpolished, intimate, storytelling focus, conversational
```

### Electronic/EDM
```
Processed, chopped, vocoder, distant, layered, euphoric
```

### Country
```
Twangy, storytelling, drawl, emotional sincerity, clear diction
```

### Metal
```
Growling, screaming, powerful clean, operatic, aggressive
```

---

## Quick Reference: Mood → Vocal Style

| Mood | Recommended Vocal Style |
|------|------------------------|
| Sad/Emotional | Soft, vulnerable, breathy, with breaks |
| Happy/Uplifting | Bright, powerful, energetic, soaring |
| Romantic | Smooth, intimate, tender, whispered |
| Energetic | Powerful, commanding, fiery |
| Peaceful | Soft, airy, gentle, humming |
| Dark | Smoky, mysterious, low, haunting |
| Epic | Powerful, operatic, soaring, choir |
| Nostalgic | Warm, honeyed, gentle, reflective |

---

## Vocal Track Workflow

When the user wants a **vocal-forward track** (not instrumental), follow this workflow:

### 1. Style Prompt: Lead with Vocals

Put the vocal description early in the Style field — Suno weights earlier descriptors more heavily.

```
Smooth male R&B vocals, silky falsetto, intimate delivery, neo-soul, warm Rhodes piano, mellow bass groove, F Major, 78 BPM, late-night bedroom vibes
```

**Formula for vocal tracks:**
```
[VOCAL DESCRIPTION] + [GENRE] + [INSTRUMENTS] + [MOOD] + [TECHNICAL SPECS]
```

### 2. Vocal Description Checklist

Include at least 3 of these in every vocal Style prompt:

| Element | Example |
|---------|---------|
| **Gender/range** | "female alto vocals", "deep male baritone" |
| **Delivery style** | "breathy", "powerful", "raspy", "smooth" |
| **Technique** | "with runs", "falsetto moments", "vocal fry" |
| **Processing** | "reverb-heavy", "dry and intimate", "Auto-Tuned" |
| **Emotional quality** | "vulnerable", "confident", "haunting" |

### 3. Lyrics: Write Real Lyrics (Not Placeholders)

For vocal tracks, Suno needs **actual words** — not structural descriptions like "[Melody rises gently]". Those placeholders are for instrumental tracks only.

**Instrumental lyrics (for background music):**
```
[Verse]
[Soft drums join]
[Mellow bassline]
[Dreamy melody floats]
```

**Vocal lyrics (for songs with singing):**
```
[Verse 1]
I left the light on by the door
Hoping you'd find your way back home
The coffee's cold, the record skipped
But I'm still sitting by the phone
```

### 4. Use Vocal Direction Tags in Lyrics

Sprinkle vocal direction tags between sections to control performance:

```
[Verse 1]
[Soft vocals]
I left the light on by the door
Hoping you'd find your way back home

[Pre-Chorus]
[Slowly building]
Every shadow looks like you
Every sound could be your step

[Chorus]
[Powerful vocals]
But I'm not waiting anymore
I'm not keeping score
```

### 5. Lyric Writing Tips for Suno

- **Syllable count matters** — match syllable density to BPM. Fast songs need shorter punchy lines, ballads can flow
- **Repeat choruses** — Suno handles repetition well and it reinforces hooks
- **Use "ooh", "aah", "mmm"** for vocal textures between sections
- **Keep verses 4-6 lines** — too long and Suno may rush through them
- **Contrast verse/chorus energy** — quiet verses + big choruses = drama
- **End lines with strong vowels** (oh, ay, ee) for better melodic moments

### 6. Style Prompt Examples (Vocal Tracks)

**Pop ballad:**
```
Emotional female pop vocals, powerful chest voice, intimate verses building to soaring chorus, piano-driven pop ballad, strings, A Minor, 72 BPM, heartbreak anthem, radio-ready production
```

**R&B:**
```
Smooth male R&B vocals, silky delivery, melismatic runs, neo-soul groove, warm Rhodes piano, deep bass, D Minor, 85 BPM, late-night sensual atmosphere, 90s golden era feel
```

**Indie folk:**
```
Natural female vocals, unpolished and intimate, conversational delivery, acoustic fingerpicked guitar, soft cello, G Major, 95 BPM, campfire storytelling, raw and honest
```

**Rock anthem:**
```
Raspy powerful male vocals, stadium-ready delivery, heavy distorted guitars, driving drums, E Minor, 130 BPM, 90s alternative rock, anthemic chorus, raw emotion
```

**Hip-hop:**
```
Confident male rap vocals, rhythmic flow with melodic Auto-Tune hook, trap drums, 808 bass, dark synth pads, A Minor, 140 BPM, modern hip-hop, street anthem energy
```

---

## Instrumental-Only Prompts

For background music, ALWAYS use:

```
[Instrumental] ... [No Vocals]
```

But you can still guide the "voice" of the music with instrumental lyrics:

```
[Intro]
[Instrumental - soft piano enters]

[Verse]
[Melody rises gently]
[Strings join softly]

[Chorus]
[Full arrangement, soaring]
[Emotional peak]

[Outro]
[Fading with reverb]
```

This guides Suno's structure without introducing actual vocals.
