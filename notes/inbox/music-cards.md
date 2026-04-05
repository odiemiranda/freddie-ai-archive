---
id: idea-20260324
title: Music Cards — Audio Reference System
tags: [idea, music-cards, reference, audio, suno, tala, rune, discover]
created: 2026-03-24
status: brainstorm
---

# Music Cards — Audio Reference System

Playable audio reference cards in the Obsidian vault that capture the groove, BPM, style, and instrument palette from YouTube sources. Cards serve as both inspiration and active constraints during Tala/Rune generation workflows.

## Problem

Expressive words in Suno lyrics tags are imprecise. "Gentle picking" produces different results every generation. We can't reliably control instrument behavior (picking speed, density, phrasing) through vibes alone. The shamisen mix work exposed this — too slow, too much plucking, inconsistent groove feel across generations.

Music cards ground the creative process in a **known-good audio reference** instead of guessing.

## Concept

A music card is a markdown note with an embedded audio sample, technical analysis, and pre-built Suno/MiniMax translations. Open the note in Obsidian, hit play, read the translations — everything in one place.

### Unified format, typed by source

All cards share the same base structure (sample, profile, snapshots, Suno translation, pitfalls). Cards with vocals add extra sections (emotional arc, phrasing map, lyrics). The `type` field tells agents what to expect — no different logic needed, just "does this card have vocal architecture? use it if so."

**Types:**

| Type | Source | Vocals | Extract |
|------|--------|--------|---------|
| `background-mix` | Lo-fi/ambient playlist (1hr+) | No | First 10 min |
| `music-video` | Full song with vocals/visuals | Yes — lyrics + delivery study | Full track (3-5 min) |
| `single-track` | Instrumental single, not a mix | No | Full track |
| `live-performance` | Concert/session recording | Maybe | Best 10 min or full |

**Vocal sections** (present only when type has vocals):
- Emotional arc — how mood shifts across sections, vocal delivery changes
- Phrasing map — line lengths, syllable density, how lyrics land on the beat
- Space & silence — where arrangement drops out, where vocals sit alone
- Lyrics — full text as structural study reference, **never** as generation input

Rune and Tala use vocal sections to understand *how* a song is built — where emotion lands, how phrasing rides the beat, where space creates impact. The lyrics are an architectural blueprint, not content to copy.

## Card Format

```markdown
---
id: mc-YYYYMMDD-HHMMSS
title: "Jazzhop Shamisen Tavern Mix"
type: background-mix
tags: [music-card, jazzhop, shamisen, lo-fi, tavern]
created: YYYY-MM-DD
source: https://youtu.be/...
source_variant: single-artist | curated-playlist
extracted: first 10 minutes
---

# Jazzhop Shamisen Tavern Mix

## Sample
![[music-cards/audio/jazzhop-shamisen-tavern-YYYY-MM-DD.mp3]]

## Mix Profile
| Field | Value |
|-------|-------|
| BPM | 82-88 |
| Key | Dm / Am |
| Groove | swung 8ths, lazy pocket |
| Palette | shamisen, rhodes, vinyl texture, muted drums |
| Production | mid-heavy, soft compression, lo-fi filtering, tape warmth |
| Mix type | single-artist session |

## Track Snapshots
| Time | What happens | Suno Translation |
|------|-------------|------------------|
| 0:45 | shamisen enters, sparse 2-bar plucks over boom-bap | `rhythmic plucked shamisen, 2-bar phrases` |
| 4:20 | harp + piano duet, straight 8ths, no drums | `gentle arpeggiated harp, soft piano chords` |
| 8:15 | full arrangement, layered but clean separation | `full ensemble, clear mix, each instrument distinct` |

## Vocal & Lyrical Architecture (vocal cards only)

### Emotional Arc
| Section | Emotion | Vocal Delivery | Arrangement |
|---------|---------|---------------|-------------|
| Verse 1 | intimate, confessional | soft, close-mic, conversational | sparse — piano + vocal only |
| Pre-chorus | tension building | rising pitch, shorter phrases | drums enter, bass swells |
| Chorus | release, anthemic | full voice, open vowels | full band, layered harmonies |
| Verse 2 | reflective, deeper | same as V1 but with breath catches | adds subtle strings |
| Bridge | vulnerability, raw | near-whisper, cracking | stripped to guitar + vocal |
| Final chorus | cathartic | belt, ad-libs | everything, then instruments drop one by one |

### Phrasing Map
- Verse: short lines (4-6 words), land on beats 1 and 3, lots of space between
- Chorus: long flowing phrases (8-12 words), syncopated, words spill across bar lines
- Bridge: fragmented — two short lines, pause, one long line

### Space & Silence
- 0:45 — full 2-beat rest before chorus drops (tension)
- 2:10 — instruments cut, vocal alone for 4 bars (intimacy)
- 3:30 — bridge ends on held note, 1 bar silence, then final chorus hits

### Lyrics (study reference — not for generation)
```
[Verse 1]
(full lyrics here, formatted with section headers)

