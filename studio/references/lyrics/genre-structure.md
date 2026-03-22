# Genre-to-Lyrics Structure Guide

How genre shapes lyrics structure — lines per verse, syllable density, rhyme approach, and song form. Use this to match lyrics composition to how the genre is actually performed.

<!-- TOC
| Section | Line | Keywords |
|---------|------|----------|
| Genre Quick Reference | 28 | genre, quick reference, structure, targets, lines per verse, syllable, density, rhyme |
| Song Architecture by Genre | 48 | architecture, sections, song form, verse count, pre-chorus, bridge, breakdown, hook vs chorus |
| Section Roles | 52 | section, role, pre-chorus, bridge, breakdown, hook, post-chorus, interlude |
| Architecture Patterns | 63 | architecture, pattern, verses, chorus, sections, total, song form |
| When to Add vs Cut Sections | 81 | add section, cut section, pre-chorus, third verse, breakdown, interlude |
| Ambient | 95 | ambient, sparse, texture, drone, minimal |
| Folk | 102 | folk, storytelling, acoustic, narrative, campfire |
| Reggae | 109 | reggae, offbeat, laid-back, patois, dub |
| Jazz | 116 | jazz, jazz vocals, conversational, loose rhyme, scat, swing, groovy |
| Country | 124 | country, story songs, twang, Nashville |
| Pop | 131 | pop, hook-driven, melodic, singable, chorus |
| Indie | 138 | indie, conversational, quirky, off-kilter |
| Lo-fi | 145 | lo-fi, lofi, chill, lo-fi vocals, dreamy, murmured |
| Rock | 152 | rock, power chorus, anthem, driving |
| R&B / Soul | 159 | r&b, rnb, soul, melisma, smooth vocals, runs |
| Melodic Hip-Hop | 166 | melodic hip-hop, melodic rap, sung hooks, trap soul |
| Rap / Hip-Hop | 173 | rap, hip-hop, hip hop, bars, dense lyrics, internal rhyme, wordplay |
| Metal | 181 | metal, aggressive, screamed, breakdown, harsh vocals |
| Fusion Rules | 190 | fusion, genre fusion, density spectrum, mixed genres |
| Suno-Specific Limits | 217 | suno, suno limits, suno-specific, syllable limits, max syllables |
TOC -->

---

## Genre Quick Reference

| Genre | Lines/Verse | Lines/Chorus | Syl/Line | Density | Rhyme Approach | Typical Structure |
|-------|-------------|--------------|----------|---------|----------------|-------------------|
| **Ambient** | 2-3 | 2 | 3-5 | Sparse | None/assonance | Intro→V→V→Outro |
| **Folk** | 4-6 | 4 | 6-8 | Moderate | ABAB/ABCB | V→V→C→V→C→V |
| **Reggae** | 4 | 4 | 5-7 | Moderate | Slant/AABB | V→C→V→C→Bridge→C |
| **Jazz** | 4 | 4 | 5-7 | Low-Mod | Loose/slant/none | V→C→V→C→Bridge→C |
| **Country** | 4-6 | 4 | 6-8 | Moderate | AABB/ABAB | V→C→V→C→Bridge→C |
| **Pop** | 4-6 | 4-6 | 7-9 | Moderate | ABAB/AABB | V→PC→C→V→PC→C→Bridge→C |
| **Indie** | 4-6 | 4 | 5-8 | Low-Mod | Slant/irregular | V→C→V→C→Bridge→C |
| **Lo-fi** | 4 | 2-4 | 4-7 | Low | Slant/assonance | V→Hook→V→Hook→Outro |
| **Rock** | 4-6 | 4-6 | 7-10 | Mod-High | ABAB/AABB | V→PC→C→V→PC→C→Bridge→C |
| **R&B/Soul** | 4-6 | 4-6 | 7-10 | Moderate | Slant/internal | V→PC→C→V→PC→C→Bridge→C |
| **Melodic Hip-Hop** | 6-8 | 4-6 | 8-10 | High | End+internal | V→Hook→V→Hook→Bridge→Hook |
| **Rap/Hip-Hop** | 8-16 | 4 (hook) | 8-12 | Very High | End+internal+multi | V→Hook→V→Hook→V→Hook |
| **Metal** | 4-8 | 4 | 6-10 | High | ABAB/none | V→C→V→C→Breakdown→C |

---

## Song Architecture by Genre

How genres break a song into sections. Not every genre uses the same building blocks — a pre-chorus is essential in pop but wrong in jazz. The number of verses, chorus placement, and whether a bridge exists are genre decisions, not defaults.

