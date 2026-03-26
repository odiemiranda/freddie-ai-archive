---
id: mem-tala-20260325-morning-window-01
title: Morning Window Groove — generation testing findings
created: 2026-03-25
tags: [suno, shamisen, harp, testing, generation-quality, neutral-labels, spacing, hierarchy, bracket-length]
type: discovery
status: validated-against-morning-market
agent: tala
connected: []
---

Testing the "morning window groove" track (groove pop, shamisen + harp + soft pop drums, D Major) revealed multiple generation quality issues and fixes.

## Confirmed fixes

1. **Neutral instrument labels** — "harp" produces cleaner output than "celtic harp". The cultural label triggers genre-associated instruments (violin, fiddle, strings) even with broad excludes. Use neutral labels and describe the sound with adjectives instead.

2. **Distinct roles in Style palette** — "shamisen melody, harp chords" works better than listing both as equals or using "duet". Giving each instrument a clear function (melody vs harmony) in the Style block prevents Suno from putting them in competitive rhythmic patterns.

3. **"spacious" keyword** — adding this to the Style block gives breathing room between plucked instruments that share frequency ranges.

4. **100 BPM over 105 BPM** — slower tempo lets two plucked instruments coexist without stepping on each other.

5. **Cooperation over competition in Lyrics** — "breathing together" works better than "trading phrases" or "call-and-response" for instruments in similar registers. The latter pushes Suno into alternating solos rather than ensemble playing.

## Unresolved issues

1. **Harp solo breakout at ~75% playback** — Suno improvises solo sections during sparse Lyrics sections. Adding "no solos" and "consistent energy" cues helped but didn't fully fix it.

2. **Blank spaces / drops** — sections with too few bracket tags leave gaps where Suno doesn't know what to do. May need more consistent section density.

3. **Weird instrument mixing** — occasional moments where harp and shamisen clash despite fixes. May be a fundamental Suno limitation with two plucked instruments in similar registers.

## Validated Style block formula (2026-03-25 evening)

Style = sonic palette + instrument behavior + dynamic control. No structure tags, no staging.

Working Style:
```
[Instrumental] Jazz fusion, shamisen melodic phrases, harp low-register pad chords as distant warmth, soft drums with lazy swing, Dm7 to Gmaj7 to Em7 to A7, D Major, 88 BPM, tape warmth, rolled-off highs, relaxed, unhurried, loop-friendly, clean mix, separated instruments [No Vocals] [Background Music]
```

Dropped "subtle melodic movement" and "consistent dynamics" — caused constant noodling and flat jam session feel. Replaced with "relaxed, unhurried".

With Lyrics block: much better, controlled, clean harp behavior.
Without Lyrics block: still harp strumming issues — confirms Lyrics block is essential for arrangement control.

## Still testing

- "melodic phrases" may push all instruments toward active melodic playing. Try "shamisen smooth melody" or "shamisen lead melody" instead — keep melodic identity on shamisen only.
- The harp strumming without lyrics suggests "pad chords" isn't strong enough to keep harp passive. May need stronger language.

## Next steps to try

- Replace "shamisen melodic phrases" with "shamisen smooth melody" or "shamisen lead melody"
- Test whether "harp sustained low chords" prevents strumming better than "pad chords"
- Test with Lyrics block always (confirmed essential for arrangement)
