---
id: mem-tala-20260324-audio-influence
title: Audio Influence Slider — Reference Audio Control
created: 2026-03-24
tags: [suno, audio-influence, slider, reference, upload, groove, persona, music-cards, settings]
type: feedback
agent: tala
connected: []
---

Suno has a third slider — **Audio Influence** — that appears when a reference audio clip is uploaded. It controls how closely the output follows the reference's rhythm, vocal phrasing, and energy. Does NOT copy exact notes — it captures feel and groove.

**The three sliders each control a different dimension:**
- Style Influence = genre purity (what genre)
- Weirdness = creative risk (how creative within that genre)
- Audio Influence = reference similarity (how close to a specific audio clip)

**Audio Influence settings by goal:**

| Goal | AI% | Use case |
|------|-----|----------|
| Loose inspiration | 10-25% | Different song, similar energy |
| Groove matching | 40-60% | Capture rhythm/phrasing, new melody |
| Persona consistency | 70-85% | Same vocal tone across album/series |
| Near-replica | 85-100% | Testing only — results feel derivative |

**Music cards pipeline:** Upload a music card's extracted audio clip as reference → set Audio Influence to guide Suno toward that groove → combine with Style block and range headers for full control.

**When NOT to use:** Most standard generations don't need a reference clip. Use Audio Influence when text descriptions in the Style block aren't precise enough to capture a specific groove, or when maintaining consistency across a series of tracks.

**Also confirmed:** W:50% is often the "most musical" sweet spot — natural phrasing without losing structure. W:0% is too mechanical, W:100% breaks form. And for extreme vocal techniques (carnatic, opera, throat singing), use W:5-15% SI:80-90%.

**Source:** AI with TechNap slider tutorial. Updated Tala agent step 8g + platform knowledge Advanced Settings section.
