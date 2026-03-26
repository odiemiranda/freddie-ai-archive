---
id: mem-echo-20260323-060000-01
title: Clean vs Muddled Comparison Analysis Pattern
created: 2026-03-23 06:00:00
tags: [suno, critique, comparison, generation-quality]
type: discovery
agent: echo
connected: []
---

When asked to diagnose muddled Suno output, compare against a known clean track from the same genre/instrument family. The six factors to check (ranked by impact on generation quality):

1. **Inline structure tags** — does the Style block embed [Intro] [Build] [Hook] or is it a flat wall of descriptors?
2. **Exclude breadth** — how many items? Clean tracks use 10-12. Narrow lists (5-6) let Suno bleed in unwanted instruments
3. **Mood bucket diversity** — are mood words from the same energy category (foggy+intimate+muted = all low-energy)? Clean tracks use contrast: "breezy, upbeat, warm" (energy + temperature + time)
4. **Texture budget** — count texture/warmth terms. Max 2 non-overlapping. "tape warmth" + "analog warmth" is redundant
5. **Verb type** — Style block should use directive verbs (leads, plucks, drives). Passive/poetic verbs (floating, weaving) give Suno no positional info
6. **Genre clarity** — "Reggae fusion" gives Suno a rhythm template. "Lo-fi jazzhop" is ambiguous

**Why:** First comparison session (Morning Market vs tavern tracks) validated all six factors. After applying fixes, generation quality improved from ~60% clean to consistently clean.

**How to apply:** When critiquing Style blocks via `/sound`, always check these six factors before diving into frequency analysis. The structural issues cause more muddled output than frequency collisions do.
