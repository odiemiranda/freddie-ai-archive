---
id: OT-0002
title: Add genre co-occurrence data and gravity wells
status: pending
created: 2026-03-26
tags: [suno, genres, references, exclude]
---

# OT-0002: Genre Co-occurrence Data

## Goal
Quantify genre gravity wells so the exclude strategy is data-driven. Know which genres pull in unwanted sounds and how strong the pull is.

## Source
nwp/suno-song-creator-plugin `references/genre-clouds.md`

## Key data points
- Pop gravity: Rock→Pop 315B links, Funk→Pop 116B, Emo→Pop 12.2B
- Inseparable clouds: Rap/Trap/Bass/Hip-Hop, Orchestral/Epic/Cinematic, Indie/Pop/Acoustic
- Weak tags (easily overwhelmed): Grunge, Math rock, Swing, Chamber music, Bluegrass
- Strong tags (dominate): Pop, Rock, Electronic, Hip-hop, Classical
- Escape strategies: explicit exclusion, weird combos, repelling pairs

## Sub-tasks
- [ ] Create `vault/studio/references/suno/genre-gravity.md`
- [ ] Map gravity wells with co-occurrence strengths
- [ ] Define repelling pairs
- [ ] Update exclude strategy guidance in suno.md
