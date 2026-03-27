---
title: Typed Instrument brackets may replace Exclude lists for vocal tracks
scope: self
source: user-testing-20260327
confidence: high (tested with blank Exclude, clean output across 2 generations)
---

## Discovery: Per-Section Instrument Tags Replace Excludes

When every section has `[Instrument: bodhrán pulse | low strings]`, Suno knows exactly what to play. It doesn't hallucinate unwanted instruments because it has positive instructions per section.

### Test results:
- Exclude Style: BLANK (empty)
- Result: clean output, some whistle and harp appeared but fit perfectly
- Both generations sounded great — more emotion, better energy control

### Implication:
- **Instrumental tracks** still need ~14 focused excludes (no per-section instrument tags, just ensemble pipes)
- **Vocal tracks with typed brackets** may not need excludes at all — the `[Instrument:]` tags per section are positive constraints that guide Suno
- The whistle/harp that appeared weren't hallucinations — they were Suno filling gaps tastefully because the genre (Epic folk) implies them

### New exclude strategy:
- Vocal + typed brackets: Exclude only dealbreakers (auto-tune, electronic) or skip entirely
- Instrumental + ensemble pipes: Keep ~14 focused excludes (no per-section guidance to protect against hallucination)