[Chorus]
...
```

### What Rune/Tala should learn from this
- The verse→chorus energy jump comes from arrangement change, not just vocal volume
- Short phrasing in verses creates conversational intimacy — longer chorus phrases feel like release
- The bridge strips everything back — the silence before the final chorus is what makes it hit
- Ad-libs in the final chorus aren't random — they echo key phrases from verse 1

---

## Suno Translation
**Style voice:**
- "rhythmic plucked shamisen, steady pulse, sparse phrasing"
- "warm vinyl lo-fi, swung groove, muted drums"

**Lyrics bracket tags:**
- [melodic shamisen, steady rhythm]
- [warm rhodes chords, soft pad]
- NOT: [shamisen solo], [rapid picking], [virtuosic]

**Exclude list:**
- rapid picking, tremolo, solo, virtuosic, free-form, improvised

**Known pitfalls:**
- Double "lead" emphasis causes shamisen to go solo
- "lazy" translates to aimless phrasing — use "melodic" instead
- Busy drums push shamisen to fill gaps — keep arrangement sparse

## Notes
(free-form observations, what makes this mix special, what to steal)
```

## Workflow: Creating a Music Card

1. **User drops a YouTube link** — lo-fi/background mix, typically 1hr+
2. **Extract first 10 minutes** — `yt-dlp` with time flag, convert to mp3 (128kbps, ~10MB)
3. **Save audio** — `vault/studio/references/shared/music-cards/audio/{slug}.mp3`
4. **Analyze together** — Echo (Gemini) analyzes audio for BPM/key/instruments + user's ear confirms
5. **Write the card** — technical description + Suno translations + pitfalls (manual, collaborative)
6. **Save card** — `vault/studio/references/shared/music-cards/{slug}.md`
7. **Discoverable** — `discover.ts` picks it up via TOC/keyword matching

### Why 10 minutes is enough
- Lo-fi mixes cycle through 5-10 tracks in the first 30 min, then loop
- Each track establishes groove within 30-60 seconds
- Palette (instruments, texture, production) is consistent across the whole mix
- BPM range is usually tight (+-5 BPM)
- 10 min captures ~3-5 distinct tracks — enough for the sonic fingerprint

### Single-artist vs. curated playlist
- **Single-artist sessions** make better cards — coherent palette throughout
- **Curated playlists** may jump between styles — flag this on the card
- Curated mixes might warrant multiple cards (one per distinct style section)

## Integration with Tala/Rune

### Discovery (step 1)
Tala already runs `discover.ts` in step 1. Music cards in the references folder get surfaced when keywords match (e.g., "shamisen", "jazzhop", "tavern"). The card's Suno Translation section gives Tala ready-to-use Style voice and bracket tags.

### Constraint during build
When a music card is matched:
- BPM from the card becomes the target (not guessed)
- Instrument phrasing descriptions replace vibe words
- Exclude list from the card merges with genre excludes
- Known pitfalls are checked against the draft

### Rune lyrics integration
- Bracket tags from the card inform `[instrument]` tags in lyrics
- Phrasing descriptions guide section dynamics (sparse verse, full chorus)
- Card's groove feel influences rhythmic word choices
- **Vocal cards:** emotional arc informs where to place heavy vs. light lyrics, phrasing rhythm guides line length and syllable density per section, space/silence map tells Rune where to leave breathing room with bracket tags like `[pause]` or `[instrumental break]`

