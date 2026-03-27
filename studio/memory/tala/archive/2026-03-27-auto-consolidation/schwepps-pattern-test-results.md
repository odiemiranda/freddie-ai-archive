---
title: schwepps pattern test results — all confirmed and partial
scope: self
source: user-testing-20260327
confidence: high (all user-tested in Suno)
---

## Pattern Test Results (from schwepps/skills/suno-music-creator)

### Fully Confirmed
- `[Energy: Low/Medium/Rising/High/Maximum]` — per-section energy control
- `[Energy: Medium→High]` — gradient notation
- `[Vocal Style: Open, Confident]` / `[Vocal Style: Power]` / `[Vocal Style: Belt]` — per-section vocal direction
- `[Instrument: X | Y]` — per-section instrumentation with pipes
- `[Texture: Tape-Saturated]` — per-section texture
- `[Mood: Triumphant]` — per-section mood
- `[angry verse]` / `[hopeful chorus]` — fused emotion+section, lowercase
- `~word~` — elongated notes (Suno stretches the syllable)
- `word-` — smooth cut off / stop (not a weird stop)
- `[crowd sings]` — audience participation / cheering
- `[Buildup]` — works, stackable (multiple = longer/more intense)
- `[Buildup, 4 bars]` — bar count duration hint works

### Partially Confirmed
- `[modulate up a key]` — works for 1 step up. MUST go before lyrics, not after. Numeric values (up 3, up 2) ignored — always shifts by 1.
- `[loop-friendly]` — prepares section for natural repeat, not seamless audio loop. Needs more testing in different contexts.

### Not Yet Tested
- `[Callback: continue with same vibe]` — needs Extend operation, not fresh generation

### Key Rules
- `[modulate]` tag must be BEFORE the lyrics in that section — position matters
- `[Buildup]` stacking creates longer buildups — 5 stacked = extended intro ramp
- Typed brackets + lean Style (~75 chars) outperforms verbose Style for vocal tracks
- Per-section `[Instrument:]` tags may replace Exclude lists entirely for vocal tracks
