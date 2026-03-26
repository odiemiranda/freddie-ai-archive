---
id: mem-tala-20260321-020000-01
title: Skank Artist Name Collision
created: 2026-03-21 02:00:00
tags: [suno, artist-filter, blocked-words, reggae, guitar]
type: discovery
agent: tala
connected: []
---

Suno's artist name filter silently replaces "skank" with generic genre tags (Reggae, Ska, Rock, Pop Rock). The word matches the Brazilian band Skank.

**Why:** User discovered the replacement warning in Suno's UI when pasting a Style block containing "skank guitar chops". The replacement destroys prompt intent by substituting specific instrument direction with vague genre labels.

**How to apply:** Use safe alternatives: "offbeat guitar chop", "reggae offbeat strum", "choppy offbeat guitar". All 19 occurrences across reference files have been fixed. If new artist name collisions are discovered, add them to the blocked words table in suno.md and fix all reference files.
