---
name: Gemini Suno Critique Filter Rules
description: Guidelines for filtering Gemini's critique responses to ensure suggestions align with Suno-specific prompt behavior.
type: knowledge
agent: echo
tags: [gemini, critique, filtering, trust, suno]
---

## Overview
Gemini provides high-confidence structural observations but its specific production "tweaks" often contradict Suno's prompt interpretation. Filter all suggestions through these reliability patterns.

## Filter Patterns

### Reliably Good (Accept)
- **Frequency Collisions**: Warnings about instruments occupying the same register.
- **Missing Structure**: Observations regarding absent inline structure tags.
- **Descriptor Redundancy**: Catching duplicate or overly similar mood/texture words.
- **Tag Counts**: Warnings about exceeding instrument or genre limits.

### Usually Bad (Reject)
- **Per-Instrument Mixing**: Suggestions like "rolled-off lows for harp." Suno's Style block is a general prompt, not a mixing console.
- **Production Effects**: Keywords like "lush reverb" or specific effects. Suno often interprets these by muddying the mix.
- **Wholesale Rewrites**: Never apply a full Gemini rewrite. **Mine individual fixes** and incorporate them into the original prompt.
- **Atmospheric Stripping**: Do not remove scene-setting or atmospheric language (e.g., "cold morning by a tavern window"). This provides emotional context that Suno utilizes effectively.

## Why
Gemini's general music knowledge is excellent, but it does not inherently understand Suno's specific quirk of interpreting "production" keywords as "mix-mud." Structural/count observations are high-confidence; mixing suggestions are low-confidence.

## How to Apply
Cross-reference every Gemini suggestion against the `suno-generation-critique-checklist.md` and `production.md` hard rules. Accept structural/count observations; be skeptical of per-instrument mixing suggestions.

**Consolidated From**:
- `mem-echo-20260323-060000-02` (Gemini Suggestion Filter — What to Accept and Reject)
- `mem-shared-20260321-183900-01` (Gemini Critique Loops Live - partial overlap)
