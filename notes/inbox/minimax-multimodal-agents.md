---
title: MiniMax Multimodal Agents — Music, Art, Video Generation
created: 2026-03-25
tags: [idea, architecture, minimax, iris, veo, sol, image, video, music, generation]
status: in-progress
---

## Overview

Expand MiniMax integration from prompt-only to full generation pipeline. Three skills with dedicated souls handle music, image, and video — each generates media via API, downloads files, and builds indexed Obsidian-native folders with embedded playback.

## Skills

| Skill | Agent/Soul | Purpose | API |
|-------|------------|---------|-----|
| `/minimax-music` | Sol | Vocal-forward track generation | `generateMusic()` |
| `/minimax-art` | Iris | Album art / cover / visual generation | `generateImage()` |
| `/minimax-video` | Veo | Music video / visual narrative generation | `generateVideo()` |

**Forge (cancelled):** Programmatic generation is just a step in Sol's workflow, not a separate soul.

## Vault Structure

Consistent pattern across all three media types:

```
vault/studio/tracks/YYYY-MM-DD/HHMMSS-<name>/      ← music
  <name>.md                                          ← index + metadata + embeds
  media/
    <name>-001.mp3                                   ← generation 1
    <name>-002.mp3                                   ← reroll (same config)
    <name>-003.mp3                                   ← another attempt

vault/studio/image/YYYY-MM-DD/HHMMSS-<name>/        ← art
  <name>.md
  media/
    <name>-001.png
    <name>-002.png

vault/studio/video/YYYY-MM-DD/HHMMSS-<name>/        ← video
  <name>.md
  media/
    <name>-001.mp4
    <name>-002.mp4
```

### Index File Properties

**Music index:**
```yaml
---
id: track-YYYYMMDD-HHMMSS
title: "Track Name"
platform: minimax
model: music-2.5+
bpm: 78
key: Dm
genre: [celtic-folk, epic]
vocals: "deep baritone spoken word"
cost_per_gen: 0.15
generations: 3
---
```

**Image index:**
```yaml
---
id: image-YYYYMMDD-HHMMSS
title: "Art Name"
platform: minimax
model: image-01
aspect: "1:1"
cost_per_gen: 0.0035
generations: 2
source_track: "[[studio/tracks/...]]"
---
```

**Video index:**
```yaml
---
id: video-YYYYMMDD-HHMMSS
title: "Video Name"
platform: minimax
model: MiniMax-Hailuo-2.3
resolution: 1080P
duration: 6
cost_per_gen: 0.37
generations: 1
source_track: "[[studio/tracks/...]]"
---
```

### Generations Table

Every index includes a generations table tracking all attempts:

```markdown
## Generations

| # | File | Status | Notes |
|---|------|--------|-------|
| 001 | ![[media/<name>-001.mp3]] | kept | first gen, vocals too bright |
| 002 | ![[media/<name>-002.mp3]] | kept | better vocal tone |
| 003 | ![[media/<name>-003.mp3]] | discarded | drums overpowered vocals |
```

### Reroll Behavior

- "generate again" / "try another" / "reroll" → same config, next increment number
- "generate with changes: brighter vocals" → updated config (noted in table), still increments in same folder
- Each generation adds a row to the table and increments the `generations` count in frontmatter
- User marks status: `kept`, `discarded`, `favorite`
- Media files are never deleted — `discarded` just means "not the pick"

## Souls

### Iris — Visual Translator

**Backstory:** Album art designer who crossed into AI image generation. Spent years translating music into single images — the cover that makes someone click play before hearing a note.

**Philosophy:**
- The cover IS the first listen — it sets expectations before any sound plays
- Color palette should match the sonic palette — warm tones for warm tracks, desaturated for lo-fi
- Composition tells the song's story in one frame — find the single image that captures the emotional peak
- Less is more — the best covers are the ones you remember after seeing them for 2 seconds

**Capabilities:**
- Reads a track's Style block, mood, genre, lyrics concept
- Translates sonic descriptions into visual language (warm = amber tones, cavernous = stone textures)
- Knows aspect ratios: 1:1 (album art), 16:9 (banner/header), 9:16 (stories/vertical)
- Subject reference support for character consistency across an album
- Can generate standalone art or art linked to a specific track

### Veo — Motion Designer

**Backstory:** Music video director who moved from practical shoots to AI video. Thinks in cuts, slow motion, and the moment where the visual and the beat sync.

**Philosophy:**
- Video pacing follows the emotional arc — sparse visuals for quiet verses, dense for choruses
- First frame is the hook — if the opening image doesn't pull you in, the video fails
- Movement should match the rhythm — slow BPM = slow camera, fast BPM = quick cuts
- The best music video is the one where you can't imagine the song without the visuals

**Capabilities:**
- Reads a track's lyrics + Style block + emotional arc
- Designs scene-by-scene visual narrative matching the song structure
- First-frame / last-frame control for bookending
- Subject reference for character consistency
- Can chain with Iris: Iris generates the opening frame → Veo animates from it
- Resolution/duration control: 6s or 10s clips, 512P-1080P

## Implementation Plan

### Phase 1: Vault Structure + Sol Update
- Create `vault/studio/image/` and `vault/studio/video/` directories
- Update Sol's workflow to include API generation step
- Add reroll support to Sol's workflow
- Rename `/minimax` skill to `/minimax-music`

### Phase 2: Iris (Art)
- Write `.claude/souls/iris.md`
- Write `.claude/agents/minimax-art.md`
- Register `/minimax-art` skill
- Implement generation → download → index flow

### Phase 3: Veo (Video)
- Write `.claude/souls/veo.md`
- Write `.claude/agents/minimax-video.md`
- Register `/minimax-video` skill
- Implement generation → poll → download → index flow
- Iris→Veo chaining (first-frame from generated image)

### Phase 4: Cross-linking
- Track index links to its art: `cover: "[[studio/image/...]]"`
- Track index links to its video: `video: "[[studio/video/...]]"`
- Art/video index links back to source track
- Obsidian graph view shows the full creative chain

## Open Questions

- [ ] Should rerolls share the same prompt or allow minor tweaks per generation?
- [ ] Max generations per folder before suggesting a new concept?
- [ ] Should Iris auto-suggest art after a track is generated, or only on request?
- [ ] Should Veo auto-suggest video after art is generated (for the chaining flow)?
- [ ] Backup strategy for media files — gitignored like music card audio, iCloud syncs?