### Tala arrangement integration
- **Vocal cards:** arrangement column shows how instruments support vocal delivery — when to strip back, when to layer. Tala uses this to write Style blocks that create the right space for vocal moments
- Dynamic shifts (sparse verse → full chorus) become explicit arrangement instructions rather than hoping Suno figures it out

### Audio Influence integration (reference clip upload)
- Music card audio clips can be uploaded directly to Suno as reference audio
- Suno's **Audio Influence** slider controls how closely the output follows the reference's rhythm, vocal phrasing, and energy
- **Recommended settings per goal:**
  - Loose inspiration (capture the vibe, new everything else): AI 10-25%
  - Groove matching (same rhythm/feel, new melody): AI 40-60%
  - Faithful recreation (maintain character across tracks): AI 70-85%
- This creates a direct pipeline: YouTube → extract audio → music card → upload to Suno → Audio Influence guides generation
- Optional — not every generation needs a reference clip. Use when the card's text descriptions aren't precise enough and the actual audio groove matters

## Storage

```
vault/studio/references/shared/music-cards/
  jazzhop-shamisen-tavern.md          <- card (markdown, discoverable)
  lo-fi-piano-rain.md
  audio/                              <- extracted clips
    jazzhop-shamisen-tavern.mp3       <- ~10MB per card
    lo-fi-piano-rain.mp3
```

### Backup considerations
- **Card `.md` files** — backed up to archive submodule (small, text)
- **`audio/` folder** — gitignored from archive submodule (too large for git)
- **iCloud sync** handles audio files — syncs to phone, available in Obsidian mobile
- If git backup is ever needed for audio: git-lfs is an option but not needed now

## Technical: Audio Extraction

```bash
# Mix card — extract first 10 minutes from a long lo-fi/background mix
yt-dlp -x --audio-format mp3 --audio-quality 128K \
  --download-sections "*0:00-10:00" \
  -o "vault/studio/references/shared/music-cards/audio/%(title)s.mp3" \
  "https://youtu.be/..."

# Vocal card — extract full track (music videos are typically 3-5 min)
yt-dlp -x --audio-format mp3 --audio-quality 128K \
  -o "vault/studio/references/shared/music-cards/audio/%(title)s.mp3" \
  "https://youtu.be/..."
```

Requirements:
- `yt-dlp` — YouTube downloader (pip install or brew)
- `ffmpeg` — audio conversion (yt-dlp dependency)

For vocal cards, lyrics can be sourced from:
- YouTube subtitles/CC (auto or manual) — `yt-dlp --write-subs --sub-lang en`
- Genius, AZLyrics, or similar (manual copy)
- Gemini transcription from the audio file

## Technical: Gemini Audio Analysis

Echo's gemini-proxy already supports `--input-file`. Gemini can analyze audio directly:
- BPM detection (approximate, +-3 BPM — verify with ear or Essentia)
- Key detection
- Instrument identification
- Groove/feel description
- Production characteristics

For precise BPM: Essentia (Python) or Moises.ai as fallback.

## Open Questions

- [ ] Should Tala auto-surface matching music cards, or only when user references them?
- [ ] Do we need a `tools/soundcard.ts` CLI (like remember.ts) or is discover.ts enough?
- [ ] Card tagging taxonomy — freeform tags or controlled vocabulary?
- [ ] Should cards link to tracks they influenced? (bidirectional: card <-> track)
- [ ] Multiple cards per mix (one per standout track) vs. one card per mix?
- [ ] Vocal card lyrics: auto-transcribe via Gemini, pull from YouTube CC, or manual paste?
- [ ] Should vocal cards tag which emotions/dynamics Rune borrowed? (traceability)
- [ ] Stem separation (Moises.ai) worth it for vocal cards? Isolated vocals reveal delivery nuance better

## Status
Idea documented. Prototype when ready — start with one card from the jazzhop YouTube link to validate the format and workflow.
