---
title: Pipe syntax prevents instrument layering conflicts (Suno)
scope: shared
source: user-testing-session-20260327
confidence: high
---

## Shared Learning: Suno Instrumental Bracket Syntax

For Suno instrumental tracks, pipe-stacked brackets keep instruments as an ensemble. Separate bracket lines cause them to layer and compete.

**Ensemble (works):** `[Verse | shamisen jazz phrases | soft harp underneath | brushed drums lazy groove]`
**Layers (fails):** Separate `[Shamisen...]` `[Harp...]` `[Drums...]` lines

This applies to all Suno instrumental generation, not just shamisen/harp tracks.
