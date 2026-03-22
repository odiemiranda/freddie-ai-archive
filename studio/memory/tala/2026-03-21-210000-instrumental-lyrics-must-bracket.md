---
id: mem-tala-20260321-210000-01
title: Instrumental Lyrics Must Bracket
created: 2026-03-21 21:00:00
tags: [suno, instrumental, brackets, vocal-leakage, lyrics]
type: feedback
agent: tala
connected: []
---

Every descriptive line in instrumental lyrics MUST be wrapped in [brackets]. Unbracketed text is treated by Suno as lyrics to sing, causing vocal leakage on instrumental tracks.

**Why:** Generated "Shamisen plucks a bright phrase, alone" without brackets in chillhop-reggae track. Suno attempted to sing the descriptive lines instead of treating them as instrument direction. User caught it during testing.

**How to apply:** For instrumental tracks, structure tags ([Intro], [Build], etc.) AND all direction lines must use brackets: `[Shamisen plucks a bright phrase]` not `Shamisen plucks a bright phrase`. This is in addition to [Instrumental] at start and [No Vocals] at end.
