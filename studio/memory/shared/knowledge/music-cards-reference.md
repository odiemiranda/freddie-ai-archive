---
name: Music Cards Reference
description: Architecture for the playable audio reference system in Obsidian.
type: knowledge
agent: shared
tags: [reference, music-cards, audio, youtube, suno, minimax]
---

# Music Cards: Audio Reference System

Music Cards are playable Markdown files in Obsidian that link real-world audio samples to AI generation constraints.

## Card Types
- `background-mix`: Long-form lo-fi/ambient (extract first 10 min).
- `music-video`: Full vocal tracks used for structural study (phrasing/density).
- `single-track`: Instrumental references.
- `live-performance`: Session recordings for groove/human-feel.

## Structure & Storage
- **Location:** `.md` cards in `vault/studio/references/shared/music-cards/`; `.mp3` clips in `audio/` subfolder (gitignored).
- **Content:**
    - Embedded audio snippet for Obsidian playback.
    - **Mix Profile:** BPM, key, groove, palette.
    - **Suno/MiniMax Translation:** Style voice, bracket tags, exclude lists.
    - **Vocal Map (for videos):** Emotional arc tables, phrasing maps (syllable density), and space/silence maps.

## Workflow Integration
- **Tala:** Uses matched cards to set BPM targets and instrument phrasing.
- **Rune:** Uses phrasing maps to ensure lyrics "ride the beat" correctly.
- **Extraction:** Process involves `yt-dlp` + `ffmpeg`, followed by Gemini analysis for technical metadata (BPM/Key).

**Consolidated from:**
- `2026-03-23-100000-sound-reference-cards-idea.md`
