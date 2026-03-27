---
title: Suno Craft Rules
tags: [suno, craft, rules, genre-normalization, vibe-map, style-formula, hard-limits, frequency, fusion, fusion-strategy, multi-genre, realism, lofi, edm, vocal, interaction, validation, slot-formula]
---

# Suno Craft Rules

Hard limits, normalization tables, and production rules that gate every Suno generation. Referenced by workflow.md nodes during the distill and build phases.

## Hard Limits

| Element | Max | Why |
|---------|-----|-----|
| Genres | 2 | More than 2 pull Suno in conflicting directions |
| Named instruments | 3 | Every instrument past 3 muddies the output |
| Rhythm sources | 1 | Multiple drum/percussion sources fight each other |
| Mood descriptors | 5 | Too many mood words dilute the emotional signal |
| Lyrics direction lines per section | 3 | Suno has a short attention span for guidance |
| Style block chars | ~75-120 (typed brackets), ~120-170 (fusion). Hard max 200/250 | Ultra-lean outperforms verbose |
| Structural tags per song | 5-7 | More reduces reliability across all tags |
| Mood tags per song | 1-2 | Supplementary only |
| Texture terms | 2 max | No overlapping textures |

## Genre Normalization Table

Normalize before counting. Each lo-fi subgenre triggers its own texture package — stacking floods output.

| User Input | Canonical | Family |
|-----------|-----------|--------|
| lo-fi, lofi, lo-fi hip-hop, lofi hip hop, study beats | lo-fi | lo-fi |
| jazzhop, jazz hop, lo-fi jazz | lo-fi (jazz-colored) | lo-fi |
| chillhop, chill hop | lo-fi (chillhop) | lo-fi |
| boom bap, boom-bap | hip-hop | hip-hop |
| trap, drill, phonk | hip-hop variant | hip-hop |
| smooth jazz, cool jazz, bebop, bossa nova | jazz variant | jazz |
| ska, dub, dancehall, rocksteady | reggae variant | reggae |
| alt rock, indie rock, post-punk, grunge | rock variant | rock |
| house, techno, trance, dubstep | electronic variant | electronic |
| synth pop, dream pop, indie pop, electro pop | pop variant | pop |

**HARD RULE: Never use culture/region as genre label.** "Japanese folk" → "Jazz shamisen fusion". The instrument IS the cultural element.

| Don't say | Say instead | Why |
|-----------|-------------|-----|
| Japanese folk fusion | Jazz shamisen fusion | Shamisen IS the Japanese element |
| Celtic folk | Folk bodhrán fusion | Bodhrán IS the Celtic element |
| African percussion | Jazz djembe fusion | Djembe IS the African element |
| Middle Eastern | Ambient oud fusion | Oud IS the Middle Eastern element |
| Indian classical | Lo-fi sitar fusion | Sitar IS the Indian element |

## Multi-Genre Fusion Strategy (3+ sources)

When blending more than 2 traditions, map each to a role — don't list them all as genres:

| Role | What it controls | Slot | Example |
|------|-----------------|------|---------|
| **Primary genre** | Rhythm template, structure, overall feel | Slot 2 | Lo-fi Jazzhop |
| **Rhythmic feel** | Drum pattern, groove character | Slot 5 | reggae one-drop rhythm |
| **Lead instrument** | Cultural color, melodic identity | Slot 3 | shamisen melodic plucks |
| **Support instrument** | Background texture, harmonic bed | Slot 4 | harp low-register drone, no melody |

## Frequency Darkening (bright instruments)

When a fusion includes naturally bright instruments (shamisen, tin whistle, banjo):

- **Global (Style block):** `lowpass filtering` — darkens entire mix
- **Per-instrument (Style block):** `low-pass jazz piano loop` — muffles only that instrument
- **Per-section (Lyrics brackets):** `[lowpass breakdown]`, `[low-pass sweeps into chorus]`

Choose lower-pitched variants when available: "low whistle" not "tin whistle", "bass shamisen" not "shamisen".

## Vibe Map — Dangerous Words

User vibes sound evocative but Suno reads them as literal production instructions:

| User Vibe | Problem | Style Translation | Lyrics Translation |
|-----------|---------|-------------------|---------------------|
| lazy | Aimless meandering | laid-back | unhurried, gentle |
| cold | Over-dampens all instruments | cool, crisp | still, quiet |
| muted | Compresses frequency range | soft, restrained | subdued, hushed |
| distant | Pushes instruments too far back | spacious | far-off, fading |
| dark | Suno reads as minor key + heavy | moody, shadowed | dimly-lit, late-night |
| floating | No positional info for instruments | sustained, legato | drifting, weightless |
| dreamy | Adds too much reverb/wash | ethereal, soft-focus | hazy, half-remembered |
| raw | Can trigger distortion/grit | organic, unpolished | honest, stripped |

Words not on this table pass through unchanged ("warm", "bright", "driving" are already production language).

**Vibe translation process:**
1. Classify each vibe word against 5 categories: Texture, Space, Era, Energy, Atmosphere
2. Check dangerous vibes table — translate if matched
3. Contrast check — ensure 2+ categories covered, drop same-bucket duplicates
4. Produce Style voice (production descriptors) + Lyrics voice (sensory/dynamic words)

## 11-Slot Style Block Formula

