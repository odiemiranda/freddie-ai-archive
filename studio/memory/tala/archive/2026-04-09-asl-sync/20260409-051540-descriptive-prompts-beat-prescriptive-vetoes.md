---
source: reflection
agent: tala
task: "Created variant of The Last Iron — cinematic Celtic neofolk dirge variant 'Unbroken Roots'. 6 generations to land. First 4 used prescriptive constraint-stacking and failed. Gen 5 succeeded after structural rebuild mirroring source prompt."
workflow: create-track-variant
outcome: Track kept at gen 6. Final prompt structure mirrors original Celtic folk source — descriptive, positive, 16 elements, zero negatives in main prompt, named instruments with active verbs, named reverb, production words.
scope: self
confidence: high
tags: [prompt-structure, suno-vocal, vetoes, descriptive-vs-prescriptive, source-prompt-mirroring, six-iteration-spiral]
session: 2026-04-09T05:15:40
---

## Observation
Building a cinematic low-voice variant of The Last Iron, gens 1-4 failed in opposite directions despite stacking 7+ negative directives in Style and Lyrics staging (`no falsetto`, `no head voice`, `no harmonized highs`, `no pitch lift`, `no melodic ascent`, `no vocal climb`, `narrow range stays in deep octave`, `low pedal tone`, `measured glacial cadence`, `ritualistic pacing`). Symptoms cycled: too fast/clean → spoken-word collapse → falsetto reaching → monotone reading.

The spiral broke when the user pasted the source track's original Style block and asked me to study it. The source had ZERO negative directives, achieved baritone+slow+ancient with pure positive description, and used a coherent genre space (Celtic folk + tavern ballad) that already had the desired qualities natively.

## Learning
**In Suno, descriptive prompts beat prescriptive prompts. Stacked negative directives in the main Style block fight Suno's training and produce unstable, oscillating failures.**

Working WITH Suno's training means:
1. **Pick a genre/sub-genre that natively has the qualities you want.** "Celtic neofolk dirge" already implies slow, low, gruff, ancient, melodic-but-not-bright. You don't need to forbid falsetto if you choose a genre that doesn't reach for it.
2. **Mirror successful source-prompt structure precisely.** The Last Iron source: 15 elements, descriptive, named instruments with active verbs ("bodhran drum pounding", "Celtic harp arpeggios answering the chant"), named reverb ("raw cavernous reverb"), production words ("warm but defined", "separated instruments", "analog warmth"), two-layer vocal (lead + group harmony), simple mood phrases. The variant prompt was rebuilt at 16 elements with the same shape and worked immediately.
3. **Move excludes to Exclude Style only.** The main prompt should describe what you want; the Exclude Style is for the truly intolerable. Slim it. Source had implicit excludes; my failed prompts had 16-veto exclude lists that did nothing the genre choice didn't already do.
4. **Lyrics staging tags follow the same rule.** 3-5 elements per tag, title case, active verbs, atmospheric/role descriptors. NO `no falsetto`, `narrow range`, `stays in deep octave` in staging — these are vetoes pretending to be staging and they cause the same failures as Style-block vetoes.

## Evidence
- Source prompt (worked immediately): `Celtic folk, deep chanting baritone male vocals, ancient and earthy, group chanting harmonies, stomping rhythm, bodhran drum pounding, Celtic harp arpeggios answering the chant, D Minor, 75 BPM, tavern ballad, primal and powerful, raw cavernous reverb, warm but defined, separated instruments, analog warmth`
- Failed gen 4 (16 vetoes total): `gravelly weathered basso profundo sung bass, low narrow range stays in deep octave, close-up mic proximity effect, sustained long low notes, cinematic Celtic folk, low string drone never drops, bodhran stomp constant, low brass swells, C Phrygian, 68 BPM, primal solemn` + Exclude `falsetto, head voice, harmonized highs, bright vocals, soaring melody, pitch lift, melodic ascent, vocal climb, female vocals, whistle, fiddle lead, electric guitar, synth, auto-tune, pop, uptempo`
- Working gen 5 rebuild (mirrored source structure): `Celtic neofolk dirge, deep gravelly basso profundo male vocals, sustained low chant phrases, group bass chant harmonies low and unified, slow stomping ritual rhythm, bodhran drum pounding deep, low cello and double bass drone underneath, distant low brass swells, C Phrygian, 68 BPM, ancient ritual lament, primal subterranean and solemn, raw cavernous cathedral reverb, close-mic warmth, separated instruments, analog warmth` + Exclude `female vocals, electric guitar, synth, auto-tune, pop, bright melody, falsetto, whistle, fiddle lead`
- The structural rebuild also fixed `cinematic Celtic folk` — a fusion that doesn't exist as a coherent space in Suno's training, which had been forcing the model to invent the combination every generation.
