# Advanced Meta Tags Reference

<!-- TOC
| Section | Line | Keywords |
|---------|------|----------|
| Lyric Formatting Symbols (Performance Notation) | 46 | formatting, symbols, parentheses, tilde, vibrato, caps, dash, hyphen, ellipsis, pause, sustain, background vocal, notation, vowel stretching |
| Parentheses Critical Rule | 61 | parentheses, instructions, brackets, background vocal, wrong, correct |
| ALL CAPS Rules | 72 | caps, loud, forceful, emphasis, shouting |
| Advanced Structure Tags | 80 | structure tags, final chorus, chorus x2, fade out, big finish, beat switch, hook first |
| Structure Tag Placement Rules | 107 | placement, section, stacked, anchor, drift |
| Tag Length and Reliability Rules | 115 | tag length, reliability, short tags, verbose tags, proven, ignored |
| Vocal Delivery Tags (Verified) | 133 | vocal, delivery, whispered, belted, falsetto, raspy, soulful, breathy, operatic, rapped, growled, personas |
| Duet Protocol (Community-Verified) | 161 | duet, voice consistency, singer, male, female, both, verse, chorus, gender |
| Duet Stability Rule | 198 | duet, stability, verse, switching, consistency |
| Ad-Lib Tags | 204 | adlib, ad-lib, boom, clap, hey, yeah, whoa, uh, ayy, ok, hip-hop, trap |
| Scream and Growl Formatting | 235 | scream, growl, metal, punk, hardcore, caps, vowel |
| Pipe Stacking Syntax | 245 | pipe, stacking, modifiers, era, anchor, section, AND operator |
| Pipe Stacking Rules | 254 | pipe, rules, core element, modifiers, era, sections |
| Producer-Level Category Tags | 271 | producer, category, instrumentation, vocal, mix, bar count |
| Atmosphere and Environmental Tags | 297 | atmosphere, environmental, stadium, crowd, concert, live |
| Human and Crowd Sounds | 311 | applause, cheering, crowd, chanting, stadium, clapping, laughter, sighs, whistling |
| Mechanical and Electronic Sounds | 327 | phone, beeping, bell, static, record scratch |
| Musical Effect Tags | 337 | silence, censored, fade, stop, drum fill, crossfade |
| Production and Effect Tags Extended | 350 | effect, lo-fi, reverb, delay, distortion, sidechain, bitcrusher, autopan, radio filter, texture, grainy |
| Performance Dynamics Tags | 370 | dynamics, soft verse, power chorus, half-time, riser, bass drop, pinch harmonics, whammy, palm mute, pedal steel, gang shouts |
| Genre Recipe Stacks | 387 | genre, recipe, stacks, edm, pop, trap, punk, country, cinematic, metal, gospel |
| EDM Festival Banger Recipe | 391 | edm, festival, riser, drop, sidechain, stereo |
| Pop Radio Hit Recipe | 401 | pop, radio, hook, bright, harmonies, anthemic |
| Trap Hip-Hop Recipe | 410 | trap, hip-hop, autotune, 808, ad-libs, hi-hats |
| Pop Punk Alt Rock Recipe | 418 | pop punk, alt rock, raspy, palm mute, anthemic, distorted |
| Country Recipe | 427 | country, banjo, acoustic, pedal steel, baritone |
| Cinematic Ambient Recipe | 435 | cinematic, ambient, drone, whispers, orchestra, choir, rain |
| Hard Rock Metal Recipe | 449 | hard rock, metal, power riff, distortion, solo, breakdown |
| Gospel Recipe | 461 | gospel, SATB, choir, organ, call response, soul |
| Theory-to-Tag Translation Guide | 471 | theory, translation, genre, tag, equivalent, rock, metal, electronic, jazz, folk, cinematic |
| Fusion Anti-Pairs | 502 | fusion, anti-pair, avoid, cinematic dark, opera melodic, dnb funk, incoherence |
| Strong Fusion Pairs | 514 | fusion, strong, pairs, rap trap, lo-fi chill, metal rock, orchestral epic, soul rnb, synthwave |
| Front-Loading and Weight Rules | 533 | front-loading, weight, signal, first words, style prompt, meta tags, lyrics, formatting, settings |
| What Goes Where | 547 | placement, genre, bpm, key, mood, style prompt, lyrics field, section tags, vocal, instruments, production, atmosphere |
| Genre-Specific Gotchas | 562 | punk, progressive metalcore, dynamics, shorter songs, genre tag, gotchas |
| Instrument Isolation (Sandwich Method) | 573 | sandwich, isolation, instrument, exclude styles, solo instrument |
TOC -->

