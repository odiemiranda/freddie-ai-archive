---
source: reflection
agent: tala
task: "morning shamisen jazz — passed after pipeline fixes"
workflow: suno-generation
outcome: success
scope: self
confidence: high
tags: [suno, generation, reroll, quality, pipeline]
session: 2026-03-26T03:15:00
---

## Observation
After applying the 11-slot formula, dual production critics, aggressive excludes, and BGM rules, the Morning Shamisen Jazz V1 (The Kettle Knows) passed Suno testing. Still has minor quirks per generation, but the layering chaos is gone and the prompt is clean enough to find good results within a few rerolls.

## Learning
A good Suno prompt doesn't guarantee a perfect generation — it guarantees that MOST generations will be in the right ballpark. The job of the prompt is to narrow the possibility space so that 3-5 rerolls will include at least one keeper. If every generation is chaotic, the prompt is broken. If most generations are close but 1-2 have quirks, the prompt is working and you just need to reroll.

This means the quality gate isn't "does this prompt produce a perfect track on first try" — it's "does this prompt produce tracks where the worst generation is still recognizably what was requested."

## Evidence
- V1 "The Kettle Knows" — first generation had minor quirks but was clearly a morning jazz shamisen track with clean harp separation. Previous versions (before 11-slot formula) produced unrecognizable chaos.
- The reroll expectation is normal for Suno — even professional prompts need 3-5 generations to find the best one.
