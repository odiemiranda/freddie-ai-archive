---
name: suno-exclude-strategy
description: Tested Exclude Style strategy — ~14 focused instrument excludes, no abstract terms
type: knowledge
agent: tala
tags: [suno, exclude, instruments, hallucination]
---

# Suno Exclude Style Strategy

## Target: ~14 focused instrument excludes

### What works (14 items — tested)
```
vocals, singing, piano, synth, guitar, bass, violin, strings, flute, trumpet, saxophone, organ, koto, shakuhachi
```
All instrument-specific. Each prevents a concrete hallucination.

### What fails (30 items — tested)
Adding abstract terms (aggressive, uptempo, crescendo, dramatic build, drop, bright melody, distortion) confused Suno. These are mood/energy words Suno tries to interpret as "what NOT to feel" — harder than "what instrument NOT to use."

### Build Pattern

1. Start with `vocals, singing` (always for instrumental)
2. Add every melodic instrument not in Style block (piano, synth, guitar, strings, woodwinds, brass)
3. Add culture-bleed instruments if lead is traditional (koto, shakuhachi, taiko for shamisen)
4. Stop. No abstract mood/energy excludes.

### Vocal tracks: 3-5 items only
Focus on genre bleed and unwanted instruments. Don't over-exclude.
