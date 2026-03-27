---
name: suno-lyrics-ensemble-layer
description: Ensemble/Layer framework for Suno Lyrics — pipe syntax, bracket rules, tested patterns
type: knowledge
agent: tala
tags: [suno, lyrics, pipe-syntax, ensemble, layer, bracket, instrumental]
---

# Suno Lyrics: Ensemble / Layer Framework

## Core Rule

Pipe (`|`) = play together. New bracket line = new layer.

## Ensemble (pipe syntax)

The instrumental bed. Always first line of every section. Default for ALL tracks.

```
[Verse | shamisen jazz phrases | soft harp underneath | brushed drums lazy groove]
```

Suno reads one instruction → instruments collaborate as one unit.

**Pipe order:** lead + action → support + position → rhythm + feel → atmosphere cue (optional)

**Max 4-6 modifiers per pipe.** More = noise.

## Layer (separate bracket lines)

Intentional additions on top of the ensemble. Layers can also use pipes internally.

```
[instrumental break saxophone | soft echo]
[Guitar phrase | single pluck]
[Whispered]
(Hey! Hey!)
```

## Track Type Rules

- **Instrumental = ensemble lines only.** No layers, no separate instrument lines.
- **Vocal = ensemble + lyrics + selective layers.**

## What Fails (tested)

- **Separate instrument lines** → 3 layers competing, chaos builds toward end
- **"Duet" framing** → equal weight = instruments battle
- **Over-describing each instrument** → Suno tries to layer everything simultaneously
- **Position-only instructions** ("harp far behind") without action → weaker than scene-based cues

## What Works (tested)

- **"Shamisen jazz phrases"** — action gives Suno something to do
- **"soft harp underneath"** — position + softness modifier
- **Atmosphere cues** ("morning air", "silence between notes", "warm fade") — mood per section
- **Support solo sections** — harp owns intro/bridge, shamisen owns verses. Dedicated space > shared space.

## Reference Pattern (instrumental)

```
[Instrumental]
[Intro | harp low arpeggios alone | warm room]
[Verse | shamisen jazz phrases | harp arpeggios underneath | brushed drums easy groove]
[Bridge | harp arpeggios alone | low register | quiet space]
[Verse | shamisen returns relaxed | harp underneath | brushed drums settle]
[Outro | harp arpeggios fade | shamisen last note rings | warm air]
[No Vocals]
[End]
```