---

## Lyric Formatting Symbols (Performance Notation)

These are non-bracket symbols placed inside/around lyrics that change how Suno performs the text. NOT tags — performance notation.

| Symbol | What Suno Does | Example | Verification |
|--------|---------------|---------|--------------|
| ( ) Parentheses | Background/backing vocal layer — softer, echoed, ad-lib | (I'm still here) | Confirmed |
| - Dash/Hyphen | Stretch syllables, break words, spell out letters. For sustained notes, repeat vowels with hyphens: `lo-o-o-ove` | al-most, G-A-L-A-X-Y, lo-o-o-ove | Confirmed |
| ALL CAPS | Louder, more forceful — use 1-3 words max per section | WE RISE together | Confirmed |
| ~ Tilde | Experimental — may add vibrato or note hold, but results vary. Only one source mentions it. Prefer hyphenated vowel stretching (`lo-o-o-ove`) for reliable sustain | ho~me | Unverified — use with caution |
| " " Quotation marks | May nudge toward spoken/whispered delivery, but unverified. For reliable spoken/whispered delivery, use explicit tags: `[Spoken Word]`, `[Whispered]` | "you were never there" | Unverified — prefer explicit tags |
| ... Ellipsis | Pauses and trailing effects — more dots = longer pause, NOT longer sustain. For sustained notes, use hyphens: `lo-o-o-ove` | Whispers... | Confirmed (for pauses) |

---

### Parentheses Critical Rule

**( ) parentheses are SUNG/PERFORMED by Suno** — they create background vocal layers. NEVER put production instructions inside parentheses. Use [ ] brackets for all instructions.

- CORRECT: [spoken word] then lyrics on next line
- WRONG: (say this in a spoken voice)
- CORRECT: (I'm still here) — sung as soft echo
- WRONG: [Chorus] (This is the big hook line) — parenthesized text becomes background vocal

---

### ALL CAPS Rules

- Use 1-3 words in ALL CAPS per section maximum
- More dilutes the effect
- Works best on one key word per line

---

## Advanced Structure Tags

Beyond the basics ([Intro], [Verse], [Chorus], etc.), these give finer structural control. Note: many "advanced" tags from viral guides are unverified. This list separates what works from what doesn't.

**Verified tags:**

| Tag | Purpose | Notes |
|-----|---------|-------|
| [Outro: Fade out] | Explicit fade-out ending | Use as two stacked tags `[Outro]` + `[Fade Out]` for most reliable results |
| [Outro: Big finish] | Explicit crescendo ending | Same — stacking `[Outro]` + `[Big finish]` is safer |
| [Break] | Strips arrangement momentarily | Use instead of verbose alternatives like [Band drop-out before final chorus] |
| [Breakdown] | Extended stripped section | Standard and reliable |

**Natural-language hints (may nudge Suno but not guaranteed):**

| Tag | What it tries to do | Reality |
|-----|-------------------|---------|
| [Final Chorus] | Signal the climactic last chorus | May work — Suno sometimes responds to "Final" as an energy cue |
| [Beat switch] | Tempo or rhythmic feel change | Sometimes works as a hint, not a command |
| [Crowd-call section] | Audience call-and-response | More of a lyric/style suggestion than a structural command |

**Section ordering tip:** To open a song with the hook before a verse, simply place [Hook] or [Chorus] as the first section in the lyrics field. No special tag needed.

**Removed (unverified/hallucinated):** The following tags appeared in viral guides but have no evidence of working in Suno: `[Callback: Chorus melody]`, `[Hook Loop]`, `[Hook delay]`, `[Band drop-out before final chorus]`, `[Emotional release]`. For the effects these describe, use standard tags: `[Break]` for arrangement strip-outs, `[Build]` for tension, `[Chorus]` repeated for chorus repetition.

**On [Chorus x2]:** The multiplier syntax is unverified. For a repeated chorus, write `[Chorus]` twice with the lyrics repeated.

### Structure Tag Placement Rules

- Place the tag at the START of its section, before any lyrics
- Every section should begin with one or more stacked meta tags
- Do not skip section tags — Suno uses them as structural anchors to prevent drift

---

## Tag Length and Reliability Rules

Short, simple tags on their own lines are the most reliable way to control Suno's output.

| Tag Style | Reliability | Example |
|-----------|-------------|---------|
| Short (1-3 words), own line | High — proven reliable | `[Break]`, `[Whispered]`, `[Guitar solo]` |
| Pipe-stacked (4-6 modifiers) | Medium — useful for section headers | `[Verse \| 60s jangly guitar \| clean Fender tone]` |
| Verbose compound (4+ words, descriptive) | Low — risks being sung as lyrics or ignored | `[Band drop-out before final chorus]` |

**Rules:**
- Default to short tags on their own lines for structural and vocal control
- Use pipe stacking for section-level instrument/production cues
- Never use verbose descriptive phrases as standalone tags — they often get interpreted as lyrics
- If you need a complex effect, break it into multiple short stacked tags

---

## Vocal Delivery Tags (Verified)

Suno's **Personas** feature is a UI tool — you save vocal characteristics from existing songs and select them from a dropdown. Personas are NOT inline tags. Tags like `[Whisper Soul]`, `[Power Praise]`, `[Retro Diva]`, `[Persona: Pop Star]`, and `[Vocal Style: ...]` are unverified and likely hallucinated from viral guides.

For vocal delivery control, use these verified inline tags:

| Tag | Description |
|-----|-------------|
| [Whispered] | Soft, intimate whispering |
| [Belted] | Full-voice powerful singing |
| [Falsetto] | High, head-voice register |
| [Raspy] | Gritty, weathered vocal texture |
| [Soulful] | Emotionally rich, R&B-influenced |
| [Breathy] | Airy, close-mic intimate |
| [Operatic] | Classical vocal technique |
| [Rapped] | Rhythmic spoken delivery |
| [Melodic Rap] | Sing-rap hybrid |
| [Growled] | Low, aggressive vocal |
| [Harmonies] | Stacked vocal layers |
| [Ad-libs] | Background vocal interjections |
| [Melisma] | Complex vocal runs |
| [Vibrato] | Sustained pitch oscillation |
| [Spoken Word] | Spoken, non-sung delivery |

These are short, 1-2 word tags that Suno reliably interprets. Place at the start of a section before lyrics.

---

## Duet Protocol (Community-Verified)

Duets in Suno are "more of an exploit than a feature" (Suno Wiki). Voice consistency is hard to maintain, but these community-tested approaches improve results:

**Approach 1 — Per-section speaker labels (most reliable):**
```
[Verse 1]
[Male]
Walking down that road alone...

[Chorus]
[Both]
We were fire and rain...

[Verse 2]
[Female]
I never thought you'd stay...
```

**Approach 2 — Style prompt context:**
```
Male and female duet, [genre], [other style tags]
```
Adding gender context to the Style prompt can help Suno maintain two distinct voices.

**Approach 3 — Named characters (may help, not guaranteed):**
```
[John]
Walking down that road alone...

[Jane]
I never thought you'd stay...
```
Named characters MAY help Suno distinguish voices, but this is not a verified protocol. Per-section `[Male]`/`[Female]`/`[Both]` labels are more reliable.

**V5 UI note:** Suno V5 has a voice gender selector in the generation UI. Use it alongside per-section labels for best results.

### Duet Stability Rule

Assign full verses to one singer rather than alternating line-by-line. Voice consistency breaks when switching mid-verse.

---

## Ad-Lib Tags

Ad-libs add bounce, attitude, and organic energy between lyric lines. Best in hip-hop, trap, pop, R&B.

**Format:**
```
[adlib TAG]
SOUND IN ALL CAPS
```

Place on its own line between lyric lines, on the beat:
```
[Verse | autotuned delivery]
Running through the city lights
[adlib HEY]
Nothing ever slows me down
[adlib UH UH]
Still the same, still alive
```

| Tag | Sound |
|-----|-------|
| [adlib boom] | Boom/impact hit |
| [adlib clap] | Hand clap |
| [adlib hey] | "HEY" shout |
| [adlib yeah] | "YEAH" |
| [adlib whoa] | "WHOA" |
| [adlib uh] | "UH" rhythmic filler |
| [adlib ayy] | "AYY" |
| [adlib ok] | "OK" |

### Scream and Growl Formatting

For metal, punk, and hardcore:

- Write scream text in ALL CAPS and stretch the vowel: `AAAAAH` not just `AH`
- More vowel letters = harder-hitting delivery
- Combine with [Growl] or [Scream] tag

---

## Pipe Stacking Syntax

Stack multiple cues using | inside a single bracket. Acts as an AND operator.

**Format:**
```
[core element | era/genre | tone/mix | quirk detail]
```

### Pipe Stacking Rules

1. Lead with the core element or section label
2. 4-6 modifiers maximum — more creates noise
3. Use era anchors for genre accuracy (60s, 80s, 90s)
4. Match stacks to sections — each section gets its own stack

**Examples:**
```
[Verse | 60s jangly guitar rhythm | clean Fender tone | bright treble EQ | light spring reverb | tambourine hits]
[Guitar solo | 80s glam metal lead guitar | heavy distortion | pinch harmonics | wide stereo | whammy bar bends]
[Drop | sidechained synth bass | layered white noise riser | sub drop impact | stereo delay tail]
[Chorus | anthemic chorus | stacked harmonies | modern pop polish | bass drop]
```

---

## Producer-Level Category Tags

For maximum precision, use labeled category tags at the start of each section:

```
[Instrumentation: overdriven punk guitar | palm-muted power chords | fast downstrokes]
[Vocal: raspy lead vocal | gang shouts on last line]
[Mix: mono bass, stereo guitar | slight room reverb]
```

**Rules:**
- Keep everything inside one set of brackets per category
- Use short, production-ready terms — no storytelling inside tags
- Place before any lyrics in the section
- Bar counts work sparingly: `[8 bar guitar solo | fast alternate picking | ride cymbal wash]`

**Full Section Example:**
```
[Chorus | Instrumentation: full punk band | overdriven guitars | bass + kick driving eighth notes]
[Vocal: anthemic chorus | stacked harmonies | crowd-style vocals | small hall reverb]
I wanna burn through every limit
Nothing left to hold me back
```

---

## Atmosphere and Environmental Tags

### Concert / Stadium Atmosphere

**Stadium Intro:**
```
[Intro: stadium crowd ambience | big applause | cheering | distant chanting | hey | stage reverb]
```

**Stadium Outro:**
```
[Outro: crowd fade | distant chanting | stadium echo | applause]
```

### Human and Crowd Sounds

| Tag | Sound |
|-----|-------|
| [Applause] | Clapping |
| [Cheering] | Crowd excitement |
| [Crowd noise] | General audience |
| [Distant chanting] | Chant echoing back |
| [Stadium ambience] | Full stadium sound |
| [Stage reverb] | Performance room feel |
| [Clapping] | Hand claps |
| [Chuckles] / [Giggles] | Soft laughter |
| [Audience laughing] | Full crowd laughter |
| [Sighs] | Breathing out |
| [Whistling] | Tuneful whistling |

### Mechanical and Electronic Sounds

| Tag | Sound |
|-----|-------|
| [Phone ringing] | Telephone |
| [Beeping] | Electronic beeps |
| [Bell dings] | Bell chime |
| [Static] | White noise |
| [Record scratch] | Vinyl scratch |

### Musical Effect Tags

| Tag | Effect |
|-----|--------|
| [Silence] | Complete quiet — dramatic pause |
| [Censored] | Bleeped content |
| [Fade] | Volume decrease |
| [Stop] | Abrupt ending |
| [Drum fill transition into chorus] | Drum fill bridging sections |
| [Smooth crossfade intro to verse] | Crossfade transition |

---

## Production and Effect Tags (Extended)

Beyond what's in production.md, these are additional effect tags. Use standalone short tags rather than "Effect:" prefixed versions — short tags are more reliably interpreted by Suno.

| Tag | Effect |
|-----|--------|
| [Lo-fi] | Dusty vinyl crackle |
| [Reverb] or [Hall Reverb] | Spacious echo (use [Hall Reverb] for massive space) |
| [Delay] or [Ping-pong Delay] | Echoes (use [Ping-pong Delay] for stereo bounce) |
| [Distortion] | Heavy clipping |
| [Sidechain] | Pumping volume (house music) |
| [Bitcrusher] | Digital degradation, 8-bit |
| [Autopan] | Sound panning left to right |
| [Radio Filter] | Narrow speaker sound |
| [Texture: Grainy] | Cracked/analog feel |
| [Sidechained synth bass] | Pumping EDM bass |
| [Layered white noise riser] | Build tension with noise |
| [Sub drop impact] | Bass drop hit |
| [Stereo delay tail] | Wide echo fade |

### Performance Dynamics Tags

| Tag | Use |
|-----|-----|
| [Soft verse \| low vocal intensity] | Low-energy verse |
| [Power chorus \| full band] | Maximum energy chorus |
| [Half-time breakdown \| minimal instrumentation] | Stripped half-time feel |
| [Riser build] | Ascending tension before drop |
| [Bass drop] | Low-end impact point |
| [Pinch harmonics] | Guitar squeal (glam/metal) |
| [Whammy bar bends] | Guitar dive/bend |
| [Palm-muted guitars] | Tight, chunky rhythm guitar |
| [Pedal steel guitar] | Country characteristic |
| [Gang shouts on last line] | Group shout on final line |

---

## Genre Recipe Stacks

Pre-built stacked meta tag sets for common genres. Copy directly into sections.

### EDM / Festival Banger

```
[Intro | wide stereo pads | layered white noise riser | atmospheric]
[Verse | melodic hook | airy female vocal | reverb tail | octave harmony stack]
[Build-Up | riser build | synth stabs | kick sidechain build]
[Drop | explosive drop | sidechained synth bass | sub drop impact | stereo delay tail]
[Chorus | chant vocals | crowd-style vocals | wide stereo]
```

### Pop / Radio Hit

```
[Intro | hook first | melodic hook | bright vocal EQ | layered harmonies]
[Verse | emotional build-up | warm piano chords | subtle kick pulse]
[Pre-Chorus | riser build | modern pop polish]
[Chorus | anthemic chorus | stacked harmonies | wide stereo pads | light snare reverb]
```

### Trap / Hip-Hop

```
[Intro | autotuned delivery | tuned male vocal | light reverb]
[Verse | chant hook | layered ad-libs | stereo panning echoes]
[Chorus | syncopated drop | 808 sub bass | off-beat hi-hats | snare rolls]
```

### Pop Punk / Alt Rock

```
[Verse | raspy lead vocal | youthful tone | light vocal grit]
[Pre-Chorus | emotional build-up | palm-muted guitars]
[Chorus | anthemic chorus | stacked harmonies | palm-muted guitars | driving kick-snare beat]
[Bridge | bridge with drop | distorted power chords | bass slide-in]
```

### Country

```
[Intro | spoken word verse | warm baritone EQ | slight drawl]
[Verse | banjo plucks | acoustic strums | light tambourine]
[Chorus | anthemic chorus | stacked harmonies | pedal steel guitar | light snare reverb]
```

### Cinematic / Ambient

```
[Intro]
[Vocal drone]
(deep resonant)
Whispers...

[Verse | cinematic vocal pacing | sparse instrumentation | wide stereo pads | piano]
[Build | string swells | taiko drums | rising orchestral tension]
[Chorus | full orchestra | choir vocals | epic]
[Outro | fade out | ambient | rain | distant echo]
```

### Hard Rock / Metal

```
[Intro | power riff | heavy distortion | palm-muted guitar | drum fill]
[Verse | raspy lead vocal | overdriven guitar | bass driving eighth notes]
[Pre-Chorus | build-up | rising tension | cymbal crescendo]
[Chorus | anthemic chorus | stacked harmonies | full band | crowd-style vocals]
[Solo | guitar solo | 80s glam metal lead guitar | heavy distortion | wide stereo | whammy bar bends]
[Bridge | breakdown | half-time breakdown | minimal instrumentation]
[Chorus | anthemic chorus | full band return | stacked harmonies | explosive]
```

### Gospel

```
[Chorus | Multiple voice chorus s a t b | organ | call and response | soul | uplifting]
```

SATB Rule: Use [Multiple voice chorus s a t b] only on the chorus — not throughout. Using it everywhere removes the impact.

---

## Theory-to-Tag Translation Guide

Direct mapping of music theory concepts to Suno tag equivalents per genre:

| Genre | Theory Element | Suno Tag Equivalent |
|-------|---------------|-------------------|
| Classic Rock | Jangly clean guitar, backbeat | [60s jangly guitar rhythm \| clean Fender tone \| tambourine hits] |
| Hard Rock | Palm-muted power chords | [palm-muted power chords \| distorted guitar \| driving kick-snare] |
| Glam Metal | Pinch harmonics, whammy bar | [80s glam metal lead guitar \| pinch harmonics \| whammy bar bends \| heavy distortion] |
| Thrash Metal | Gallop rhythm, fast 16ths | [thrash metal \| fast gallop \| double bass drums \| palm-muted riffs] |
| Death Metal | Death growls, blast beats | [death growl \| guttural vocal \| blast beats \| downtuned guitars] |
| Black Metal | Tremolo picking, shrieks | [tremolo guitar \| blast beats \| shrieked vocals \| icy production \| lo-fi black metal] |
| Power Metal | Operatic tenor, fast double bass | [operatic \| double bass drums \| fast gallop \| twin lead guitars \| epic \| symphonic] |
| Nu Metal | Syncopated groove, rap+scream | [nu metal \| syncopated riff \| rap verse \| screamed chorus \| downtuned bass] |
| Sludge Metal | Slow crushing groove | [sludge metal \| slow \| heavy \| feedback \| growl \| downtuned guitar \| lo-fi] |
| Progressive Rock | Odd time, Mellotron | [progressive rock \| 7/8 \| Mellotron \| keyboard \| complex structure] |
| Psychedelic Rock | Fuzz, wah-wah | [psychedelic rock \| fuzz guitar \| wah-wah \| sitar \| heavy reverb \| dreamy] |
| Grunge | Loud/soft dynamic | [grunge \| distorted guitar \| raw \| raspy lead vocal \| loud verse] |
| Shoegaze | Wall of reverb | [shoegaze \| wall of reverb \| dreamy \| ethereal \| guitar-heavy \| lo-fi] |
| Emo | Clean twinkle guitar | [emo \| clean guitar arpeggio \| emotional \| raw vocals \| heartfelt] |
| Lo-fi Hip Hop | Vinyl crackle, jazz chops | [lo-fi hip hop \| vinyl hiss \| mellow \| jazz samples \| chill \| slow] |
| Deep House | Sidechain pump | [deep house \| sidechained bass \| 4/4 \| warm bassline \| smooth \| deep] |
| Synthwave | Supersaw, analog | [synthwave \| supersaw \| analog synth \| 80s \| retrowave \| dark \| driving] |
| Trap | 808, hi-hat triplets | [trap \| 808 bass \| hi-hats \| autotuned delivery \| dark \| aggressive] |
| Gospel | SATB choir, call-response | [gospel \| SATB choir \| call and response \| soul \| uplifting \| organ] |
| Bossa Nova | Nylon guitar, sway | [bossa nova \| nylon guitar \| soft female vocal \| jazz \| smooth \| laid-back shuffle] |
| Flamenco | Phrygian, fast guitar | [flamenco \| Spanish guitar \| dark \| fast guitar \| Phrygian \| clapping] |
| Cinematic | Taiko, string swells | [cinematic \| full orchestra \| string swells \| taiko drums \| epic \| wide stereo pads] |

---

## Fusion Anti-Pairs

High-value fusion pairs are in `advanced-techniques.md`. These are the ANTI-pairs — combinations that community testing shows produce incoherent results:

### Avoid These Combinations

| Genre A | Genre B | Why It Fails |
|---------|---------|-------------|
| Cinematic | Dark (as genre tag) | "Dark" as a genre tag overrides cinematic dynamics, flattening the arrangement. Use mood descriptors instead: `[cinematic \| haunting \| ominous]` |
| Opera | Melodic | Suno can't resolve operatic technique with generic "melodic" — too vague. Specify the melody character: `[opera \| lyrical vocal line \| soaring]` |
| Drum and Bass | Funk | BPM incompatibility — DnB's 170+ BPM destroys funk groove (100-120 BPM) entirely |

### Strong Fusion Pairs

Community-observed natural pairings that consistently produce coherent results:

| Genre A | Genre B | Notes |
|---------|---------|-------|
| Rap | Trap | Strongest co-existing pair |
| Lo-fi | Chill | Dominant lo-fi pairing |
| Metal | Rock | Core metal anchor |
| Orchestral | Epic | Cinematic sweet spot |
| Emo | Emotional | Highest emo combination |
| Soul | R&B | Natural soul pairing |
| Synthwave | Synth | Core synthwave anchor |
| House | Deep | Deep house standard |
| Folk | Acoustic | Folk anchor |
| Jazz | Funk | Natural jazz pairing |

---

## Front-Loading and Weight Rules

Suno uses a layered signal system — tags are weighted signals, not guaranteed instructions.

**The #1 Rule:** The first few descriptors of the Style prompt carry the most weight. Front-load genre, primary instrument, and key mood.

| Layer | Location | Controls |
|-------|----------|----------|
| Style Prompt Box | Top style field | Overall sonic lane, genre, tempo, key, texture |
| Meta Tags | Inside lyrics field [ ] | Section identity, energy, vocal delivery, instrument cues |
| Lyric Writing | Body of lyrics | Phrasing, hook structure, emotional arc, syllable density |
| Formatting Symbols | Inside/around lyrics | Performance delivery, emphasis, stretching, background layers |
| Suno Settings | Weirdness / Style Influence | How closely Suno follows your prompt |

### What Goes Where

| Category | Where It Lives |
|----------|---------------|
| Genre, BPM, key, mood, texture | Style Prompt box |
| Section labels | Lyrics field — start of section |
| Vocal style, emotion, effects | Lyrics field — in tag at section start |
| Instruments | Lyrics field — stacked in section tag |
| Production/mix | Lyrics field — stacked or as [Mix: ...] |
| Atmosphere / SFX | Lyrics field — as standalone tags |
| Formatting (-, ALL CAPS) | Inside/around the actual lyric text |
| Parentheses | Around lyrics for background/echo layer |

---

## Genre-Specific Gotchas

Community-discovered quirks with specific genre tags in Suno:

- **"punk" in Style causes shorter songs** — Suno interprets punk's fast tempos and short song tradition literally. Use "post-hardcore" or "pop punk" for full-length tracks
- **"progressive metalcore"** tends to trigger power-metal shredding when you want thick rhythm guitars. Use "metalcore" + specific instrument descriptions instead
- **Suno adds its own dynamics** unless you explicitly say "no lulls, full intensity" or "consistent energy" — important for workout/hype tracks where you don't want quiet breakdowns
- **V5 lyrics field max: 5,000 characters** — plan section count accordingly

---

## Instrument Isolation (Sandwich Method)

For solo instrument focus, use both Style prompt AND Exclude Styles together — neither alone is reliable:

1. **Style prompt:** Place the target instrument name at the START and END of the Style prompt
2. **Exclude Styles:** Block other instruments that might bleed in

**Example — solo piano:**
- Style: `Piano, intimate solo piano, C Minor, 72 BPM, warm room reverb, gentle piano`
- Exclude Styles: `guitar, drums, bass, synth, strings`

Both are needed. Style-only leaves room for Suno to add accompaniment. Exclude-only doesn't guarantee the target instrument gets focus.