### Section Roles

| Section | Role | Genres That Need It | Genres That Skip It |
|---------|------|--------------------|--------------------|
| **Pre-Chorus** | Tension builder before chorus payoff | Pop, rock, R&B | Jazz, folk, hip-hop, ambient, lo-fi |
| **Bridge** | Perspective shift, the volta | Pop, rock, country, R&B, hip-hop | Ambient, lo-fi (optional everywhere else) |
| **Breakdown** | Strip energy before rebuild | Metal, EDM, rock | Jazz, folk, country, ambient |
| **Hook** (vs Chorus) | Short repeated phrase, not a full section | Hip-hop, lo-fi, melodic hip-hop | Pop, rock (use chorus instead) |
| **Post-Chorus** | Ride the energy after chorus | Pop, EDM | Most other genres |
| **Interlude/Break** | Breathing room, transition | Jazz, ambient, lo-fi | Hip-hop, metal |

### Architecture Patterns

| Genre | Verses | Chorus/Hook | Pre-Chorus | Bridge | Total Sections | Notes |
|-------|--------|-------------|------------|--------|----------------|-------|
| **Ambient** | 2 | 0-1 | No | No | 3-4 | Sections blur into each other. Intro and outro do heavy lifting |
| **Folk** | 3-4 | 1 (repeated) | No | Optional | 5-8 | Story-driven — more verses than most genres. Chorus is an anchor, not the point |
| **Reggae** | 2-3 | 1 (repeated) | No | Optional | 5-7 | Chorus carries the message. Verses set it up. Keep it simple |
| **Jazz** | 2-3 | 1 (repeated) | No | Yes | 5-7 | No pre-chorus — goes straight from verse to chorus. Bridge is where the turn lives |
| **Country** | 2-3 | 1 (repeated) | No | Yes | 6-8 | Bridge often delivers the twist/reveal. Third verse optional but powerful if it lands |
| **Pop** | 2 | 1 (repeated 3x) | Yes | Yes | 8-10 | Most structured. PC→C is the core engine. Bridge before final chorus |
| **Indie** | 2-3 | 1 (repeated) | Optional | Optional | 5-8 | Can break rules intentionally. Asymmetric structures work |
| **Lo-fi** | 2 | Hook (repeated) | No | No | 4-5 | Minimal sections. Hook is a short phrase, not a full chorus. Loop-friendly |
| **Rock** | 2-3 | 1 (repeated 3x) | Yes | Yes | 8-10 | Similar to pop but heavier dynamics. Breakdown replaces bridge sometimes |
| **R&B/Soul** | 2 | 1 (repeated 2-3x) | Yes | Yes | 7-9 | Pre-chorus is a slow burn. Bridge gets vulnerable. Ad-libs on final chorus |
| **Melodic Hip-Hop** | 2-3 | Hook (repeated) | No | Optional | 6-8 | Hook is melodic, verses are rhythmic. Bridge adds emotional depth |
| **Rap/Hip-Hop** | 3 | Hook (repeated) | No | Optional | 7 | Three verses is standard. Hook between each. Bridge rare but powerful when used |
| **Metal** | 2-3 | 1 (repeated) | Optional | Breakdown | 7-9 | Breakdown replaces bridge — drops energy before final chorus rebuild |

### When to Add vs Cut Sections

- **Add a pre-chorus** when the verse-to-chorus energy gap is big and needs a ramp (pop, rock, R&B)
- **Skip the pre-chorus** when the genre flows conversationally — a ramp-up would feel forced (jazz, folk, hip-hop)
- **Add a third verse** when the story needs a final chapter (folk, country, rap) or when the third verse recontextualizes the chorus
- **Skip the third verse** when two verses + bridge already deliver the turn (pop, R&B)
- **Use a hook instead of chorus** when the genre wants a short repeated anchor, not a full singalong section (hip-hop, lo-fi)
- **Add a breakdown** when the genre needs a dramatic energy drop before the final push (metal, EDM, rock)
- **Add an interlude** when the song needs breathing room between high-energy sections (jazz, ambient)

---

## Per-Genre Structure

### Ambient
- **Lines per verse**: 2-3 (or none — just phonetic textures)
- **What makes it work**: Space matters more than words. Long pauses between phrases. Open vowels that can sustain (ooh, aah, oh). Words are texture, not narrative
- **Rhyme**: Assonance only — forced rhyme sounds wrong in ambient
- **Singability**: Whispered, hummed, or sustained vowel sounds. No rapid syllables
- **Suno note**: Pair with `[Soft vocals]` or `[Humming]` tags. Less text = more atmospheric space

