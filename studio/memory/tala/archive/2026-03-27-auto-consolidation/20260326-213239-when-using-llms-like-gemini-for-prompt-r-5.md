---
source: reflection
agent: tala
task: "Shamisen jazz fusion harp instrumental BGM - 3 variations"
workflow: suno-generation
outcome: success
scope: self
confidence: high
tags: [llm-behavior, prompt-refinement, instrumental, bgm, lyrics, agent-rules, gemini]
session: 2026-03-26T21:32:39
---

## Observation
Gemini's suggestions for instrumental BGM Lyrics consistently stripped out instrument direction, leaving only bare structural tags.

## Learning
When using LLMs (like Gemini) for prompt refinement, especially for instrumental BGM, be vigilant about their tendency to over-simplify Lyrics blocks by removing specific instrument direction. Agent rules requiring detailed direction must override such suggestions.

## Evidence
Gemini consistently wanted to strip Lyrics to bare structure tags - rejected per agent rules requiring instrument direction for BGM
