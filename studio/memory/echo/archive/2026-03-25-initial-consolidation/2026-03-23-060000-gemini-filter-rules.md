---
id: mem-echo-20260323-060000-02
title: Gemini Suggestion Filter — What to Accept and Reject
created: 2026-03-23 06:00:00
tags: [gemini, critique, filter, trust]
type: feedback
agent: echo
connected: []
---

When filtering Gemini's critique responses, these patterns emerged:

**Reliably good suggestions (accept):**
- Frequency collision warnings between instruments in the same register
- Missing structure tag observations
- Mood descriptor redundancy catches
- Texture count warnings

**Usually bad suggestions (reject):**
- Per-instrument frequency directives ("rolled-off lows for harp") — Suno's Style block is a general prompt, not a mix console
- "Lush reverb" or production effect keywords — risk muddying the mix, Suno handles reverb poorly
- Wholesale lyrics rewrites — mine individual fixes, never apply Gemini's full rewrite
- Removing scene/atmosphere language from Style blocks ("cold morning by a tavern window") — the scene gives Suno emotional context

**Why:** First comparison analysis showed Gemini's structural observations are high-confidence but its specific production tweaks often contradict how Suno actually interprets prompts.

**How to apply:** Cross-reference every Gemini suggestion against the six-factor checklist and production.md hard rules. Accept structural/count observations. Be skeptical of per-instrument mixing suggestions.
