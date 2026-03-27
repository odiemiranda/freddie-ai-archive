# Tavern Stone

> Part of: Hammer and Dawn

---

### Track Names

1. Iron Hearth's Echo (minimax)
2. Forge's Anvil Chant (minimax)
3. Stonefoot Tavern Song (minimax)
4. Ember's Stomping Hymn (minimax)
5. Dwarven Anvil's Roar (minimax)
6. Cinders Under the Mountain (minimax)
7. Anvil's Heartbeat (gemini)
8. Stone Hearth Revel (gemini)
9. Deep Earth's Pulse (gemini)
10. Gravel Voice Oath (gemini)
11. Brewmaster's Chant (gemini)
12. Iron Mountain's Call (gemini)
13. Forgefire Anvil Chant (grok)
14. Dwarven Hammer Heart (grok)
15. Embered Depths Roar (grok)
16. Stoneblood Tavern Oath (grok)
17. Grimforge Echoes Rise (grok)
18. Molten Veins Pulse (grok)

### Style
```
Epic folk, baritone male vocals gravelly, group chanting, bodhrán stomping pulse, low strings harmonic bed, D Minor, 78 BPM, ancient, earthy, clean mix, separated instruments
```

### Exclude Style
```
piano, synth, electric guitar, violin, flute, tin whistle, female vocals, auto-tune, modern drums, hi-hat, snare, electronic, bright, pop
```

### Advanced Settings
- **Weirdness**: 35%
- **Style Influence**: 70%

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

**Charcount:** 184 chars (target ~250-350 for vocal, hard max 500). Well under budget -- lean and focused.

**Slot validation:**
- Slot 2 (Genre): Epic folk -- tested genre tag from The Forge archetype
- Slot 3 (Lead): baritone male vocals gravelly -- vocal character with texture
- Slot 3b (Arrangement): group chanting -- vocal arrangement element
- Slot 4 (Support): low strings harmonic bed -- positive role descriptor (removed "no melody")
- Slot 5 (Rhythm): bodhrán stomping pulse -- single rhythm source with behavior
- Slot 7 (Key/BPM): D Minor, 78 BPM -- both present
- Slot 9 (Mood): ancient, earthy -- from 2 categories (Era + Texture)
- Slot 10 (Production): clean mix, separated instruments -- mandatory keywords present

**Tri-critic summary:** Gemini 1 pass, MiniMax 1 pass, Grok 1 pass. 11 flags total. Accepted 1: removed "deep chanting" from vocal identity to prevent Style biasing all sections toward singing (Lyrics tags handle per-section switching). Rejected 7 (exclude reduction, BPM removal, genre bracket, vocal header consolidation, genre rename). Deferred 2 (cave reverb for testing, gravelly distortion risk). Key consensus: all three praised the exclude list and structure tags.

**Exclude rationale (14 items):** Blocks every common folk instrument Suno adds unbidden (violin, flute, tin whistle), prevents modern instrument drift (piano, synth, electric guitar, modern drums/hi-hat/snare), locks gender (female vocals), blocks production drift (auto-tune, electronic, bright, pop). The Forge archetype was tested with similar excludes on 2026-03-22.

**W/SI justification:** W:35% keeps the stomping rhythm steady and predictable -- chanting needs a locked groove, not experimental drift. SI:70% locks the deep group chanting character and prevents drift into clean melodic singing. Both values align with The Forge archetype tested range (W:35-45%, SI:60-70%).
