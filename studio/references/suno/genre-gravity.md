---
title: Genre Gravity Wells
type: reference
platform: suno
tags: [genre, gravity, co-occurrence, exclude, escape, fusion, weak, strong, repelling, pop, rock, jazz, electronic, metal, indie, rap, trap, classical, bleed, cloud]
---

# Genre Gravity Wells

Suno's model learned from billions of tracks. Genres that frequently appear together are statistically linked — requesting one pulls in the other. Understanding these links explains why certain genres bleed into each other and how to prevent it.

## Co-occurrence Strength

Higher numbers = stronger gravitational pull. These are approximate link strengths from music metadata co-occurrence analysis.

| From | To | Strength | What happens |
|------|-----|----------|-------------|
| Rock | Pop | 315B | Almost any rock prompt picks up pop polish |
| Rap | Trap | 327B | Nearly inseparable — requesting rap gets trap beats |
| Funk | Pop | 116B | Funk prompts drift toward pop grooves |
| Indie | Pop | 89B | "Indie" alone often produces indie-pop |
| Electronic | Dance | 203B | Electronic without subgenre = dance music |
| Emo | Pop | 12.2B | "Emo" gives emotional pop, not emo-punk |
| Emo | Metal | ~0 | Zero direct link — "emo metal" = emotional pop with guitars |
| Jazz | Lo-fi | 45B | Jazz prompts frequently pick up lo-fi textures |
| Classical | Cinematic | 67B | Classical prompts drift toward film score |

## Genre Clouds (inseparable clusters)

These genres travel in packs. Requesting one member often pulls in the others.

| Cloud | Members | Dominant |
|-------|---------|----------|
| **Rap Cloud** | Rap, Trap, Bass, Hip Hop, Beat | Trap (327B links to Rap) |
| **Orchestral Cloud** | Orchestral, Epic, Cinematic, Dramatic, Piano | Cinematic |
| **Indie Cloud** | Indie, Pop, Acoustic, Dreamy, Psychedelic | Pop |
| **Dark Electronic Cloud** | Dark, Synth, Electro, Synthwave, Futuristic | Synth |
| **Metal Cloud** | Metal, Heavy, Aggressive, Distortion, Dark | Metal |
| **Lo-fi Cloud** | Lo-fi, Chill, Jazz, Beats, Ambient | Lo-fi |

**Implication:** To get a specific member without the dominant, you must explicitly exclude the dominant or use escape strategies.

## Tag Strength Tiers

### Strong tags (dominate easily)
These genres overpower everything else. When present, they define the output.
```
Pop, Rock, Electronic, Hip-hop, Classical, Metal, Jazz, R&B
```

### Medium tags (hold their own)
Can coexist with other genres but won't fight strong tags.
```
Folk, Country, Reggae, Funk, Soul, Blues, Ambient, Indie
```

### Weak tags (easily overwhelmed)
These get absorbed by any strong tag in the prompt. Must be protected.
```
Grunge, Math rock, Swing, Chamber music, Bluegrass,
Zydeco, Krautrock, Shoegaze, Post-punk, Bossa nova,
Highlife, Afrobeat (without drums), Throat singing
```

**Rule:** When using a weak tag, exclude the strong tag it's most likely to be absorbed by. Grunge needs `exclude: Pop, Modern Rock`. Math rock needs `exclude: Progressive Rock, Metal`.

## Escape Strategies

When a genre keeps pulling in unwanted sounds despite excludes:

### 1. Explicit Exclusion (first try)
Exclude the dominant member of the cloud.
```
Genre: "emo punk"
Exclude: Pop, Modern Pop, Dance Pop, Synth Pop
```

### 2. Force Weird Combinations (break the cloud)
Pair genres that never co-occur. Suno can't default to a familiar pattern.
```
emo industrial
orchestral phonk
math rock gospel
country drill
folk darkwave
```

### 3. Strategic Contrast — Repelling Pairs
Use opposing descriptors to push away from the gravity well.

