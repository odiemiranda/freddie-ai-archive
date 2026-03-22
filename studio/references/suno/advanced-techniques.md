# Advanced Prompt Techniques

<!-- TOC
| Section | Line | Keywords |
|---------|------|----------|
| Technique 1: Emotion Layering | 23 | emotion layering, complementary feelings, depth |
| Technique 2: Era Specificity | 56 | era, specificity, years, decades, period sounds |
| Technique 3: Dynamic Contrast | 82 | dynamic contrast, loud, soft, dynamics, pixies |
| Technique 4: Storytelling Arcs | 112 | storytelling, arcs, rising, falling, wave, plateau |
| Technique 5: Sensory Language | 136 | sensory, visual, temperature, texture, atmosphere |
| Technique 6: Negative Space Prompting | 172 | negative space, avoidance, what to avoid |
| Technique 7: The Iteration Method | 197 | iteration, variation, 4-8 rule, systematic changes |
| Before & After: Technique Applied | 228 | before, after, examples, improvement |
| Prompt Quality Scorecard | 276 | scorecard, quality, rating, criteria |
| Quick Reference: Technique Combinations | 298 | quick reference, combinations, lo-fi, sleep, workout |
| Fusion Compatibility Reference | 320 | fusion, anti-pairs, compatibility, genre pairs, avoid combining |
TOC -->

Master-level techniques for crafting professional Suno AI prompts. These build on the foundations in other reference files.

---

## Technique 1: Emotion Layering

Instead of single emotions, combine complementary feelings for depth and complexity.

| Basic | Advanced |
|-------|----------|
| "Sad song" | "Melancholic yet hopeful, nostalgic with hints of acceptance" |
| "Happy music" | "Joyful but grounded, celebratory with underlying warmth" |
| "Dark track" | "Mysterious and brooding, haunting yet beautiful" |

### Effective Combinations

```
Melancholic yet hopeful
Nostalgic with quiet joy
Peaceful but slightly sad
Energetic yet controlled
Dark but beautiful
Intense yet refined
Playful with depth
Vulnerable but strong
Dreamy and contemplative
Bittersweet and warm
```

### Why This Works

"Melancholic yet hopeful" creates the emotional texture that dominates viral lo-fi playlists — sad enough to feel deep, optimistic enough to sustain long study sessions. Single emotions feel flat; layered emotions feel human.

For full mood keyword lists and key/BPM recommendations per mood, see `moods.md`.

---

## Technique 2: Era Specificity

Reference **specific years** (not just decades) for authentic period sounds.

```
"1967 psychedelic rock, Summer of Love atmosphere"
"1973 funk, early disco influence, analog warmth"
"1983 synth-pop, MTV era production, gated reverb"
"1995 East Coast hip-hop, golden age boom bap"
"1997 trip-hop, Bristol sound, Portishead influence"
"2003 early 2000s R&B, Timbaland production style"
"2010 blog era indie, chillwave aesthetic"
```

### Example

```
[Instrumental] 1983 synth-pop, MTV era production, gated snare, DX7 electric piano, analog arpeggios, E Major, 118 BPM, neon-lit aesthetic, John Hughes soundtrack feel [No Vocals]
```

**Why specific years beat decades:** "80s synth-pop" gives Suno a wide range. "1983 synth-pop, MTV era" narrows it to early-MTV DX7 brightness, not late-80s hair metal excess.

For era-by-era production characteristics, see `production.md` (Era References).

---

## Technique 3: Dynamic Contrast

Specify dynamics for more interesting, professional compositions.

### Contrast Patterns

```
"Quiet verses building to explosive choruses, dramatic dynamic range"
"Soft-loud-soft structure, Pixies-style dynamics"
"Minimal opening expanding to full arrangement"
"Dense texture thinning to solo instrument"
"Building tension releasing into euphoria"
"Intimate verse, anthemic chorus"
```

### Structure Example

```
[Intro] Sparse, single instrument
[Verse] Quiet, minimal elements
[Pre-Chorus] Building tension, adding layers
[Chorus] Full arrangement, maximum energy
[Verse 2] Pull back, but retain momentum
[Bridge] Contrast - different texture entirely
[Final Chorus] Biggest version, all elements
[Outro] Gradual strip-down or abrupt end
```

---

## Technique 4: Storytelling Arcs

Guide the emotional journey of the track.

