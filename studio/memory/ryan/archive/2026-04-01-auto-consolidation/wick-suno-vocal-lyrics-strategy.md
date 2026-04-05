---
name: Wick's Suno Vocal Lyrics Strategy
description: Details Wick's capability to generate structured vocal lyrics for Suno, including schema, parser, and formatter.
type: knowledge
agent: wick
tags: [prompt-generation, vocal-lyrics, suno, strategy, schema, parser, formatter]
---

## Knowledge
Wick has implemented a comprehensive strategy for generating structured vocal lyrics specifically for Suno. This includes the `VocalLyricsInput` schema, `SECTION_TAG_ALIASES`, and the core `suno-vocal-lyrics.ts` strategy. The strategy encompasses a parser, voice header generation, typed bracket handling, and a formatter. The `buildVocalLyrics()` function in `index.ts` and a CLI option (`--build vocal-lyrics` in `prompt-builder.ts`) facilitate its use. All associated tests have passed.

## Why
This capability enables Wick to produce high-quality, correctly formatted vocal lyrics tailored for Suno, ensuring compatibility and optimal performance with the Suno platform. It streamlines the creation of vocal components for music prompts.

## How to apply
Utilize the `buildVocalLyrics()` function or the CLI `--build vocal-lyrics` option to generate structured vocal lyrics for Suno. Ensure adherence to the `VocalLyricsInput` schema for consistent output.

## Consolidated from
- `raw-log-20260331-162743-note.md`
