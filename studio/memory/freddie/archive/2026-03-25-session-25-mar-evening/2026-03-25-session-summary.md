---
name: 2026-03-25 session — music cards + memory consolidation + generation fixes
description: Major session building music cards pipeline, memory consolidation tool, and fixing Suno Style block rules
type: project
agent: freddie
tags: [music-cards, memory-consolidation, suno, style-block, generation-quality]
---

## Music cards system (built)

Full pipeline: YouTube URL → yt-dlp download → ffmpeg trim → librosa audio analysis (BPM, key, energy, spectral) → metadata genre profiling → Gemini refinement → Obsidian card with embedded audio.

- `tools/musiccard.ts` — end-to-end card creation
- `tools/analyze-audio.py` — librosa analysis (BPM, key, energy, spectral brightness, rhythm density)
- Discovery integration — `discover.ts` scans frontmatter tags, outputs `[music-card]` tag
- All agents updated with music card usage instructions
- Two test cards created: coffee-lofi-cafe-song, groove-pop-laid-back-vol13-a

## Memory consolidation system (built)

Three-tier structure: inbox → knowledge → archive.

- `tools/consolidate-memory.ts` — Gemini-powered analysis, contradiction detection, theme clustering
- `tools/remember.ts` updated — saves to inbox, reads from knowledge
- All agents consolidated: 39 inbox files → 17 knowledge files
- Fixed Windows renameSync bug (copy+delete instead)

## Suno Style block fix (critical)

Root cause of muddled generation: structure tags (`[Intro]`, `[Build]`, `[Hook]`) were in the Style block. Fixed across:
- Agent files (suno.md, gemini.md)
- 3 memory files (contradictory findings corrected)
- 6 reference files (examples updated, deprecation notes added)
- Rule: Style = sonic palette only. Lyrics = structure + arrangement.

## Generation testing (in progress)

Morning window groove track tested iteratively. Key findings saved to Tala's inbox. Neutral instrument labels, distinct roles, spacious keyword, and cooperation language all improved output. Still unresolved: harp solo breakouts, blank spaces, occasional instrument clashing.

**Why:** This session established the infrastructure for grounded music generation (music cards with real audio data) and clean agent memory (no more contradictions poisoning behavior).
