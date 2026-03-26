---
source: reflection
agent: tala
task: "morning shamisen jazz — systematic Style block failures"
workflow: suno-generation
outcome: partial
scope: self
confidence: high
tags: [style-block, formula, suno, slots, separated-instruments, constraints]
session: 2026-03-26T02:30:00
---

## Observation
Repeated failures in Morning Stillness track traced to freeform Style block composition. Each iteration fixed one symptom but missed others — culture-as-genre, missing `separated instruments`, unconstrained harp, too many textures, wrong slot ordering. A working reference (First Sip 2026-03-25) followed an exact slot order with specific critical keywords that the new track improvised away from.

## Learning
Style blocks must follow a strict 11-slot formula. Suno front-loads importance — genre first, instruments second, production last. The slot order is not a suggestion.

Critical discoveries:
1. **`separated instruments` is the single most important production keyword.** Without it, Suno layers instruments on top of each other. This one keyword is the difference between a clean mix and chaos.
2. **Support instruments need explicit CONSTRAINTS.** "harp warm harmonic bed" = Suno gives harp melodic freedom = layering chaos. "harp low-register pad chords as background warmth" = register locked, role locked, prominence locked.
3. **`rolled-off highs`** keeps support instruments in their lane — prevents harp/pad from climbing into lead instrument's register.
4. **Genre must be a known Suno genre**, not a culture name. "Jazz fusion" works. "Japanese folk fusion" floods the output with unwanted traditional instruments.
5. **Max 2 textures, max 2 mood words.** Every additional descriptor is another variable Suno tries to satisfy.

## Evidence
- 5 iterations of Morning Stillness, each fixing one problem but missing others
- Working reference: `tracks/2026-03-25/161416-morning-window-groove/first-sip.md` — follows the 11-slot formula exactly
- A/B comparison showed `separated instruments` + `rolled-off highs` + support instrument constraints as the critical three
