---
name: fusion-frequency-control
description: Multi-genre fusion role-mapping strategy, frequency darkening for bright instruments, three-level low-pass application.
type: knowledge
agent: tala
tags: [suno, fusion, frequency, lowpass, genre-mapping, bright-instruments, instrument-variants]
---

# Fusion & Frequency Control

## 1. Multi-Genre Fusion Strategy (3+ Source Genres)
Don't list all genres as genres — Suno can't process that many genre signals. Map each source to a specific role:

| Role | What it controls | Style Slot |
|---|---|---|
| Primary genre | Rhythm template, structure, feel | Slot 2 |
| Rhythmic feel | Drum pattern, groove character | Slot 5 |
| Lead instrument | Cultural color, melodic identity | Slot 3 |
| Support instrument | Background texture, harmonic bed | Slot 4 |

**Example:** "Lo-fi Jazzhop + Reggae + Shamisen + Celtic harp" becomes:
- Genre = Lo-fi Jazzhop (slot 2)
- Rhythm = one-drop (slot 5)
- Lead = shamisen melodic plucks (slot 3)
- Support = harp low-register drone, no melody (slot 4)

Two genre words max. Rest is instrumentation.

## 2. Frequency Darkening for Bright Instruments
When a fusion includes naturally bright instruments (shamisen, tin whistle, banjo, sitar), control highs with production tags.

**Keywords:** `lowpass filtering`, `muffled highs`, `rolled-off highs`

**Instrument variant selection:** Choose lower-pitched variants when available:
- "low whistle" not "tin whistle"
- "bass shamisen" not "shamisen"
- "nylon guitar" not "steel string guitar"

## 3. Three-Level Low-Pass Application

| Level | Where | Example | When |
|---|---|---|---|
| Global | Style block | `lowpass filtering` | Darken entire mix (lo-fi, chillhop, ambient) |
| Per-instrument | Style block | `low-pass jazz piano loop` | Only the support needs darkening |
| Per-section | Lyrics brackets | `[lowpass breakdown]`, `[low-pass sweeps into chorus]`, `[outro loop with low-pass fade]` | Transitional darkening for builds, breakdowns, endings |
