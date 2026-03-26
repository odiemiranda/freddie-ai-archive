---
id: mem-tala-20260323-060000-03
title: Broad Exclude Lists Prevent Instrument Bleed
created: 2026-03-23 06:00:00
tags: [suno, exclude-style, generation-quality]
type: discovery
agent: tala
connected: []
---

Morning Market excludes 11 items (vocals, singing, tin whistle, fiddle, flute, saxophone, horn, brass, piano, aggressive drums, catchy hook) and generates clean. Tavern tracks originally excluded 5-6 items and had muddled output. Suno was likely bleeding in extra instruments that weren't excluded.

**Why:** Suno's Exclude Style is a negative prompt. A narrow exclude list leaves gaps for Suno to "hallucinate" competing instruments into the mix. Piano pads, synth layers, or whistles can bleed in if not explicitly banned.

**How to apply:** Default to 10-12 exclude items for instrumental/background tracks. Always exclude: vocals, singing, plus any instrument in the same frequency register as the lead that isn't named in the Style block. Use Morning Market's exclude list as the baseline template for shamisen + harp arrangements.
