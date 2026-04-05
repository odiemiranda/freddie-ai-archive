---
id: OT-0004
title: Add AI-slop word list to lyrics critic
status: pending
created: 2026-03-26
tags: [critics, lyrics, quality]
---

# OT-0004: AI-Slop Word List

## Goal
Add concrete banned terms to the lyrics-intentionality critic. These are words/phrases that AI models default to when they lack specific imagery.

## Source
nwp/suno-song-creator-plugin `agents/quality-reviewer.md`

## Word list (from source)
- neon lights, neon glow, neon signs
- digital, pixels, screens
- echoes in the void, void, abyss
- shadows dance, dancing shadows
- electric, static, signals
- whispers in the dark
- fragments of, pieces of, shards of
- fading memories, distant memories
- lost in time, time stands still

## Genre-specific cliches
- Country: trucks, beer, dirt roads, small towns
- Pop: dancing all night, party all night, living for the moment
- Rock: breaking chains, breaking free, rebel without cause
- Indie: coffee shop romance, autumn leaves, vinyl records
- Romantic: heart on my sleeve, set me free, meant to be, falling for you

## Sub-tasks
- [ ] Add AI-slop section to lyrics-intentionality.md
- [ ] Add genre-specific cliche lists
- [ ] Cross-reference with Rune's existing anti-cliche rules (avoid duplication)
- [ ] Test with a generation to verify critic catches them
