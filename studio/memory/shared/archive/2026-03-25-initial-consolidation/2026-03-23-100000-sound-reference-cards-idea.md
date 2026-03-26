---
id: mem-shared-20260323-100000-01
title: Music Cards — Audio Reference System
created: 2026-03-23 10:00:00
updated: 2026-03-24
tags: [suno, minimax, reference, music-cards, audio, youtube, architecture]
type: decision
agent: shared
connected: []
---

**Evolved from "sound reference cards" → now "music cards."** Playable audio reference cards in the Obsidian vault, extracted from YouTube sources. Cards capture groove, BPM, instruments, Suno/MiniMax translations, and optionally vocal/lyrical architecture.

**Unified format with types:**
- `background-mix` — lo-fi/ambient playlists (1hr+), extract first 10 min
- `music-video` — full song with vocals, includes lyrics as structural study (never generation input)
- `single-track` — instrumental single
- `live-performance` — concert/session recording

**Card structure (all types):**
- Embedded audio sample (`![[music-cards/audio/slug.mp3]]`) for Obsidian playback
- Mix profile: BPM, key, groove, palette, production
- Track snapshots with timestamps + Suno translations
- Suno Translation: Style voice, Lyrics bracket tags, Exclude list, Known pitfalls

**Vocal cards add:**
- Emotional arc table (section → emotion → vocal delivery → arrangement)
- Phrasing map (line length, syllable density, how lyrics ride the beat)
- Space & silence map (where arrangement drops out, vocal sits alone)
- Full lyrics as structural study — Rune studies architecture, Tala studies arrangement support

**Storage:** `vault/studio/references/shared/music-cards/` for `.md` cards, `audio/` subfolder for `.mp3` clips. Audio gitignored from archive submodule (too large). iCloud syncs audio.

**Integration:** Discoverable via `discover.ts`. Tala uses matched cards as active constraints (BPM target, instrument phrasing, exclude list merge). Rune uses vocal card phrasing maps for line length and section dynamics.

**Extraction:** `yt-dlp` + `ffmpeg`. 10 min for background mixes, full track for music videos. Gemini analyzes audio for BPM/key/instruments, user ear confirms.

**Status:** Design documented at `vault/notes/inbox/music-cards.md`. Not yet built — prototype with a real YouTube link first.
