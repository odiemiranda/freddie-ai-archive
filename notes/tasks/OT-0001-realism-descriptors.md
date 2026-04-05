---
id: OT-0001
title: Integrate realism descriptors for acoustic genres
status: in-progress
created: 2026-03-26
tags: [suno, production, references, acoustic]
---

# OT-0001: Integrate Realism Descriptors

## Goal
Add recording-engineer vocabulary to Suno references for acoustic/folk/jazz/blues/classical genres. Physical reality descriptors ("close mic presence", "fret squeak") produce better results than vibe words ("realistic", "authentic").

## Source
nwp/suno-song-creator-plugin `references/realism-descriptors.md`

## Scope

### Reference file
Create `vault/studio/references/suno/realism-descriptors.md` with:
- Room acoustics (small room, room tone, bedroom acoustics)
- Microphone characteristics (close mic, off-axis, proximity effect)
- Performance detail (one-take, natural timing drift, micro-rubato)
- Breath/body (breath detail, mouth noise, body shift)
- Instrumental performance (pick noise, fret squeak, finger movement, string vibration)
- Recording imperfections (light mic handling noise, environmental bleed, noise floor)
- Analog character (tape saturation, wow and flutter, tube warmth)
- Spatial (limited stereo, mono, narrow stereo, short room reverb)

### 3-tier stacking
- Minimal: close mic, natural dynamics, room tone
- Moderate: + breath detail, pick noise, short room reverb, limited stereo
- Maximum: + full detail (20+ descriptors)

### Genre gating
- ESSENTIAL: acoustic, folk, jazz, blues, classical, singer-songwriter
- OPTIONAL: rock, country, R&B
- COUNTERPRODUCTIVE: electronic, hip-hop, synthwave, metal, pop

### Agent integration
- Update suno.md to reference the file when genre matches
- Add to discover.ts tag matching so it surfaces for acoustic genres
- Consider adding as optional slot in the 11-slot formula

## Sub-tasks
- [ ] Write the reference file
- [ ] Add frontmatter tags for discover.ts
- [ ] Update suno.md with genre-gated guidance
- [ ] Test with an acoustic folk generation
