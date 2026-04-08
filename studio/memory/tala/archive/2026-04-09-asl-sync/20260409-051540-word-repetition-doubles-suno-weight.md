---
source: reflection
agent: tala
task: "Created variant of The Last Iron — Unbroken Roots cinematic Celtic neofolk dirge"
workflow: create-track-variant
outcome: Gen 5 sounded structurally correct (gruff/low/slow/dramatic) but monotone, like reading a book. Diagnosed root cause as duplicate use of `chant` in the Style block. Three word swaps in gen 6 fixed it without disturbing anything else.
scope: self
confidence: medium
tags: [prompt-structure, suno-vocal, word-repetition, monotone-vs-melodic, surgical-fixes]
session: 2026-04-09T05:15:40
---

## Observation
The gen 5 Style block used the word `chant` twice: `sustained low chant phrases` and `group bass chant harmonies low and unified`. The result was a successful cinematic dirge with the right voice and pacing, but completely monotone delivery — sounded like reading a book aloud, no melodic movement.

The fix was three surgical word swaps:
- `sustained low chant phrases` → `sustained low melodic phrases`
- `group bass chant harmonies low and unified` → `group bass harmonies low and lyrical`
- `ancient ritual lament` → `ancient melodic ritual lament`

After the swaps, gen 6 had melodic vocal movement (Irish keening / Gaelic lament feel) without losing any of the gruff/low/slow/dramatic profile.

## Learning
**Word repetition in a Suno Style block doubles that word's interpretive weight.** Using `chant` twice forced Suno to lock into the strongest meaning of `chant` — Gregorian-style monotone — and override other tonal cues that would normally have allowed melodic movement.

Practical rules:
1. **Don't repeat content words across elements.** If a word appears twice in the Style block, it doubles its pull on Suno's interpretation.
2. **If you need similar concepts in two elements, balance with synonyms or counter-cues.** Instead of `chant phrases` + `chant harmonies`, try `melodic phrases` + `harmonies low and lyrical` — both still suggest the chant aesthetic, but neither word is "chant" so the monotone trap is avoided.
3. **`chant` specifically pulls toward monotone in Suno.** It's safe to use ONCE if balanced with melodic cues; using it twice locks the delivery flat. Counter-cues that work alongside `chant`: `melodic phrases`, `lyrical`, `lament`, `melodic ritual`.
4. **Surgical word swaps are the right fix when the structure is working but a sonic detail is off.** Don't rebuild a working prompt to fix one symptom. Find the specific word(s) producing the unwanted pull and swap minimal terms.

## Evidence
- Gen 5 (monotone): used `chant` twice, sounded like Gregorian recitation
- Gen 6 (melodic): three word swaps removed both `chant` instances, replaced with `melodic`/`lyrical`/`melodic ritual lament`. Same gruff/low/slow profile, now with melodic movement.
- The fact that adding `melodic` once to `ancient ritual lament` (a phrase that already conceptually implies melody — laments are melodic by tradition) noticeably helped suggests Suno needs explicit reinforcement even of concepts that should be implicit. Implicit concepts are not enough when there are competing strong cues elsewhere.