### Folk
- **Lines per verse**: 4-6 (storytelling verses can run longer)
- **What makes it work**: Narrative clarity — each verse advances the story. Conversational phrasing, like someone telling you something over a campfire. Simple vocabulary
- **Rhyme**: ABAB or ABCB — structured but not forced. Slant rhymes feel natural
- **Singability**: Must flow like speech. If you can't say it naturally, don't sing it
- **Suno note**: Folk tolerates longer verses than most genres. 6-line verses work if the story needs them

### Reggae
- **Lines per verse**: 4 (tight, rhythmic)
- **What makes it work**: Offbeat phrasing — lyrics land between beats, not on them. Laid-back delivery with space for the rhythm section. Message-driven but not preachy
- **Rhyme**: AABB couplets or slant rhyme. Patois-influenced phrasing acceptable
- **Singability**: Relaxed mouth-feel. Avoid consonant clusters. Open syllables (words ending in vowels)
- **Suno note**: Keep verses short — the groove does the work, not the words

### Jazz
- **Lines per verse**: 4 (standard) — NEVER 8+
- **What makes it work**: Conversational, unhurried phrasing. Lyrics feel like someone musing aloud, not performing. Space between phrases for melodic interpretation. The singer needs room to phrase around the beat, stretch syllables, and breathe
- **Rhyme**: Loose — slant rhymes, assonance, or no rhyme at all. Tight end-rhymes on every line sound like pop or rap, not jazz
- **Singability**: Short phrases with melodic space. Open vowels (oh, you, stay, mmm) that can sustain over chord changes. Avoid packed syllables — a jazz singer needs to bend notes, not race through words
- **What to avoid**: Dense internal rhyme (that's rap). 8+ line verses (that's hip-hop). Consonant clusters (strengths, scratched) that choke the melody. Rapid-fire syllable delivery
- **Suno note**: Max 10 syl/line for jazz. Suno interprets dense text as rap even with jazz Style tags. Keep it sparse

### Country
- **Lines per verse**: 4-6
- **What makes it work**: Story songs with concrete details — truck brands, county roads, specific weather. Emotional directness without irony. The bridge often delivers the turn/twist
- **Rhyme**: AABB or ABAB — country listeners expect clear rhyme structure
- **Singability**: Plain language, common words. Twang-friendly vowels (long a, long i)
- **Suno note**: Country and folk have similar density. Differentiate through vocabulary and imagery, not structure

### Pop
- **Lines per verse**: 4-6
- **What makes it work**: Hook-forward — the chorus is the point, verses exist to set it up. Melodic phrases that are immediately singable. Pre-chorus builds tension for the chorus payoff
- **Rhyme**: ABAB or AABB — clean, predictable rhyme supports memorability
- **Singability**: 7-9 syllables per line is the sweet spot. Common words, open vowels on held notes. The shower test: can someone sing this after hearing it twice?
- **Suno note**: Pop is the safest default when genre is unclear. Suno handles pop structure reliably

### Indie
- **Lines per verse**: 4-6
- **What makes it work**: Conversational, slightly off-kilter phrasing. Unexpected word choices. Less polished than pop — imperfection is the point. Can break structural rules intentionally
- **Rhyme**: Slant rhyme or irregular — strict rhyme sounds too mainstream for indie
- **Singability**: Natural speech rhythm. Lines can be uneven lengths. Quirky > polished
- **Suno note**: Indie benefits from slight irregularity — vary line lengths within a verse

### Lo-fi
- **Lines per verse**: 4 (or fewer)
- **What makes it work**: Minimal, dreamy, repetitive. Lyrics are texture, not story. Short phrases that loop in the listener's head. Nostalgic, slightly hazy imagery
- **Rhyme**: Slant or assonance — nothing tight. Rhyme should feel accidental
- **Singability**: Soft, murmured delivery. Short phrases. Lots of space. Open vowels
- **Suno note**: Less text is better for lo-fi. Dense lyrics fight the chill atmosphere

### Rock
- **Lines per verse**: 4-6
- **What makes it work**: Power comes from the chorus — verses build, chorus delivers. Rhythmic drive. Can be narrative or anthemic. Energy contrast between verse and chorus is key
- **Rhyme**: ABAB or AABB — solid rhyme structure. Internal rhyme adds drive
- **Singability**: 7-10 syllables per line. Can push density higher than pop because rock vocals carry harder
- **Suno note**: Rock can handle 10 syl/line where pop maxes at 9. Use `[Powerful vocals]` or `[Belting]` for choruses

