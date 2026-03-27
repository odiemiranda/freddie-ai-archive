---
name: style-block-validation
description: Style block character budgets, 8-step validation gate, element count limit, anti-conflict rules, production keyword vocabulary.
type: knowledge
agent: tala
tags: [suno, style-block, validation, char-budget, production-keywords, anti-conflict, front-loading]
---

# Style Block Validation

## 1. Control Panel, Not Sentences
Write comma-separated keyword tags, not prose. Never write "a warm harp playing gentle chords" — write "harp low-register pad chords, no melody".

## 2. Character Budgets

| Track Type | Target | Hard Max |
|---|---|---|
| Instrumental/BGM | ~200-250 chars | 350 |
| Vocal (simple) | ~250-350 chars | 450 |
| Vocal (complex/fusion) | ~350-450 chars | 500 |

## 3. Front-Loading (Tag Dominance)
First tag = ~60% influence, second = ~25%, third = ~10%, fourth = ~5%. Genre and lead instrument MUST be the first two elements. Everything after has sharply diminishing returns. Optimal: 3-5 tags total.

## 4. Slot Priority (Cut Order)
When over budget, cut in this order:
1. Slot 6 — Chord progression (LOW)
2. Slot 8 — Textures (LOW)
3. Slot 9 — Mood, keep 1 (MEDIUM)
4. Slot 4 — Simplify constraint (HIGH)

Never cut: Genre (2), Lead instrument (3), Key/BPM (7), Production keywords (10).

## 5. Total Element Count: 3-5 Optimal, Max 7
Count every distinct element (genre, each instrument, each mood, each texture, BPM, key, each production keyword). 3-5 tags is the sweet spot. Tags like `[Instrumental]`/`[No Vocals]` don't count. Over 7 = over-stuffed.

Audio quality degrades with length: 500-1500 chars = 9.5/10 clarity, 3500-5000 chars = 8.0/10. Less = cleaner.

## 6. Anti-Conflict Rule
No contradictory tags ("calm aggressive", "lo-fi polished"). Either cut one OR reframe as complementary pair ("uplifting yet peaceful").

Common conflicts:
- Energy: calm vs aggressive, driving vs laid-back
- Texture: lo-fi vs polished, crisp vs fuzzy
- Space: intimate vs vast, dry vs reverb-heavy

## 7. BPM Is Critical
Without BPM, Suno guesses tempo → hallucinations and sync issues. Always specify exact BPM.

## 8. The 11-Step Validation Gate
After writing Style + Lyrics blocks:

**Style block (steps 1-8):**
1. **Char count** — within target/hard max?
2. **Slot order** — 11 slots in correct sequence?
3. **Slot bleed** — texture/groove terms in instrument slots?
4. **Front-load** — genre + lead instrument as first two elements (60% + 25% weight)?
5. **Instrument count** — max 3 (lead + support + rhythm)?
6. **Keyword check** — "clean mix" + "separated instruments" present?
7. **Conflict check** — contradictory tags?
8. **Element count** — 3-5 optimal, max 7?

**Lyrics block (steps 9-11):**
9. **Tag reliability** — any unreliable tags? Replace: `[Intro]`→`[Short Instrumental Intro]`, `[Outro]`→`[Big Finish]`, `[Break]`→`[melodic interlude]`
10. **Section count** — max 5-7 structural tags total
11. **Mood tags** — max 1-2 `[Mood: X]` per song

## 9. Production Keyword Vocabulary

| Category | Keywords | When |
|---|---|---|
| Polish | polished, modern pop polish, glossy | Radio-ready pop/R&B |
| Clarity | Hi-Fi, pristine digital clarity, crisp | High-definition, anti-lo-fi |
| Frequency | surgical EQ, bright treble EQ, punchy | Tight mix, prevent muddy drums |
| Space | stereo wideners, wide spatial imaging | Expansive soundstage |
| Dryness | minimal FX and reverb, dry | Intimate, no echo |
| Vintage | analog warmth, tape saturation, vinyl crackle | Lo-fi, retro, cozy |

Default: "clean mix, separated instruments" unless user requests specific character.