| Repelling Pair | Effect |
|---------------|--------|
| Acoustic ↔ Electronic | Naming one repels the other |
| Raw ↔ Polished | "Raw" pushes away pop polish |
| Vintage ↔ Modern | "Vintage" fights modern production |
| Minimal ↔ Dense | "Minimal" prevents layering |
| Dry ↔ Wet | "Dry" prevents reverb wash |
| Intimate ↔ Epic | "Intimate" fights cinematic scale |
| Lo-fi ↔ Hi-fi | Opposing production aesthetics |

**How to use:** Add the repelling descriptor to your Style block's mood slot. If Rock keeps drifting to Pop, add "raw" or "vintage" to push it back.

### 4. Instrument Anchoring (strongest escape)
The instrument IS the genre signal. Suno recognizes instrument ↔ genre associations.

| Instrument | Genre Signal |
|-----------|-------------|
| Shamisen | Japanese/East Asian |
| Tin whistle | Celtic/Irish |
| Sitar | Indian classical |
| Banjo | Bluegrass/Country |
| Kora | West African |
| Theremin | Sci-fi/Experimental |
| Oud | Middle Eastern |
| Erhu | Chinese classical |

**Rule:** Use the instrument instead of the culture label. "shamisen" already signals "Japanese" — adding "Japanese folk" on top doubles the signal and triggers instrument flooding.

## Exclude Strategy by Genre

Based on gravity data — what to exclude for clean genre output:

| Target Genre | Exclude | Why |
|-------------|---------|-----|
| Indie rock | Pop, Dance Pop, Synth Pop | Indie→Pop is 89B links |
| Emo/punk | Pop, Modern Pop, Auto-tune | Emo→Pop is 12.2B |
| Jazz | Lo-fi, Beats, Chill | Jazz→Lo-fi is 45B |
| Classical | Cinematic, Epic, Film score | Classical→Cinematic is 67B |
| Folk | Pop, Indie Pop, Modern | Folk gets absorbed by Indie cloud |
| Funk | Pop, Disco, Dance | Funk→Pop is 116B |
| Ambient | Electronic, Dance, Beats | Ambient gets pulled into EDM |
| Bluegrass | Country Pop, Modern Country | Weak tag, needs protection |
| Grunge | Pop, Modern Rock, Alt-pop | Weak tag, needs protection |

## Fusion Rules

When combining two genres intentionally:

1. **Put the rhythm-template genre first** — it defines the groove
2. **Put the color genre second** — it adds flavor without dominating
3. **Max 2 genres** — more than 2 creates mud
4. **Use a single-word genre when possible** — "Jazz" not "Jazz fusion"
5. **Check the gravity table** — if both genres have strong links to a third, exclude that third

**Example:**
- Want: Jazz + Shamisen → `Jazz, shamisen melodic plucks` (Jazz = rhythm, shamisen = color)
- Risk: Jazz→Lo-fi pull (45B) → exclude `lo-fi, beats, chill` if you want clean jazz
- Want: Folk + Electronic → `Folk, acoustic guitar, subtle electronic pads` (Folk = rhythm, electronic = texture)
- Risk: Folk→Indie→Pop pull → exclude `pop, indie pop, modern`

## High-Value Fusion Pairs (strongest co-existence)

| Genre A | Genre B | Strength | Notes |
|---------|---------|----------|-------|
| Rap | Trap | 5/5 | Strongest co-existing pair |
| Lo-fi | Chill | 5/5 | Dominant lo-fi pairing |
| Metal | Rock | 5/5 | Core metal anchor |
| Orchestral | Epic | 5/5 | Cinematic sweet spot |
| Emo | Emotional | 5/5 | Highest emo combination |
| Soul | R&B | 5/5 | Natural soul pairing |
| Synthwave | Synth | 5/5 | Core synthwave anchor |
| House | Deep | 5/5 | Deep house standard |
| Folk | Acoustic | 4/5 | Folk anchor |
| Jazz | Funk | 4/5 | Natural jazz pairing |

## Anti-Pairs (avoid — produces incoherence)

| Genre A | Genre B | Score | Why |
|---------|---------|-------|-----|
| Cinematic | Dark | 10 | Near zero compatibility |
| Opera | Melodic | 16 | Conflicting vocal expectations |
| Drum and Bass | Funk | 7 | Extremely low co-existence |
