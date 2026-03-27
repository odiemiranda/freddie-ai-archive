---
source: reflection
agent: freddie
task: "Deep audit and fix all agents, critics, and knowledge files for metatag reliability consistency"
workflow: research
outcome: success
scope: suno
confidence: high
tags: [suno, structure, tags, unreliable, platform-behavior]
session: 2026-03-26T16:50:15
---

## Observation
The `suno.md` file was updated to remove references to `[Fade Out]` and `[Outro]` because these structural tags were found to be unreliable in Suno generations.

## Learning
Specific structural tags like `[Fade Out]` and `[Outro]` are unreliable in Suno and should be avoided or replaced with more descriptive, non-structural prompting to ensure consistent outcomes.

## Evidence
suno.md lines 490/891 still referenced unreliable [Fade Out] and [Outro]
