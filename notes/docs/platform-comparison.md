# Platform Comparison: Suno AI vs MiniMax Music

Quick guide for choosing the right platform and translating prompts between them.

---

## When to Use Which

| Goal | Use | Why |
|------|-----|-----|
| **Instrumental/background music** | Suno | MiniMax doesn't have stable instrumental-only mode yet |
| **Vocal-forward songs** | MiniMax | Superior vocal timbre control, physics-level fidelity |
| **Lo-fi / study beats** | Suno | Battle-tested for this niche, `[Instrumental]` + `[No Vocals]` works reliably |
| **Pop / R&B with singing** | MiniMax | Better vocal quality, deeper timbre control |
| **Ambient / soundscapes** | Suno | Instrumental mode + nature textures work well |
| **Detailed vocal production** | MiniMax | Spatial, dynamics, stereo, timbre all controllable |
| **Quick iteration** | Suno | Natural-language prompts, Weirdness/Style Influence sliders |
| **Maximum prompt detail** | MiniMax | 2000-char style limit gives room for nuance |
| **Near-instrumental with hums/chants** | MiniMax | Use vocal hum (ooh, aah, mmm, la) for instrumental feel |
| **Genre fusion experiments** | Suno | More forgiving with unusual genre combos |

---

## Platform Specs Side by Side

| Feature | Suno AI | MiniMax Music |
|---------|---------|---------------|
| **Style field limit** | 1,000 characters | 2,000 characters |
| **Lyrics limit** | ~3,000 characters | 3,500 characters |
| **Instrumental mode** | Yes (`[Instrumental]` + `[No Vocals]`) | Not stable (use vocal hum workaround) |
| **Structure tags** | ~15 types | 14 types |
| **Prompt style** | Tagged or natural-language | Natural-language, 6-Section Analysis, or JSON |
| **Advanced settings** | Weirdness (0-100%), Style Influence (0-100%) | None (model handles creativity) |
| **Vocal control depth** | Basic (style + lyrics tags) | Deep (timbre, spatial, dynamics, stereo) |
| **Extend/continue** | Yes (Extend button) | Not available |
| **Strength** | Versatile, great instrumental | Vocal music excellence |

---

## Translating Prompts: Suno → MiniMax

### Style Field

**Suno prompt (tagged):**
```
[Instrumental] Lo-fi hip hop, warm Rhodes piano, soft boom-bap drums, vinyl crackle, C Minor, 82 BPM, nostalgic and peaceful [No Vocals]
```

**MiniMax translation:**
```
Lo-fi hip hop with warm Rhodes piano, soft boom-bap drums, vinyl crackle texture, C Minor, 82 BPM. Nostalgic and peaceful atmosphere, warm analog production. Female vocal hum (ooh, aah) — breathy, intimate, with light reverb. Soft and mellow, like a cozy study session on a rainy day.
```

### Key Changes When Converting Suno → MiniMax

1. **Remove `[Instrumental]` and `[No Vocals]`** — not supported in MiniMax
2. **Add vocal description** — MiniMax needs vocals. Use hum/chant for near-instrumental
3. **Expand descriptions** — you have 2x the character budget, use it
4. **Add vocal timbre details** — "breathy female hum, intimate, light reverb"
5. **Remove Suno-specific tags** — `[Sustained]`, `[Background Music]` don't apply

### Lyrics Field

**Suno instrumental lyrics:**
```
[Intro]
[Vinyl crackle fades in]
[Rhodes piano, gentle]

[Verse]
[Soft drums enter]
[Melody drifts, melancholic]
```

**MiniMax translation:**
```
[intro]
Ooh... mmm...
(vinyl crackle, gentle Rhodes)

[verse]
Aah... ooh...
La la la... mmm...
(soft drums, drifting melody)
```

**Key difference:** MiniMax needs actual syllables for the vocalist. Use ooh/aah/mmm/la to create near-instrumental texture.

---

## Translating Prompts: MiniMax → Suno

### Style Field

**MiniMax prompt:**
```
Indie pop with warm female vocals, breathy and intimate delivery with light reverb. Acoustic fingerpicked guitar, soft piano, and subtle strings. Nostalgic, bittersweet atmosphere like looking through old photographs. 95 BPM, G Major.
```

**Suno translation (tagged):**
```
Indie pop, warm breathy female vocals, intimate delivery, fingerpicked acoustic guitar, soft piano, subtle strings, G Major, 95 BPM, nostalgic bittersweet atmosphere, light reverb
```

### Key Changes When Converting MiniMax → Suno

1. **Compress to 1000 characters** — cut descriptive prose, keep keywords
2. **Add `[Instrumental]` + `[No Vocals]`** if converting to instrumental
3. **Add Weirdness/Style Influence** — choose values based on genre (see SKILL.md)
4. **Convert lowercase tags to bracketed** — `[intro]` → `[Intro]`
5. **Remove vocal timbre details** if going instrumental

---

## Structure Tag Mapping

| Suno | MiniMax | Notes |
|------|---------|-------|
| `[Intro]` | `[intro]` | Same concept |
| `[Verse]` | `[verse]` | Same concept |
| `[Pre-Chorus]` | `[pre-chorus]` | Same concept |
| `[Chorus]` | `[chorus]` | Same concept |
| `[Bridge]` | `[bridge]` | Same concept |
| `[Outro]` | `[outro]` | Same concept |
| `[Hook]` | `[hook]` | Suno = instrumental melody; MiniMax = vocal hook |
| `[Build]` | `[build-up]` | Different name, same concept |
| `[Drop]` | `[drop]` | Same concept |
| `[Break]` | `[break]` | Same concept |
| `[Solo]` | `[solo]` | Same concept |
| `[Instrumental]` | `[instrumental]` | Suno = no vocals; MiniMax = instrumental section |
| `[Instrumental Break]` | `[interlude]` | Different name |
| `[Fade Out]` | — | No MiniMax equivalent |
| `[Sustained]` | — | No MiniMax equivalent |
| `[End]` | — | No MiniMax equivalent |
| — | `[breakdown]` | MiniMax has this; Suno uses `[Break]` |
