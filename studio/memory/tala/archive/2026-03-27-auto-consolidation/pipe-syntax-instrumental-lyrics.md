---
title: Pipe syntax for instrumental Lyrics blocks
scope: self
source: user-testing-session-20260327
confidence: high (tested through 8+ iterations, consistent results)
---

## Discovery: Pipe vs New Lines in Instrumental Lyrics

**Rule: Use pipe-stacked brackets for instrumental sections. Never put instruments on separate bracket lines.**

### What works (ensemble — instruments play together)
```
[Verse | shamisen jazz phrases | soft harp underneath | brushed drums lazy groove]
```
Suno reads one instruction with modifiers → one scene → instruments collaborate.

### What fails (layers — instruments compete)
```
[Verse]
[Shamisen jazz melodies]
[Harp far behind]
[Brushed drums]
```
Suno reads 3 separate instructions → 3 independent layers → instruments fight for space.

### Pattern
```
[Section tag | lead instrument + action | support instrument + position/texture | rhythm + feel]
```
Optional atmosphere cue as final modifier: `[Intro | shamisen alone | bright pluck | morning air]`

### Additional findings from this session

1. **Position words work for support**: "far behind", "underneath", "soft chord" — not "no melody", "barely audible"
2. **"duet" framing fails**: Suno treats both instruments as equal leads → battle
3. **Scene cues improve coherence**: "morning air", "silence between notes", "warm fade" give Suno atmosphere context
4. **Chord progressions in Style invite hallucination**: Jazz extended chords (Gmaj7-Em7-Cmaj9-D7) caused piano to appear. Dropping chords and letting genre imply harmony = cleaner output
5. **Lean Style blocks win**: 161 chars outperformed 291 chars for the same track. Less info = less confusion
6. **Exclude list sweet spot**: ~14 focused items. 30 items was noise. Drop abstract excludes (aggressive, uptempo, crescendo) — keep instrument-specific ones only
7. **Descriptive section tags**: `[Bridge | solo shamisen | sparse plucks | silence between notes]` outperforms rigid `[melodic interlude]` + separate direction lines
8. **"Jazz" not "Jazz fusion"**: Simpler genre label = cleaner output