### R&B / Soul
- **Lines per verse**: 4-6
- **What makes it work**: Smooth, flowing lines with room for vocal runs and melisma. Emotional vulnerability. Repetition of key phrases with slight variation (ad-lib territory)
- **Rhyme**: Slant rhyme + internal rhyme. Tight end-rhymes work but should feel smooth, not mechanical
- **Singability**: Must flow like water. Space for vocal embellishment — don't pack every beat. Open vowels for runs (ooh, you, stay)
- **Suno note**: Leave breathing room in the lyrics for Suno to add vocal texture

### Melodic Hip-Hop
- **Lines per verse**: 6-8 (bridge between rap and singing)
- **What makes it work**: Sung hooks with rapped verses, or melodic flow throughout. More syllabic density than pop but less than pure rap. Internal rhyme is important but not as stacked as rap
- **Rhyme**: End rhyme + moderate internal rhyme. Multisyllabic rhymes elevate it
- **Singability**: Lines should be half-singable, half-rhythmic. The hook must be fully melodic
- **Suno note**: This is where Suno genre interpretation gets tricky. If Style says "hip-hop" but lyrics are sparse, Suno may still rap. Match density to intent

### Rap / Hip-Hop
- **Lines per verse**: 8-16 (standard: 16 bars)
- **What makes it work**: Dense syllabic packing. Internal rhyme, multisyllabic rhyme, assonance chains. Rhythmic precision — every syllable is a drum hit. Wordplay, double meanings, punch lines
- **Rhyme**: End rhyme on every line + stacked internal rhyme + multisyllabic chains. Rhyme density is the craft
- **Singability**: NOT singable — rapped. Consonant clusters are fine. Packed syllables are the point
- **What to avoid**: Trying to make rap lyrics "singable" — they're rhythmic speech, not melody
- **Suno note**: Max 10-12 syl/line even for rap. Suno rushes and stumbles at 14-16 syllables per line. Dense verse + sparse hook creates the best contrast

### Metal
- **Lines per verse**: 4-8
- **What makes it work**: Aggressive delivery, rhythmic precision. Dark imagery. Can alternate between screamed and clean sections. Breakdowns use short, punchy phrases
- **Rhyme**: ABAB or no rhyme — depends on subgenre. Rhyme matters less than rhythmic impact
- **Singability**: Clean sections need melodic lines. Screamed sections need short, impactful phrases. ALL CAPS for screams
- **Suno note**: Use `[Screaming vocals]` tags for harsh sections, contrast with `[Clean vocals]` for melodic sections

---

## Fusion Rules

When two genres are specified, the density spectrum determines how to spread variations:

**Density Spectrum (least → most dense):**
> Ambient → Folk → Reggae → Lo-fi → Jazz → Indie → Country → Pop → Rock → R&B → Melodic Hip-Hop → Rap → Metal

### How to handle fusions

1. **Identify both genres on the spectrum**
2. **At least one variation must lean toward the less-dense genre** — this prevents all variations defaulting to the denser genre's structure
3. **Spread across the range**:
   - Variation 1: Leans toward less-dense genre structure
   - Variation 2: True midpoint blend
   - Variation 3: Leans toward denser genre structure
4. **Use the less-dense genre's line count** as the floor and the denser genre's as the ceiling

### Example: Jazz + Hip-Hop fusion

| Variation | Lines/Verse | Syl/Line | Rhyme | Lean |
|-----------|-------------|----------|-------|------|
| Jazz-leaning | 4 | 5-7 | Loose/slant | Conversational, space for melody |
| Fusion midpoint | 6 | 7-9 | Moderate internal | Melodic flow with rhythmic edge |
| Hip-hop-leaning | 8-10 | 8-12 | Dense internal+end | Rhythmic, packed, rappable |

---

## Suno-Specific Limits

These limits apply regardless of genre — Suno's AI has processing constraints that override "correct" genre structure:

- **Max 10-12 syllables/line for rap** — Suno rushes and stumbles at 14-16 syl/line. Real rappers handle 16+; Suno can't
- **Dense verse + sparse chorus = best contrast** — Suno handles energy shifts better when the chorus is simpler than the verse
- **Jazz/soul vocals need ≤10 syl/line** — Suno interprets dense text as rap even with jazz Style tags. Keep it sparse for sung genres
- **Genre tags in Style don't override lyric density** — if the lyrics read like rap (dense, internal rhyme, packed syllables), Suno will rap them regardless of Style genre tags. The lyrics structure IS the genre signal for vocal delivery
