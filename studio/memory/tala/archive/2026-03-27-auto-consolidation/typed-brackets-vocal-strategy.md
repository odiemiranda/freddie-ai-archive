---
title: Typed brackets for vocal tracks — lean Style + tag-rich Lyrics
scope: self
source: user-testing-session-20260327
confidence: high (tested in Suno, live concert quality confirmed)
---

## Discovery: Typed Brackets for Vocal Tracks

**Strategy: Ultra-lean Style (~75 chars) + rich per-section typed brackets in Lyrics.**

### Tested and confirmed working:
- `[Energy: Medium]`, `[Energy: High]`, `[Energy: Rising]`, `[Energy: Maximum]`
- `[Vocal Style: Open, Confident]`, `[Vocal Style: Power]`, `[Vocal Style: Belt]`
- `[Instrument: Bright electric guitars | live drums building]`
- `[Texture: Tape-Saturated]`
- `[Mood: Anticipation]`, `[Mood: Triumphant]`

### Unified bracket syntax:
```
[(type:) value | value | value]
```
- Type prefix + colon = typed bracket (Energy, Vocal Style, Instrument, Texture, Mood)
- No type prefix = ensemble line (our existing pipe syntax for instrumentals)
- Both coexist in the same Lyrics block

### Dual strategy:
- **Instrumental tracks:** Lean Style (~140-170 chars) + ensemble pipe Lyrics (proven)
- **Vocal tracks:** Ultra-lean Style (~75-120 chars) + tag-rich Lyrics with typed brackets per section

### Why this works:
Suno reads Lyrics tags more carefully than Style descriptors for per-section control. Putting instruments, energy, texture, and vocal style IN the Lyrics gives Suno section-level precision. The Style block just sets the global genre/BPM/mood foundation.

### Source:
schwepps/skills suno-music-creator — user tested their example, confirmed live concert quality with audience cheering, voice control, and instrument precision.
