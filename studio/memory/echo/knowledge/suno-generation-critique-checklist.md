---
name: Suno Generation Critique Checklist
description: Six-factor diagnostic pattern for identifying and fixing muddled output quality in Suno AI Style blocks.
type: knowledge
agent: echo
tags: [suno, critique, checklist, generation-quality]
---

## Overview
When diagnosing muddled or low-quality Suno output, compare the failing Style block against a known clean track from the same genre or instrument family. Use these six factors to identify structural issues that often outweigh frequency collisions in their impact on generation quality.

## The Seven Factors (Ranked by Impact)

1.  **Style block completeness**: Style block needs sonic palette + instrument behavior + dynamic control. Bare instrument names without behavior hints ("shamisen" instead of "shamisen melodic phrases") cause Suno to guess — leading to chaos. But NO structure tags (`[Intro]`, `[Build]`) — those go in Lyrics.
2.  **Support instrument hierarchy**: Support instruments need subordination in BOTH Style ("harp low-register pad chords as background warmth") AND Lyrics brackets ("far behind", "no melody of its own"). Without hierarchy cues, Suno gives all instruments equal presence and they compete.
3.  **Exclude Breadth**: With typed `[Instrument:]` brackets in Lyrics, Exclude Style can be minimal or empty — positive per-section instructions replace negative constraints. Only add dealbreakers. Without typed brackets, fall back to ~14 focused instrument excludes.
4.  **Mood Bucket Diversity**: Ensure mood descriptors aren't all from the same energy category (e.g., all low-energy terms like "foggy, intimate, muted"). Use contrast: Energy + Temperature + Time (e.g., "breezy, upbeat, warm").
5.  **Texture Budget**: Limit to a maximum of 2 non-overlapping texture/warmth terms. Redundant terms (e.g., "tape warmth" + "analog warmth") muddy the signal.
6.  **Verb Type**: Use directive verbs (e.g., "leads", "plucks", "drives") rather than passive or poetic ones (e.g., "floating", "weaving") to provide positional information.
7.  **Genre Clarity**: Use simple rhythm templates (e.g., "Jazz", "Reggae") over compound labels (e.g., "Jazz fusion", "Lo-fi jazzhop"). Simpler genre labels = cleaner output.

## Why
Analysis of "Morning Market" vs "Tavern" tracks confirmed that structural factors are the primary cause of muddled output. Applying these fixes improved generation quality from ~60% to consistent cleanliness.

## How to Apply
Before performing a frequency-based analysis via `/sound`, validate the Style block against this six-factor checklist. Address structural failures first.

**Consolidated From**:
- `mem-echo-20260323-060000-01` (Clean vs Muddled Comparison Analysis Pattern)
