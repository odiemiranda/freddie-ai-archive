---
agent: tala
type: observation
source: create-track run (Sunlit Shamisen Bossa, 2026-04-09)
importance: high
tags: [bgm, style-block, canonical-envelope, quality-gate]
---

# BGM Style Block: Canonical Envelope Checklist

## The learning

BGM Style blocks have a **canonical envelope** that is not optional. Missing any part of it is a Phase 3 quality failure that the user WILL catch during review. On the Sunlit Shamisen Bossa build on 2026-04-09, I shipped a BGM Style block missing three required envelope elements in a single run, and the user caught two of them in sequence before I proactively caught the third.

## The canonical envelope

```
[Instrumental] {genre}, {instruments}, {mood}, {Key}, {NN BPM}, {atmosphere}, {mix} [No Vocals]
```

Elements in order:
1. **`[Instrumental]` tag at the front** — not the word "instrumental" inline. The bracketed tag locks structural meta-tag behavior at the prompt level; the bare word is a weak fallback.
2. **Genre** — at the 2-genre cap max.
3. **Instruments** — at the 3-instrument cap max, with role/timbre hints.
4. **Mood descriptors** — if warranted; often elided when Key/BPM carry the feel.
5. **Key** — `{Letter} {Major/Minor}` format, e.g., `A Major`, `F Minor`. Paired with BPM.
6. **BPM** — format `NN BPM` (integer, space, capital BPM). Never `bpm`, never bracketed, never `bpm: 104`.
7. **Atmosphere** — scene/space descriptor, e.g., `sunlit cafe`, `moonlit temple`.
8. **`clean mix`** — soul mandatory.
9. **`[No Vocals]` tag at the end** — paired with the `[Instrumental]` opener. Belt-and-suspenders with Lyrics block tags, not a duplication.

## The Phase 3 quality gate

Before declaring Phase 3 done on ANY instrumental/BGM build, explicitly verify the envelope by walking these four slots against the Style block:

1. ✓ `[Instrumental]` tag at start?
2. ✓ `[No Vocals]` tag at end?
3. ✓ BPM token inline (`NN BPM`)?
4. ✓ Key token inline (`{Letter} Major/Minor`)?

Missing any one = Phase 3 is not done. Do NOT rely on charcount or validate-prompt to catch these — charcount's "slot 10 mandatory" warnings do catch "separated instruments" but not the envelope tags or BPM/Key tokens, and validate-prompt only flags structural issues.

## What NOT to interpret as this envelope

The envelope is specifically for BGM/instrumental tracks. For vocal tracks, the pattern differs:
- `[Instrumental]` tag is WRONG for vocal tracks.
- `[No Vocals]` is WRONG for vocal tracks.
- BPM and Key still apply, but mood descriptors typically carry more weight than scene/atmosphere.

## Evidence from the Sunlit Shamisen Bossa build

- **Phase 3 Style block I shipped**: `shamisen-led bossa nova jazz fusion instrumental, soft nylon bossa comp, brushed kit clave sway, sunlit cafe, clean mix` (119 chars)
- **Missing**: `[Instrumental]` tag, `[No Vocals]` tag, BPM token, Key token. Four of nine envelope slots missing on a single BGM build.
- **How it should have been from Phase 3**: `[Instrumental] shamisen-led bossa nova jazz fusion, nylon bossa comp, brushed kit clave, A Major, 104 BPM, sunlit cafe, clean mix [No Vocals]` (141 chars, BGM target band 140-170)

## Trigger

MiniMax critic in Phase 3 flagged "`[No Vocals]` unreliable without `instrumental` in Style." I responded by inserting the bare word `instrumental` inline as belt-and-suspenders — interpreting the flag at the cheapest possible level. The correct response was the full `[Instrumental]` tag at the envelope's front slot. When a critic says "add instrumental to Style," reach for the bracketed tag first, not the word.

## Preventing this in future Phase 3 runs

Add an explicit step at the end of Phase 3 generation (before returning to Freddie):

> **BGM Envelope Check** — if `type === "bgm"` or `"instrumental"`, walk the four envelope slots against the final Style block. If any are missing, fix them before reporting Phase 3 done, even if charcount and validate-prompt pass.

Reference files that document the envelope:
- `references/suno/niche-lofi.md`
- `references/suno/niche-japanese.md`
- `references/suno/niche-ambient.md`
- `references/suno/niche-relaxation.md`
- `references/suno/niche-sleep-healing.md`
- `references/suno/global-sounds.md`
- `references/suno/examples.md`
- `references/suno/moods.md`
