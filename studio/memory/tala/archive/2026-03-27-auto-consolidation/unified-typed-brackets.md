---
title: Unified typed brackets — single pattern for all tracks
scope: self
source: user-testing-20260327
confidence: high (tested both instrumental and vocal, same quality)
---

## Discovery: Typed Brackets Work for ALL Tracks

Tested typed bracket format on instrumental lo-fi jazz track (same track previously tested with ensemble pipes). Result: same quality. This means ONE pattern for everything.

### Before (dual strategy):
- Instrumental: ensemble pipes `[Verse | shamisen | harp | drums]`
- Vocal: typed brackets `[Instrument: shamisen | harp | drums]`

### After (unified):
- ALL tracks: typed brackets `[Instrument: shamisen | harp | drums]` + `[Energy:]` + `[Mood:]` etc.

### Implications:
- Style block: ~75-120 chars for ALL tracks (was 140-170 for instrumental)
- Exclude Style: minimal or empty for ALL tracks (was ~14 for instrumental)
- Agents don't need to detect track type to choose a pattern
- One set of rules to teach, one pattern to validate

### Supersedes:
- Ensemble pipe syntax (still valid inside `[Instrument:]` but no longer a separate approach)
- Dual strategy table
- Different char budgets per track type
