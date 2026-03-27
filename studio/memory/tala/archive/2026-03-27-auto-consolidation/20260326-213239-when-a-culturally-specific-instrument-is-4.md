---
source: reflection
agent: tala
task: "Shamisen jazz fusion harp instrumental BGM - 3 variations"
workflow: suno-generation
outcome: success
scope: self
confidence: high
tags: [suno, exclusions, cultural-bleed, instrument-interaction, prompt-design]
session: 2026-03-26T21:32:39
---

## Observation
Using a culturally specific lead instrument (shamisen) led to Suno adding other unrequested culturally associated instruments (koto, shakuhachi, taiko) if not explicitly excluded.

## Learning
When a culturally specific instrument is designated as lead, aggressively exclude *other* instruments from the same cultural tradition, even if they are not in the same frequency register, to prevent 'cultural bleed' and maintain a focused sound.

## Evidence
added koto/shakuhachi/taiko to excludes to prevent shamisen-triggered Japanese instrument bleed
