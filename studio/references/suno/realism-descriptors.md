---
title: Realism Descriptors
type: reference
platform: suno
tags: [realism, acoustic, recording, production, microphone, performance, analog, folk, jazz, classical, singer-songwriter]
---

# Realism Descriptors

Physical reality descriptors produce better results than vibe words. "Close mic presence" beats "realistic." "Fret squeak" beats "authentic guitar." Suno responds to recording-engineer language because it describes measurable physical properties, not subjective feelings.

**Genre gating — when to use:**

| Use | Genre | Why |
|-----|-------|-----|
| Essential | Acoustic, folk, jazz, blues, classical, singer-songwriter | These genres are defined by their physical sound. Without realism cues, Suno defaults to polished/digital. |
| Optional | Rock, country, R&B | Can add authenticity but not required for genre identity. |
| Counterproductive | Electronic, hip-hop, synthwave, metal, pop | These genres are defined by processing. Realism cues fight the genre's nature. Use synthesis descriptors instead. |

## Categories

### Room Acoustics
```
small room, room tone, bedroom acoustics, natural room sound,
live room, studio live room, short room reverb, early reflections,
ambient mic bleed, room resonance
```

### Microphone Characteristics
```
close mic presence, off-axis placement, proximity effect,
single-mic capture, condenser clarity, ribbon mic warmth,
dynamic mic grit, room mic blend, overhead capture
```

### Performance Characteristics
```
one-take, natural timing drift, natural dynamics, human micro-rubato,
slight tempo wander, performance imperfections, live feel,
unrehearsed looseness, confident hesitation
```

### Breath and Body Detail
```
breath detail, audible breaths, mouth noise, body shift,
lip sounds, throat catch, exhale between phrases,
breath before high note, swallow
```

### Instrumental Performance Detail
```
pick noise, fret squeak, finger movement noise, string vibration,
hammer-ons, slide noise, palm muting texture, bow scratch,
key click, pedal noise, reed buzz, valve click,
drumstick wood tone, brush texture on snare
```

### Recording Imperfections
```
imperfections kept, light mic handling noise, environmental bleed,
background noise floor consistent, slight distortion on peaks,
pre-roll silence, headphone bleed, amp hum, ground loop
```

### Analog Character
```
tape saturation, analog warmth, wow and flutter, tape hiss,
gentle preamp drive, tube warmth, transformer color,
vinyl surface noise, analog compression, tape speed variation
```

### Spatial
```
limited stereo, mono, narrow stereo image, short room reverb,
early reflections, dry center, wet sides, close and intimate,
no artificial reverb, natural decay
```

### Production Style
```
single-source path, authentic take, raw performance texture,
live-to-tape, minimal processing, no pitch correction,
no time alignment, minimal EQ, unmastered feel
```

## 3-Tier Stacking

Stack realism descriptors in the Style block's production slot (slot 10) or as a dedicated `recording:` line. More tiers = more physical, less polished.

### Tier 1: Minimal (3 descriptors)
For subtle warmth. Won't dramatically change the sound.
```
close mic, natural dynamics, room tone
```

### Tier 2: Moderate (6-8 descriptors)
Noticeable "recorded in a room" quality. Good default for acoustic genres.
```
close mic, natural dynamics, room tone, breath detail,
pick noise, short room reverb, limited stereo
```

### Tier 3: Maximum (12+ descriptors)
Full bedroom/live recording feel. Use when the user wants raw authenticity.
```
close mic, natural dynamics, room tone, breath detail, pick noise,
fret squeak, finger movement, tape saturation, analog warmth,
wow and flutter, limited stereo, short reverb, noise floor,
one-take, micro-rubato, imperfections kept
```

## How to Use in Style Blocks

**In the 11-slot formula:** Realism descriptors go in slot 8 (textures) and/or slot 10 (production). For Tier 2+, they may replace mood descriptors (slot 9) since the realism IS the mood.

**Lean instrumental (~170 chars with Tier 1):**
```
[Instrumental] Folk, fingerpicked acoustic guitar, upright bass warm, brushed drums, G Major, 84 BPM, close mic, room tone, clean mix, separated instruments [No Vocals]
```

**Fuller vocal (~320 chars with Tier 2):**
```
Folk singer-songwriter, fingerpicked acoustic guitar, upright bass, soft female alto whisper-to-belt, G Major, 84 BPM, close mic, natural dynamics, breath detail, pick noise, room tone, short room reverb, limited stereo, clean mix, separated instruments
```

## Anti-Patterns

- **"realistic"** — too vague, Suno ignores it
- **"authentic"** — same problem, means nothing specific
- **"natural sounding"** — Suno doesn't know what "natural" means
- **Tier 3 on electronic genres** — fights the genre, produces confused output
- **Mixing realism + heavy processing** — "tape saturation, pristine digital clarity" contradicts

## Synthesis Descriptors (for electronic genres)

When realism is counterproductive, use synthesis language instead:

```
FM synthesis, wavetable, granular texture, LFO-driven,
spectral morphing, bitcrushed, sidechain compression,
frequency modulation, vocoder processed, glitch artifacts,
sample-chopped, time-stretched, pitch-shifted
```
