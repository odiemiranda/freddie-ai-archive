---
name: Music Cards — audio reference system
description: Playable audio reference cards from YouTube with librosa analysis, genre profiling, Gemini refinement, and discovery integration
type: project
agent: freddie
tags: [music-cards, audio, reference, youtube, suno, minimax, groove, bpm, vocal, lyrics, architecture]
---

Audio reference card system — built and integrated. Cards live in `vault/studio/references/shared/music-cards/`.

**Pipeline (tools/musiccard.ts):**
1. yt-dlp downloads audio from YouTube
2. ffmpeg trims to 5-min MP3 sample (configurable --start, --duration)
3. librosa analyzes audio (tools/analyze-audio.py) — real BPM, key, energy, spectral brightness, rhythm density
4. Metadata genre matching fills groove, palette, mood from hardcoded profiles
5. Gemini refines the card with genre knowledge + audio data (skippable with --no-analyze)

**Naming:** Card slug is ~32 chars from title (e.g. `groove-pop-laid-back-vol13-a.md`). Audio file prefixed with unix timestamp for uniqueness (e.g. `1774393666-groove-pop-laid-back-vol13-a.mp3`).

**Card types** (unified format, one `type` field):
- `background-mix` — lo-fi/ambient playlists, 5-min extract, groove + palette focus
- `music-video` — full song with vocals, lyrics as structural study
- `single-track` — instrumental single
- `live-performance` — concert/session recording

**Discovery:** `discover.ts` scans frontmatter tags and title. Cards appear as `[music-card]` in output. Agents load full card and use Profile values (BPM, key) as hard constraints.

**Storage:** `.md` cards backed up to archive submodule. `audio/` folder gitignored in archive — iCloud handles audio sync.

**Why:** Expressive words in Suno tags are imprecise. Music cards ground the creative process in known-good audio references with measured data and pre-translated platform vocabulary.

**Not yet built:** Vocal card support (lyrics as architectural blueprint), Rune integration for phrasing study.
