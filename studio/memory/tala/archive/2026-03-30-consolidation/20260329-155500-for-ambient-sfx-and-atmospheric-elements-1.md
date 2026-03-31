---
source: reflection
agent: tala
task: "Built /create-music-card skill with phase-gated workflow, genre/mood context flags, test protocol for card validation experiment. Discovered Suno Style block ignores ambient SFX (rain) — must go in Lyrics as staging brackets. Planned NotebookLM research agent integration."
workflow: create-music-card
outcome: success
scope: shared
confidence: high
tags: [suno, style-block, lyrics, sfx, atmosphere, prompt-construction, platform-behavior]
session: 2026-03-29T15:55:00
---

## Observation
During testing, it was discovered that placing ambient SFX (e.g., 'rain') in the Suno Style block results in them being ignored by the model.

## Learning
For ambient SFX and atmospheric elements to be effectively processed by Suno, they must be placed within the Lyrics section using specific typed brackets (e.g., `[Effect: Rain]`, `[Instrument: Rain]`, or `[Mood: Rainy]`). The Style block should be reserved for global genre and core instrumentation.

## Evidence
Discovered Suno Style block ignores ambient SFX (rain) — must go in Lyrics as staging brackets. Test finding: Style=instruments/genre, Lyrics=staging/atmosphere/SFX.
