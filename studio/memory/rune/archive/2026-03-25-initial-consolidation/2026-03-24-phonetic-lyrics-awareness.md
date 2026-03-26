---
id: mem-rune-20260324-phonetic
title: Phonetic Lyrics — Writing for Accent and Character Voices
created: 2026-03-24
tags: [suno, lyrics, phonetic, accent, spelling, regional, character, pronunciation]
type: feedback
agent: rune
connected: []
---

When Tala requests lyrics for a regional or character voice (Scottish, Jamaican, Southern, etc.), Rune should write standard English lyrics first, then provide a phonetic variant.

**Why:** Suno's dictionary normalizes all words to clean American English pronunciation. Phonetic misspelling forces Suno to render sounds manually, producing authentic accents. This is Tala's production technique but Rune needs to write lyrics that survive phonetic translation.

**What Rune should know:**
- Write singable standard lyrics first — the phonetic translation comes after
- Favor words with strong vowels and clear consonants — they translate better
- Avoid complex consonant clusters that become unreadable when phonetically respelled
- Short common words on held notes sustain better in both standard and phonetic form
- When writing for a known accent, lean into vocabulary natural to that dialect (e.g., Scottish: "bonnie", "wee", "lass" — these are already phonetically correct)

**What Rune does NOT do:**
- Rune does not write the phonetic translation — that's done via ChatGPT as a translation step
- Rune does not write the metadata header or range header — that's Tala's job
- Rune writes the emotional architecture; Tala handles the vocal production layer

**Practical tip:** When writing lyrics that will be phonetically translated, keep lines slightly shorter than usual — phonetic spelling adds characters and Suno has a 5,000 char lyrics limit.
