# Shamisen-Forward

> Part of: Morning Coffee Jazz

---

### Track Names

Generated via `bun src/tools/tracknames.ts` — ALL 18 names, with model source AND naming strategy labels.

1. Reading at Dawn (minimax) *— simple ritual*
2. Amber Haze Lull (minimax) *— contrast pair*
3. A-Minor Morning Mist (minimax) *— mood caption*
4. Shamisen in Fog (minimax) *— object as title*
5. Morning Ember (minimax) *— fragment*
6. Lo-Fi Dawn Plucks (minimax) *— sound/texture word*
7. Amber Glow Awakening (gemini) *— contrast pair*
8. Unfurling Silk Strings (gemini) *— quiet observation*
9. A Minor Dawn Drift (gemini) *— mood caption*
10. First Sip Warmth (gemini) *— simple ritual*
11. Shamisen's Gentle Bloom (gemini) *— quiet observation*
12. Quiet Harp Reverie (gemini) *— object as title*
13. Coffee Haze Awakening (grok) *— simple ritual*
14. Shamisen Dawn Sip (grok) *— fragment*
15. Low Harp Reverie (grok) *— object as title*
16. Unrushed Morning Pluck (grok) *— sound/texture word*
17. Anticipatory Steam Whisper (grok) *— inner thought*
18. Lazy Focus Ember (grok) *— contrast pair*

### Style
```
[Instrumental] Lo-fi jazz, shamisen slow plucks, harp low-register, A Minor, 82 BPM, laid-back, clean mix, separated instruments [No Vocals]
```

### Exclude Style
```
vocals, singing, electronic, drums, percussion, auto-tune
```

### Advanced Settings
- **Weirdness**: 35% — lo-fi background/study track needs consistency, no surprises. Instruments stay in pocket.
- **Style Influence**: 70% — strong genre adherence for lo-fi jazz feel. Suno commits to the dust and jazz chords without drifting.

### Lyrics
```
[Instrumental]

[Short Instrumental Intro]
[Energy: Low]
[Mood: Morning quiet]
[Instrument: Shamisen alone | single slow pluck]
[Texture: Vinyl hiss, barely there]

[Verse]
[Energy: Low]
[Mood: Unhurried]
[Instrument: Shamisen relaxed jazz plucks | harp low-register pad chords underneath | gentle harmonic motion]
[Texture: Vinyl crackle, almost silent]

[melodic interlude]
[Energy: Low]
[Mood: Still, anticipatory]
[Instrument: Harp low arpeggios alone | space between notes]

[Verse]
[Energy: Low]
[Mood: Gentle forward motion]
[Instrument: Shamisen returns relaxed | harp pad chords underneath]

[Verse]
[Energy: Fading]
[Instrument: Shamisen last pluck hangs | harp low chord sustains]
[Texture: Vinyl crackle fades]

[No Vocals]
[End]
```

### Style Block Notes

- **Char count**: 140 chars (instrumental target ~75-120, acceptable for 2-instrument lo-fi jazz)
- **Slot validation**: pass — slots 1-11 in order, no bleed, genre + lead front-loaded
- **Element count**: 8 elements (slightly over 3-7 optimal, but typed brackets carry per-section detail)
- **Tri-critic summary**: Gemini 2 passes, MiniMax 1 pass, Grok 1 pass. 4/6 flags accepted. Key changes: tightened mood to single descriptor in Style, confirmed no drums is intentional for ambient focus
- **Exclude rationale**: minimal 6 items — typed [Instrument:] brackets provide positive per-section constraints. Vocals/singing/electronic are dealbreakers for acoustic instrumental. Drums/percussion excluded since no rhythm source intended.
- **W/SI justification**: W:35% keeps lo-fi background consistent for focus/reading. SI:70% locks Suno to lo-fi jazz conventions — dust, jazz voicings, laid-back groove.
