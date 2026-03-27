---
title: Suno Quirks & Troubleshooting
tags: [suno, quirks, troubleshooting, vocal-leakage, tags, blocked-words, iteration, iterate, pivot, abandon, genre-gotchas, sandwich-method]
---

# Suno Quirks & Troubleshooting

Known platform behaviors, workarounds, and debugging checklist.

## Vocal Leakage

Suno sometimes adds vocals even with `[Instrumental]` and `[No Vocals]`. Workarounds:
- Use BOTH `[Instrumental]` at start AND `[No Vocals]` at end
- Add `[Background Music]` as third reinforcement
- Avoid lyrical words in structure descriptions — "singing melody" triggers vocals; use "soaring melody"
- Lower Style Influence (30-40%) for stubborn cases

## Reliable vs Unreliable Tags

| High Reliability (85-95%) | Unreliable / Platform Ignores |
|----------|------------|
| `[Instrumental]`, `[No Vocals]`, `[Background Music]` | `[Accelerando]`, `[Ritardando]` |
| `[Short Instrumental Intro]` (90%), `[Big Finish]` (85%) | `[Intro]` (30-40%), `[Outro]` (40%), `[Fade Out]` (35%) |
| `[Verse]` (90%), `[Chorus]` (95%), `[Bridge]` (85%) | `[Key Change]`, `[Break]` (45%) |
| `[Catchy Hook]` (90%), `[melodic interlude]` (85%) | Exact BPM (±5 BPM is normal) |
| `[Final Chorus]`, `[Mood: X]` (75-85%) | Exact song duration, specific instrument solos |

## When to Iterate vs Pivot vs Abandon

| After... | If results are... | Action |
|----------|------------------|--------|
| 2-3 gens | Mostly right | **Iterate** — tweak 1 element |
| 4-6 gens | Consistent flaw | **Pivot** — change the problematic element entirely |
| 6-8 gens | Consistently wrong | **Simplify** — strip to 30 words, rebuild |
| 8+ gens | Nothing usable | **Abandon** — try the other prompt style |

## Quick Fixes Checklist

1. Prompt under 1000 characters?
2. Contradictory descriptors?
3. Instruments competing for same frequency range?
4. Genre combination realistic? (Max 2 compatible genres)
5. `[Instrumental]` at START and `[No Vocals]` at END?
6. Important elements front-loaded?

## Blocked Words & Artist Name Collisions

Suno has two filters that silently replace or remove words:

1. **Content filter** — blocks words flagged as inappropriate
2. **Artist name filter** — replaces any word matching a known artist name with generic genre tags. This happens silently and ruins prompt intent

**Rule:** Never use direct artist names in Style blocks. Describe the *sound* instead. The artist reference files (`artists-*.md`) contain pre-translated descriptions for 700+ artists.

| Blocked Word | Why Blocked | Safe Alternatives |
|-------------|-------------|-------------------|
| `skank` | Artist name + content filter | `offbeat guitar chop`, `reggae offbeat strum`, `choppy offbeat guitar` |

**Artist name avoidance:**
- Don't write "Norah Jones style" → `smoky female jazz vocal, intimate, piano-led, late-night`
- Don't write "Dropkick Murphys style" → `Celtic punk, raspy male vocals, gang chorus, driving fiddle`
- Don't write "Nujabes style" → `jazzy lo-fi hip-hop, mellow piano samples, boom-bap, nostalgic`

## Genre-Specific Gotchas

- **"punk" causes shorter songs** — Suno interprets punk's fast tempos and short song tradition literally. Use "post-hardcore" or "pop punk" for full-length tracks
- **"progressive metalcore"** triggers power-metal shredding when you want thick rhythm guitars. Use "metalcore" + specific instrument descriptions instead
- **Suno adds its own dynamics** unless you explicitly say "no lulls, full intensity" or "consistent energy"

## Tag Length Rule

Short tags (1-3 words) on their own lines are the most reliable. Verbose compound tags (4+ words) risk being sung as lyrics or ignored entirely. Always prefer high-reliability tags over long descriptive tags.

## Instrument Isolation (Sandwich Method)

For solo instrument focus, BOTH Style prompt AND Exclude Styles are needed:
1. Place the target instrument at the START and END of Style prompt
2. Block competing instruments in Exclude Styles
