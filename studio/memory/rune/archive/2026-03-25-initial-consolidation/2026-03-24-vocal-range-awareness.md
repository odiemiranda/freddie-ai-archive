---
id: mem-rune-20260324-vocal-range
title: Vocal Range Awareness — Phrasing for Register
created: 2026-03-24
tags: [suno, vocals, range, phrasing, register, lyrics, bass, baritone, tenor, soprano]
type: feedback
agent: rune
connected: []
---

Tala now enforces vocal range headers on all vocal tracks (three-line header at top of Lyrics box with note range + exclusion list). Rune doesn't write the header — that's Tala's job during the build phase. But Rune should be aware of the target register when writing lyrics.

**Why it matters for lyrics:**
- **Low registers (bass/baritone, E2-G4):** Favor short common words with long vowels (oh, home, low, stay). Consonant clusters (strengths, scratched) choke low voices. Sparse phrasing — fewer words per line, more space between phrases.
- **High registers (tenor/soprano, C3-C6):** Can handle faster syllable density and longer melodic phrases. Open vowels (ay, ee, oh) sustain well on held high notes.
- **Phrasing rhythm should match the register's natural delivery.** A bass voice sings slower, heavier — write lines that breathe. A soprano floats — write lines that flow.

**Practical rule:** When Tala specifies a voice type in the track request (e.g., "deep male vocals", "female alto"), Rune should factor the register into syllable density and line length choices. No need to reference vocals.md directly — just match phrasing weight to voice weight.

**Music cards (future):** Vocal-type music cards will include phrasing maps showing how real songs distribute syllable density and line length per section at a given register. Rune should use these as structural study references when available.
