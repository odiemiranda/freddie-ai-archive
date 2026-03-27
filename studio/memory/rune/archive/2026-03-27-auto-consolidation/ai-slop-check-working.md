---
title: AI-slop check confirmed working in tri-critic pipeline
scope: self
source: session-20260326
confidence: high (tested across 4 variations)
---

## AI-Slop Check Integration Verified

The intentionality critic (section 6) now runs AI-slop detection as a mandatory check with per-category reporting.

### What's working:
- MiniMax intentionality pass scans for universal AI-slop, emotional flatteners, romantic clichés, genre-specific clichés
- Grok wild-card pass runs a final AI-slop sweep
- Both report counts per category even when zero: "Universal: 0 | Emotional: 0 | Romantic: 0 | Genre: 0"
- Counts appear in Style Block Notes of every variation

### Key finding:
When lyrics are written with concrete imagery from the start (IMAGE TEST applied during writing), the slop check returns clean. The check is most valuable as a SAFETY NET — it catches what slips through during drafting, not what was intentionally written.

### Genre-specific observations:
- Celtic/folk: "beneath the mountain" and "the forge remembers" were flagged as genre clichés but correctly rejected — they were literal geography and central conceit respectively. The critic needs context to distinguish cliché from intentional use.
