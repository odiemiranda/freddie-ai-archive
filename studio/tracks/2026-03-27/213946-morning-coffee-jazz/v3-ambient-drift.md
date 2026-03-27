# Ambient Drift

> Part of: Morning Coffee Jazz

---

### Track Names

Generated via `bun src/tools/tracknames.ts` — ALL 18 names with model source and naming strategy labels.

1. Soft Amber Hours (minimax) *-- mood caption*
2. Silent Page Turn (minimax) *-- simple ritual*
3. Quiet Ember Drift (minimax) *-- contrast pair*
4. Silken Silence Fold (minimax) *-- quiet observation*
5. Warm Stillness Breathes (minimax) *-- inner thought*
6. Barely There, Dreaming (minimax) *-- fragment*
7. Unfurling Stillness (gemini) *-- sound/texture word*
8. Shamisen Page Drifts (gemini) *-- object as title*
9. Barely There Glow (gemini) *-- mood caption*
10. Harp of Quietude (gemini) *-- object as title*
11. Warmth Before Silence (gemini) *-- contrast pair*
12. A Minor Reverie (gemini) *-- fragment*
13. Fading Page Glow (grok) *-- mood caption*
14. Shamisen Whisper (grok) *-- sound/texture word*
15. Stillness Unfurls (grok) *-- inner thought*
16. Harp's Quiet Drift (grok) *-- object as title*
17. Silent Ink Reverie (grok) *-- quiet observation*
18. Warm Void Echo (grok) *-- contrast pair*

### Style
```
[Instrumental] Lo-fi jazz, shamisen sparse plucks, harp low arpeggios, A Minor, 78 BPM, warm unhurried, clean mix, separated instruments [No Vocals]
```

### Exclude Style
```
vocals, singing, electronic, drums, percussion, auto-tune
```

### Advanced Settings
- **Weirdness**: 45% -- ambient focus needs some creative room for texture play, but not enough to break the stillness
- **Style Influence**: 60% -- moderate adherence; the sparse arrangement needs Suno to fill space naturally, not mechanically

### Lyrics
```
[Instrumental]

[Short Instrumental Intro]
[Energy: Very Low]
[Mood: Serene, barely awake]
[Instrument: Harp single low note | long sustain | silence between]
[Texture: Vinyl hiss, constant and soft]

[Verse]
[Energy: Very Low]
[Mood: Deep reading quiet]
[Instrument: Shamisen single pluck far apart | harp low arpeggio underneath slow | both sparse, no rush]
[Texture: Vinyl crackle steady, almost silent]

[melodic interlude]
[Energy: Very Low]
[Mood: Contemplative]
[Instrument: Harp single note returns alone]
[Texture: Vinyl crackle alone]

[Verse]
[Energy: Very Low]
[Mood: Warm, unhurried]
[Instrument: Shamisen slow pluck | harp low chord sustains underneath | space between everything]

[Sustained]
[Energy: Fading to silence]
[Instrument: Harp last low note sustains | shamisen gone]
[Texture: Vinyl crackle fades to nothing]

[No Vocals]
[End]
```

### Style Block Notes

- **Char count**: 148 chars (within instrumental target, under 250 hard max)
- **Slot validation**: pass -- all slots in order, no bleed, genre + lead front-loaded
- **Element count**: 8 elements (1 over optimal 7 -- acceptable for the sparse arrangement needing explicit space cues)
- **Tri-critic summary**: Gemini 2 passes, MiniMax 1 pass, Grok 1 pass. 4/9 flags accepted. Key changes: added "sparse" to shamisen descriptor, tightened vinyl texture language
- **Exclude rationale**: minimal set -- drums/percussion excluded to protect ambient silence, vocals/singing/auto-tune standard instrumental protection, electronic prevents synth bleed
- **W/SI justification**: W:45% gives Suno room to interpret the sparse space naturally without breaking stillness. SI:60% moderate adherence -- too high would make Suno fill every beat, defeating the ambient drift purpose
