---
id: mem-tala-20260324-vocal-range
title: Vocal Range Headers — Mandatory for Vocal Tracks
created: 2026-03-24
tags: [suno, vocals, range, pitch, header, exclusion, register-drift, bass, baritone, tenor]
type: feedback
agent: tala
connected: []
---

Vocal tracks MUST use a three-line range header at the top of the Lyrics box, before any section tags:

```
[Voice description + texture]
[Vocal range: E2-E4]
[Exclude: falsetto, soprano register, head voice, bright tenor]
```

**Key presets (from vocals.md → Range Presets for Suno):**
- Sub-bass narrator: C2-C3 (trailers, intros)
- Bass-baritone: E2-E4 (country bass, blues, folk)
- Standard baritone: G2-G4 (rock, jazz, most genres)
- Tenor: C3-C5 (pop, theatre, rock)
- Contralto: E3-E5 (jazz, blues, torch songs)
- Soprano: C4-C6 (opera, pop ballads, ethereal)

**Why this matters:** "Deep" is an opinion, E2 is a fact. Without a range header, Suno defaults to tenor-pop and drifts higher on choruses (register drift). The exclusion list is mandatory — it's what actually prevents the drift.

**Advanced techniques in vocals.md:** proximity effect (close mic + low range + dry = max bass), single-note drone (E2-E2 for goth/doom), sub-bass narrator (strip instruments via exclusion), texture stacking (gravel/rasp + range).

**Where in workflow:** Step 8b (vocal track production rules) → pick preset. Step 8e → write header as first thing in Lyrics box.

**Source:** AI with TechNap YouTube research, validated against Suno generation behavior.