| Arc Type | Description | Best For |
|----------|-------------|----------|
| **Rising** | Uncertainty → Determination → Triumph | Motivational, workout |
| **Falling** | Peace → Tension → Resolution | Meditation, sleep |
| **Wave** | Build → Release → Build → Release | EDM, dance |
| **Plateau** | Consistent energy, minimal variation | Background, study |
| **Descent** | Bright → Melancholic → Peaceful | Emotional, reflective |

### Example Prompts

```
"Opening with uncertainty, building to determined climax, ending with peaceful resolution"

"Starting minimal and mysterious, growing into full triumphant arrangement, fading to gentle conclusion"

"Beginning bright and optimistic, gradually becoming introspective, resolving in bittersweet acceptance"
```

---

## Technique 5: Sensory Language

Use non-musical sensory words to evoke atmosphere. Suno responds well to visual and tactile language.

### Visual → Audio Translation

| Visual | Audio Translation |
|--------|-------------------|
| "Neon-lit" | Bright synths, electronic shimmer |
| "Dusty" | Vintage, tape saturation, lo-fi |
| "Golden hour" | Warm, amber tones, soft glow |
| "Midnight blue" | Deep, nocturnal, contemplative |
| "Rain-soaked" | Reverb-heavy, melancholic, distant |

### Temperature Words

```
Warm - Analog saturation, bass presence, vintage
Cold - Digital precision, sparse, clinical
Frozen - Static, minimal movement, icy synths
Hot - Aggressive, distorted, energetic
Cool - Laid-back, jazzy, relaxed
```

### Texture Words

```
Velvet - Smooth, lush, enveloping
Rough - Gritty, distorted, raw
Silk - Flowing, delicate, refined
Gravel - Raspy, textured, organic
Glass - Crystalline, fragile, bright
```

---

## Technique 6: Negative Space Prompting

Specify what to AVOID as much as what to include.

### Avoidance Phrases

```
"No sudden changes, continuous flow"
"Avoid aggressive elements, keep gentle"
"Without electronic elements, purely organic"
"No drums, focus on harmonic instruments"
"Minimal variation, avoid complexity"
```

### For Sleep/Meditation Music

```
"No sudden dynamic changes that would startle"
"Avoid high frequencies that cause alertness"
"No rhythmic complexity that engages the mind"
"Without melodic hooks that demand attention"
```

---

## Technique 7: The Iteration Method

Small systematic changes unlock the best results.

### The 4-8 Rule

Generate 4-8 variations of each prompt, changing only 1-2 elements:

1. **Base prompt** - Your starting point
2. **BPM variation** - ±5-10 BPM
3. **Key variation** - Try relative minor/major
4. **Instrument swap** - Rhodes → Wurlitzer
5. **Mood adjustment** - "peaceful" → "serene"
6. **Texture change** - Add/remove vinyl crackle
7. **Structure change** - Different intro/outro
8. **Era shift** - 80s version → 90s version

### Iteration Example

```
Base: "Lo-fi hip hop, Rhodes piano, 82 BPM, C Minor, vinyl crackle"

V2: "Lo-fi hip hop, Wurlitzer keys, 82 BPM, C Minor, vinyl crackle"
V3: "Lo-fi hip hop, Rhodes piano, 78 BPM, C Minor, vinyl crackle"
V4: "Lo-fi hip hop, Rhodes piano, 82 BPM, A Minor, vinyl crackle"
V5: "Lo-fi hip hop, Rhodes piano, 82 BPM, C Minor, tape hiss"
V6: "Lo-fi hip hop, Rhodes piano, 82 BPM, C Minor, vinyl crackle, rain ambience"
```

---

## Before & After: Technique Applied

These show how a mediocre prompt improves when you apply specific techniques.

### Example 1: Emotion Layering + Sensory Language

**Before:**
```
[Instrumental] Lo-fi hip hop, piano, drums, sad mood, 80 BPM [No Vocals]
```

**After:**
```
[Instrumental] Lo-fi hip hop, warm Rhodes piano, dusty boom-bap drums, melancholic yet hopeful, golden hour warmth, C Minor, 82 BPM, vinyl crackle, nostalgic with quiet joy [No Vocals]
```

**What changed:** Single emotion ("sad") → layered ("melancholic yet hopeful"). Generic "piano" → specific "warm Rhodes piano". Added sensory language ("dusty", "golden hour warmth"). Added key and texture.

