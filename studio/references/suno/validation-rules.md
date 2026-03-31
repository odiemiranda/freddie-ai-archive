---
title: Suno Prompt Validation Rules
type: reference
tags: [suno, validation, conflicts, rules, quality]
source: suno-prompter (Claude Desktop), community findings
verified: 2026-03
---

# Suno Prompt Validation Rules

Suno deprioritizes or averages out conflicting tags rather than erroring. These rules catch combinations that produce muddy or unpredictable output.

## Style Limits

| Limit | Value | Note |
|-------|-------|------|
| Style max chars | 200 | Hard limit, Suno truncates beyond |
| Style soft warning | 180 | Leave headroom for iteration |
| Recommended tags | 4-8 | Fewer = underspecified, more = averaged out |
| Genre minimum | 1 | Genre is Suno's highest-impact descriptor |

## Lyrics Limits

| Limit | Value | Note |
|-------|-------|------|
| Lyrics max chars | 3000 | Hard limit |
| Lyrics soft warning | 2550 (85%) | Leave room for iteration |
| Max content sections | 8 | More = cramped for ~2min clips |
| Max lines per section | 8 | Suno handles 2-6 best |
| Max delivery tags per section | 2 | More gets ignored or averaged |

## Mood Conflicts

Pairs that fight each other and produce averaged-out mush:

| Mood A | Mood B |
|--------|--------|
| happy | sad, melancholic, mournful, dark |
| joyful | sad, melancholic |
| cheerful | somber, dark |
| uplifting | dark, brooding |
| euphoric | melancholic |
| optimistic | mournful |
| peaceful | aggressive, intense, fierce |
| calm | aggressive, angry, high energy, energetic |
| serene | aggressive, intense |
| relaxing | driving, pumping, high energy |
| tranquil | aggressive |
| soothing | aggressive |
| chill | aggressive, intense |
| mellow | high energy, energetic |
| laid-back | driving, pumping |
| low energy | high energy, energetic, driving |
| playful | menacing |
| fun | ominous |
| carefree | brooding |

## Genre Conflicts

Genre combos that confuse Suno rather than creating interesting fusions:

| Genre A | Genre B |
|---------|---------|
| death metal | lo-fi hip hop, bossa nova, smooth jazz |
| black metal | city pop, bedroom pop |
| thrash metal | ambient, downtempo |
| hardcore punk | smooth jazz |
| drill | bossa nova, celtic |
| dubstep | bluegrass, celtic |
| vaporwave | thrash metal |
| grunge | city pop |
| opera | trap |

## Production Conflicts

Tags that cancel each other out:

| Tag A | Tag B |
|-------|-------|
| polished | raw, rough, lo-fi, unpolished, garage recording |
| radio-ready | raw, lo-fi |
| professional mix | unpolished, garage recording |
| crisp | dusty |
| clean | raw, rough |
| modern | vintage, retro |
| futuristic | vintage, retro |
| dry | wet, heavy reverb |
| no reverb | heavy reverb, hall reverb, cathedral |
| intimate | spacious, wide stereo |
| compressed | dynamic |

## Genre Tempo Ranges

Expected BPM ranges per genre. Outside = unusual (may work for creative effect):

| Genre | Min BPM | Max BPM |
|-------|---------|---------|
| ambient | 50 | 90 |
| downtempo | 60 | 100 |
| lo-fi hip hop | 60 | 95 |
| reggae | 60 | 90 |
| blues | 60 | 120 |
| hip hop | 70 | 115 |
| boom bap | 80 | 110 |
| folk | 80 | 130 |
| country | 80 | 140 |
| pop | 90 | 140 |
| bossa nova | 100 | 140 |
| rock | 100 | 160 |
| house | 115 | 135 |
| deep house | 115 | 130 |
| tech house | 120 | 135 |
| techno | 120 | 150 |
| trance | 125 | 150 |
| trap | 130 | 170 |
| drill | 130 | 150 |
| dubstep | 130 | 150 |
| punk rock | 150 | 200 |
| thrash metal | 150 | 220 |
| drum and bass | 160 | 180 |
| jungle | 155 | 175 |
| jazz | 80 | 200 |
| metal | 100 | 200 |
| doom metal | 50 | 80 |

## Vocal / Genre Unusual Combos

Not errors — just flags for unexpected results:

| Vocal Style | Unusual With |
|-------------|-------------|
| Operatic | trap, drill, lo-fi hip hop, dubstep, boom bap |
| Growled | pop, city pop, bossa nova, smooth jazz, bedroom pop |
| Screamed | lo-fi hip hop, ambient, bossa nova, smooth jazz, R&B |
| Mumble Rap | opera, classical, orchestral, celtic, bluegrass |
| Falsetto | death metal, black metal, thrash metal, drill |
| Whispered | thrash metal, death metal, punk rock, drill |

## Section Flow Rules

Typical section transitions. Unusual transitions may work for creative effect:

| After | Typically Followed By |
|-------|----------------------|
| Intro | Verse, Verse 1, Instrumental, Hook |
| Verse | Pre-Chorus, Chorus, Hook, Verse, Bridge, Refrain |
| Verse 1 | Pre-Chorus, Chorus, Hook, Bridge, Refrain |
| Verse 2 | Pre-Chorus, Chorus, Hook, Bridge, Refrain, Outro |
| Verse 3 | Pre-Chorus, Chorus, Bridge, Outro |
| Pre-Chorus | Chorus, Drop, Build |
| Chorus | Verse, Verse 2, Verse 3, Post-Chorus, Bridge, Instrumental, Interlude, Break, Outro, End, Fade Out |
| Post-Chorus | Verse, Verse 2, Verse 3, Bridge, Instrumental, Interlude |
| Bridge | Chorus, Outro, Solo, Guitar Solo, Drop, Build |
| Hook | Verse, Verse 2, Chorus, Bridge |
| Refrain | Verse, Verse 2, Chorus, Bridge |
| Instrumental | Verse, Verse 2, Chorus, Bridge, Outro |
| Interlude | Verse, Verse 2, Chorus, Bridge |
| Break | Chorus, Verse, Drop, Build |
| Build | Chorus, Drop |
| Drop | Verse, Verse 2, Breakdown, Outro, Build |
| Solo | Chorus, Bridge, Outro |
| Guitar Solo | Chorus, Bridge, Outro |
| Breakdown | Build, Chorus, Drop, Outro |
| Outro | End, Fade Out |

## Cross-Prompt Validation

Checks between Style and Lyrics:

| Rule | Severity | Description |
|------|----------|-------------|
| Instrumental + lyrics | warning | Style excludes vocals but lyrics have text |
| Vocal redundancy | info | Style defines vocal + lyrics has same delivery tag (redundant, not harmful) |
| EDM without drops | info | EDM genre but no [Build] or [Drop] in lyrics |
| Rap without delivery | info | Hip hop/rap genre but no rap delivery tags |
| Slow tempo + drops | info | <80 BPM with Build/Drop — use Crescendo/Swell instead |
