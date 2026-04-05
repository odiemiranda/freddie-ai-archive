---
name: Wick's Prompt Engineering Strategies
description: Details Wick's advanced strategies for structured prompt generation, including the MiniMax prose builder and specialized Suno vocal lyrics creation.
type: knowledge
agent: wick
tags: [prompt-generation, strategy, prose-builder, minimax, structured-prompts, testing, vocal-lyrics, suno, schema, parser, formatter]
---

## Knowledge
Wick employs advanced strategies for generating high-quality, structured prompts and specialized content.

**MiniMax Prompt Strategy:**
Wick has successfully implemented the "Phase 4 MiniMax strategy," an advanced 6-section prose builder for prompt generation. This strategy is encapsulated in `minimax.ts` and has been thoroughly tested with `minimax-strategy.test.ts`, where all 21 tests passed. It is integrated into `index.ts`. Additionally, a regression test against Suno prompts showed a 30/30 pass rate, confirming its robustness and compatibility.

**Suno Vocal Lyrics Strategy:**
Wick has implemented a comprehensive strategy for generating structured vocal lyrics specifically for Suno. This includes the `VocalLyricsInput` schema, `SECTION_TAG_ALIASES`, and the core `suno-vocal-lyrics.ts` strategy. The strategy encompasses a parser, voice header generation, typed bracket handling, and a formatter. The `buildVocalLyrics()` function in `index.ts` and a CLI option (`--build vocal-lyrics` in `prompt-builder.ts`) facilitate its use. All associated tests have passed.

## Why
The MiniMax strategy provides a structured and robust method for generating complex, multi-section prose prompts, ensuring high-quality and consistent output. The Suno vocal lyrics strategy enables Wick to produce correctly formatted vocal lyrics tailored for the Suno platform, ensuring compatibility and optimal performance for music prompts. Both strategies reduce the need for manual refinement and streamline content creation.

## How to apply
Leverage the MiniMax strategy for generating detailed, multi-section prose prompts, especially when a structured narrative or complex prompt architecture is required. Utilize the `buildVocalLyrics()` function or the CLI `--build vocal-lyrics` option to generate structured vocal lyrics for Suno, ensuring adherence to the `VocalLyricsInput` schema for consistent output.

## Consolidated from
- `wick-minimax-prompt-strategy.md`
- `wick-suno-vocal-lyrics-strategy.md`