### Example 2: Era Specificity + Dynamic Contrast

**Before:**
```
[Instrumental] Synth pop, retro, electronic drums, upbeat, catchy [No Vocals]
```

**After:**
```
[Intro] Single synth arpeggio [Build] [Instrumental] 1983 synth-pop, MTV era DX7 electric piano, gated snare, analog arpeggios, E Major, 118 BPM, minimal opening expanding to full arrangement, neon-lit aesthetic [No Vocals]
```

**What changed:** Vague "retro" → specific "1983 synth-pop, MTV era". Added dynamic structure (intro → build). Added sensory ("neon-lit"). Named specific instruments.

### Example 3: Negative Space + Storytelling Arc

**Before:**
```
[Instrumental] Sleep music, ambient, slow, relaxing [No Vocals]
```

**After:**
```
[Intro] [Instrumental] Deep ambient sleep, warm drone layers, F Major, 50 BPM, no sudden dynamic changes, no rhythmic complexity, consistent energy minimal variation, womb-like therapeutic atmosphere [Sustained] [No Vocals]
```

**What changed:** Added negative space ("no sudden dynamic changes, no rhythmic complexity"). Applied plateau arc ("consistent energy minimal variation"). Added specific BPM, key, and therapeutic context.

---

## Prompt Quality Scorecard

Rate your prompt before generating. Aim for 7+ points.

| Criteria | 0 Points | 1 Point | 2 Points |
|----------|----------|---------|----------|
| **Genre clarity** | Vague ("music") | Generic ("lo-fi") | Specific ("lo-fi hip hop, boom-bap") |
| **Instruments** | None named | Generic ("piano, drums") | Specific with adjectives ("warm Rhodes, dusty boom-bap drums") |
| **Emotion** | None or single word | Single emotion ("sad") | Layered ("melancholic yet hopeful") |
| **Technical specs** | No BPM or key | BPM only | BPM + Key + tempo feel |
| **Atmosphere** | None | Generic ("relaxing") | Sensory/scene ("golden hour warmth, rainy window") |
| **Bracketing** | Missing tags | Partial (`[Instrumental]` only) | Full (`[Instrumental]` + structure + `[No Vocals]`) |

| Score | Quality | Action |
|-------|---------|--------|
| 0-4 | Weak | Rewrite — too vague for consistent results |
| 5-7 | Solid | Good for generation, may need 4-6 tries |
| 8-10 | Strong | Should produce usable results in 2-4 tries |
| 11-12 | Expert | Maximum control, consistent quality |

---

## Quick Reference: Technique Combinations

### For Lo-Fi Study Music
- Emotion layering: "melancholic yet hopeful"
- Era: "90s golden age sample aesthetic"
- Negative space: "no sudden changes"
- Sensory: "dusty, warm, golden hour"

### For Sleep Music
- Negative space: "no rhythmic complexity, no melodic hooks"
- Storytelling arc: "plateau - consistent, minimal variation"
- Sensory: "warm, enveloping, dark"
- Emotion layering: "deeply peaceful, surrendering"

### For Energetic/Workout
- Dynamic contrast: "building to explosive release"
- Era: specific decade for nostalgia
- Storytelling: "rising arc - uncertainty to triumph"
- Sensory: "hot, driving, relentless"

---

## Fusion Compatibility Reference

### High-Value Fusion Pairs

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

### Anti-Pairs (Avoid Combining)

These combinations tend to produce incoherent results in Suno:

| Genre A | Genre B | Why It Fails |
|---------|---------|-------------|
| Cinematic | Dark (as genre tag) | "Dark" as a genre tag overrides cinematic dynamics, flattening the arrangement |
| Opera | Melodic | Suno can't resolve operatic technique with generic "melodic" — too vague |
| Drum and Bass | Funk | BPM incompatibility — DnB's 170+ BPM destroys funk groove (100-120 BPM) entirely |

**Rule:** If you want a "dark cinematic" track, use `[cinematic | haunting | ominous]` instead of the Dark genre tag. If you want "melodic opera", specify the actual melody character: `[opera | lyrical vocal line | soaring]`.

### Cross-References

- For instrument personality and adjective libraries → `instruments.md`
- For artist DNA extraction and usage guide → `artists-a-f.md`
- For production specifics (compression, reverb, texture) → `production.md`
- For mood keywords and key/BPM pairings → `moods.md`
