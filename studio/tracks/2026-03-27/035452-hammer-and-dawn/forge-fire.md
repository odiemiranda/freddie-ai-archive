# Forge Fire

> Part of: Hammer and Dawn

---

### Track Names

1. Forge Fire Stomp (minimax)
2. Ironclad Echo (minimax)
3. Stoneheart Drum (minimax)
4. Molten Hearth Chant (minimax)
5. Ember's Primal Roar (minimax)
6. Dwarven Ember Dirge (minimax)
7. Forgefire Stomp (gemini)
8. Stone Blood Oath (gemini)
9. Anvil Heartbeat (gemini)
10. Deep Earth Rumble (gemini)
11. Gravel Voice Chant (gemini)
12. Ember and Steel (gemini)
13. Forgefire Stomp Eternal (grok)
14. Anvil Heartbeat Roars (grok)
15. Dwarven Ember Wrath (grok)
16. Primal Hammer Chant (grok)
17. Iron Veins Ignite (grok)
18. Molten Forge Fury (grok)

### Style
```
Epic folk, baritone male vocals gravelly, group chanting, bodhrán driving stomps, low strings drone, D Minor, 78 BPM, organic, primal, clean mix, separated instruments
```

### Exclude Style
```
piano, synth, electric guitar, violin, flute, tin whistle, female vocals, auto-tune, orchestra, brass, woodwinds, electronic, bright, pop
```

### Advanced Settings
- **Weirdness**: 30%
- **Style Influence**: 75%

### Lyrics
```
[Deep bass-baritone male vocal, gravelly texture, chest voice only]
[Vocal range: E2-E4]
[Exclude: falsetto, soprano register, head voice, bright tenor, high notes, light vocals, airy]

[Short Instrumental Intro]
[Group humming, Deep, Ancient]
Hmm... hmm... hmm...
Hmm... hmm... hmm...

[Verse 1]
[Spoken word, Deep baritone, Gravelly]
His hands were wider than the anvil face.
My father. And his father before.
They'd start the forge at half past four
and the whole mountain shook by five.

[Pre-Chorus]
[Building, Group chanting joins, Deeper]
Down... down... the mine road
Down... where the torches burn in rows

[Chorus]
[Powerful chanting, Full group vocals, Stomping rhythm]
Hammer and dawn, hammer and dawn
We struck before the light came on
The mountain sang beneath our feet
And every blade knew forge-fire heat

[Verse 2]
[Spoken word, Deep baritone, Gravelly, Stripped back]
I left when the tunnels got thin.
Told myself the ceiling was too low.
Took a smith's kit and a copper ring
and walked until the sky got wide.

[Pre-Chorus]
[Building, Group chanting joins, Deeper]
Down... down... the mine road
Down... where the lanterns thinning glow

[Chorus]
[Powerful chanting, Full group vocals, Stomping rhythm]
Hammer and dawn, hammer and dawn
We struck before the light came on
The mountain sang beneath our feet
And every blade knew forge-fire heat
SING! SING! SING!

[Bridge]
[Spoken word, Deep baritone, Stripped back, Vulnerable]
The ceiling wasn't low...
The tunnels weren't too thin.
[Spoken word, Gravelly, Vulnerable]
Or maybe I outgrew them.
Or maybe they outgrew me.

[Final Chorus]
[Full chanting, Maximum power, Stomping, Ancient]
HAMMER AND DAWN, HAMMER AND DAWN
WE STRUCK BEFORE THE LIGHT CAME ON
THE MOUNTAIN SANG BENEATH OUR FEET
AND EVERY BLADE KNEW FORGE-FIRE HEAT
SING! SING! SING!

[Big Finish]
[Group humming, Fading into stone]
Hmm... hmm... hmm...
Hmm... hmm...
```

### Style Block Notes

**Charcount:** 177 chars (target ~250-350 for vocal, hard max 500). Leanest of the three variations.

**Slot validation:**
- Slot 2 (Genre): Epic folk -- same genre as V1, percussion emphasis comes from instrument behavior
- Slot 3 (Lead): baritone male vocals gravelly -- same vocal character across all 3
- Slot 3b (Arrangement): group chanting -- vocal arrangement
- Slot 4 (Support): low strings drone -- static harmonic support (removed "no melody", positive descriptor only)
- Slot 5 (Rhythm): bodhrán driving stomps -- "driving stomps" is the most aggressive rhythm descriptor of the 3 variations (V1: "stomping pulse", V2: "steady pulse")
- Slot 7 (Key/BPM): D Minor, 78 BPM -- same anchors
- Slot 9 (Mood): organic, primal -- Energy + Texture categories. "Raw" was on the dangerous vibes table (triggers distortion/grit), translated to "organic" (safe production word)
- Slot 10 (Production): clean mix, separated instruments -- mandatory

**Tri-critic summary:** Gemini 1 pass, MiniMax 1 pass, Grok 1 pass. 7 flags total. Accepted 0 -- Gemini called the Style block "perfectly optimized, no changes needed." MiniMax gave 4/5 with no structural issues. Grok flagged vocal header format and "primal" vibe risk (both rejected per reference). No revisions needed.

**Exclude rationale (14 items):** Blocks the same instrument set as V1/V2 but swaps the variation-specific risks. V3 excludes "orchestra, brass, woodwinds" instead of individual orchestral instruments -- broader block because the percussion-heavy lean means any orchestral creep would fight the bodhrán for dominance. Also blocks "bright" and "pop" to keep the primal/organic character.

**W/SI justification:** W:30% is the tightest of all three -- percussion-heavy tracks need the bodhrán locked into a steady driving pattern with zero experimental drift. SI:75% is the highest -- maximum genre adherence keeps the stomping folk character dominant and prevents Suno from softening the primal edge. This combination produces the most predictable, groove-locked variation.
