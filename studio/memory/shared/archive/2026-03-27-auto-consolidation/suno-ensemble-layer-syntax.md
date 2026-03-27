---
name: Suno Typed Brackets — Unified Pattern
description: Typed brackets ([Energy:], [Instrument:], [Mood:], [Texture:], [Vocal Style:]) are the single pattern for ALL Suno tracks. Pipe inside [Instrument:] = play together.
type: knowledge
agent: shared
tags: [suno, typed-brackets, instrument, energy, mood, texture, vocal-style, unified]
---

# Suno Typed Brackets — Unified Pattern

## Core Rule

Typed brackets per section for ALL tracks. Pipe (`|`) inside any bracket = play together. Separate untyped bracket lines = layering conflict.

## Typed Bracket Types

- `[Energy: Low/Rising/High/Maximum]` — section energy
- `[Instrument: lead | support | rhythm]` — per-section instrumentation (pipe = ensemble)
- `[Mood: descriptor]` — section mood
- `[Texture: descriptor]` — section texture/production
- `[Vocal Style: descriptor]` — vocal direction (vocal tracks only)

## Section Order

Structure tag → Energy → Mood → Vocal Style (if vocal) → Instrument → Texture → Lyrics (if vocal)

## Style Block

Ultra-lean ~75-120 chars. Genre + BPM + key + vocal character (if vocal) + mood. All detail in Lyrics typed brackets.

## Exclude Style

Minimal or empty. Per-section `[Instrument:]` tags provide positive constraints that replace excludes.

## Layers

Intentional additions between lyrics: `[instrumental break saxophone]`, `[Whispered]`, `(Hey! Hey!)`.

Tested 2026-03-26 through 2026-03-27. Confirmed working for both instrumental and vocal tracks.
