# Variation 1: Classic Jazz Shamisen

> Part of: Shamisen Jazz Tavern Morning

---

### Track Names

1. Steam & Silk (minimax) -- contrast pair
2. Tavern Window Gaze (gemini) -- quiet observation
3. First Brew (gemini) -- simple ritual
4. Brushed Hearthside (gemini) -- sound/texture word
5. Unfurling Quiet (gemini) -- mood caption
6. Morning Light (minimax) -- fragment
7. Warm Brew at Dawn (minimax) -- slice-of-life
8. Tavern Dawn Plucks (grok) -- object as title

### Style
```
[Instrumental] Jazz, shamisen melodic plucks, harp background pad chords, brushed drums sparse, G Major, 88 BPM, warm, clean mix, separated instruments [No Vocals]
```

### Exclude Style
```
vocals, singing, piano, synth, guitar, bass, violin, strings, flute, trumpet, saxophone, organ, koto, shakuhachi
```

### Advanced Settings
- **Weirdness**: 25% — BGM needs predictability, low creative risk
- **Style Influence**: 85% — tight genre adherence prevents late-track chaos

### Lyrics
```
[Instrumental]

[Intro | shamisen alone | bright pluck | morning air]

[Verse | shamisen jazz phrases | soft harp underneath | brushed drums lazy groove]

[Verse | shamisen leads new phrase | harp soft chord | steady groove]

[Bridge | solo shamisen | sparse plucks | silence between notes]

[Verse | shamisen relaxed | harp underneath | brushed drums settle deeper]

[Outro | shamisen rings out | last note lingers | warm fade]

[No Vocals]
[End]
```

### Style Block Notes

- **Char count**: 161 chars (lean — under 250 target, well within budget)
- **Slot validation**: Slots 1-5, 7, 9-11 present. Slot 6 (chords) dropped for budget. Slot 8 (textures) dropped — genre implies warmth.
- **Element count**: 5 core elements (genre, 3 instruments, key/BPM) — lean and clean
- **Exclude rationale**: 14 items. Focused on instrument hallucination prevention (piano, synth, guitar, strings, brass/woodwinds) plus shamisen-triggered Japanese bleed (koto, shakuhachi). Dropped abstract excludes (aggressive, uptempo, crescendo) — those confuse more than help.
- **W/SI justification**: W:25% SI:85% — BGM needs maximum predictability. High SI locks the jazz rhythm template. Low W prevents surprises that break background feel.
- **Lyrics approach**: Pipe syntax — all instruments in one bracket line per section. Separate lines cause Suno to layer/compete. Pipe keeps instruments as one ensemble. Tested through 8 iterations (V1.0-V1.4 + bracket tests A-H).
- **Key iteration findings**: "duet" framing causes equal-weight battles. "far behind" works for position. Descriptive section tags outperform rigid structure tags. Scene-based atmosphere cues ("morning air", "silence between notes") improve coherence. Chord progressions in Style invite piano hallucination.
