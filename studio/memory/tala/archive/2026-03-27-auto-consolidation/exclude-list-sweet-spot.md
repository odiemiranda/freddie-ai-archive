---
title: Exclude Style sweet spot is 14 focused items
scope: self
source: user-testing-session-20260326
confidence: high
---

## Discovery: Exclude List Size

**Rule: ~14 focused instrument excludes. Drop all abstract excludes.**

### What works (14 items)
```
vocals, singing, piano, synth, guitar, bass, violin, strings, flute, trumpet, saxophone, organ, koto, shakuhachi
```
All instrument-specific. Each one prevents a concrete hallucination.

### What fails (30 items)
Adding abstract terms (aggressive, uptempo, crescendo, dramatic build, drop, bright melody, distortion) confused Suno more than helped. These are mood/energy words that Suno tries to interpret as "what NOT to feel" — which is harder than "what instrument NOT to use."

### Pattern
- Start with `vocals, singing` (always for instrumental)
- Add every melodic instrument not in the Style block (piano, synth, guitar, strings, woodwinds, brass)
- Add culture-bleed instruments if lead is traditional (koto, shakuhachi, taiko for shamisen)
- Stop there. No abstract mood/energy excludes.