```
SLOT  CONTENT                         PRIORITY
───── ─────────────────────────────── ─────────
1     [Instrumental]                  if instrumental
2     Genre (max 2)                   CRITICAL — ~15 chars
3     Lead instrument + behavior      CRITICAL — ~20-30 chars
4     Support instrument + CONSTRAINT HIGH — ~20-30 chars
5     Rhythm + constraint             HIGH — ~15-25 chars
6     Chord progression               LOW — cut first
7     Key, BPM                        CRITICAL — ~15 chars
8     Textures (max 2)                LOW — cut second
9     Mood (max 2)                    MEDIUM
10    Production keywords             CRITICAL — "clean mix, separated instruments"
11    [No Vocals] [Background Music]  if instrumental
```

**Cut order when over budget:** Slot 6 → Slot 8 → Slot 9 (keep 1) → Slot 4 (simplify). Never cut 2, 3, 7, or 10.

**Front-load importance:** First tag = ~60% influence, second = ~25%, third = ~10%. Genre and lead instrument MUST be first two elements.

**Element count:** 3-5 tags optimal, max 7. Past 7 elements = over-stuffed.

**Audio quality vs length:**
| Style + Lyrics Length | Audio Clarity |
|---|---|
| 500-1,500 chars | 9.5/10 — cleanest |
| 1,500-2,500 chars | 9.0/10 |
| 2,500-3,500 chars | 8.5/10 |
| 3,500-5,000 chars | 8.0/10 |

## Style Block Validation (8-step gate)

1. **Char count** — ~75-120 (typed brackets), hard max 200. Validate via `charcount` tool
2. **Slot order** — walk slots 1-11, fix missing/out-of-order
3. **Slot bleed** — texture/groove terms in instrument slots? Move to 8-9
4. **Front-load** — genre + lead instrument must be first two elements
5. **Instrument count** — max 3 named instruments
6. **Keywords** — "clean mix" + "separated instruments" present? (mandatory)
7. **Conflict check** — contradictory tags? (calm vs aggressive, lo-fi vs polished)
8. **Element count** — 3-5 optimal, max 7

## Conflicting Tags (never stack)

- Energy: calm vs aggressive, driving vs laid-back, explosive vs restrained
- Texture: lo-fi vs polished, crisp vs fuzzy, clean vs distorted
- Space: intimate vs vast, dry vs reverb-heavy

If two tags fight: cut one OR reframe as complementary pair ("uplifting yet peaceful" works).

## Support Instrument Constraints

Support instruments MUST have explicit constraints:
- `harp low-register pad chords, no melody` — register + role
- NOT `harp warm harmonic bed` — too vague, Suno gives melodic freedom
- Don't mix texture ("warmth") or groove ("laid-back swing") into instrument slots

## Instrument Interaction Rules

- Describe instrument *relationships*, not isolated roles
- **Style block:** directive verbs ("leads", "plucks", "drives", "answers")
- **Lyrics block:** dynamic interaction verbs ("trades phrases with", "responds to")
- At least one line per section should describe how instruments relate
- Anti-pattern: "A does X / B does Y / C does Z" → "A and B trading phrases over C's groove"
- Lead gets more bracket lines than support — equal presence = Suno treats both as co-leads

## Lo-fi / Background Music Rules

- Always jazz-extended chords (7ths, 9ths, 13ths) — never plain triads
- Always swing/off-grid/humanized drums — never quantized
- Max 2 texture elements (vinyl crackle + tape warmth, NOT vinyl + tape + rain + hum)
- Warm filtered bass — not clean or bright
- Roll off highs: `soft high-end` or `rolled-off highs`
- Background/study: no bright melodies, loop-friendly, relaxed, even energy
- Loop-safe endings: `[Sustained]` (not `[Fade Out]` or `[Outro]`)
- Include `loop-friendly` in production keywords

## Electronic/EDM Rules

- Be specific about subgenre ("melodic techno" not "techno")
- Specify bass type and synth texture
- Triple-layer instrumental: `[Instrumental]` start + `[No Vocals]` end + Exclude: vocals
- Use pipe stacking for drops/transitions:
  - Drops: `[explosive drop | sidechained synth bass | sub drop impact]`
  - Builds: `[Build-Up | orchestral crescendo | white noise riser]`
  - Breaks: `[Break | cinematic interlude | pads only]`

## Vocal Track Rules

- Vocals occupy mid range (1-4 kHz) — instruments must not compete there
- Describe how instruments relate TO the vocals in Style block
- Vocal range header at top of Lyrics block (mandatory for vocal tracks):
  ```
  [Voice description + texture]
  [Vocal range: {note}-{note}]
  [Exclude: {unwanted registers}]
  ```
- Phonetic accent control: triple reinforcement (Style + metadata + phonetic lyrics)
- Reference `vocals.md` for range presets and accent recipes

## Realism Descriptors (acoustic genres)

For acoustic-forward genres (folk/jazz/blues/classical/singer-songwriter):

- **Tier 1 (subtle):** `close mic, natural dynamics, room tone`
- **Tier 2 (recorded feel):** + `breath detail, pick noise, short room reverb, limited stereo`
- **Tier 3 (full bedroom/live):** + `fret squeak, tape saturation, wow and flutter, one-take, imperfections kept`

Go in slots 8 (textures) and 10 (production). **Counterproductive for electronic/hip-hop/synthwave.**

## Six-Factor Pre-Build Check

| # | Factor | Check | If Failing |
|---|--------|-------|------------|
| 1 | Structure tag separation | ALL structure tags in Lyrics only? | Move them |
| 2 | Exclude breadth | Minimal with `[Instrument:]` tags? | Only dealbreakers |
| 3 | Mood bucket diversity | 2+ production categories? | Swap same-bucket |
| 4 | Texture budget | Max 2 non-overlapping? | Drop excess |
| 5 | Verb type | Directive verbs in Style? | Replace passive |
| 6 | Genre clarity | Clear rhythm template? | Sharpen label |
