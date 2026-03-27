---
source: reflection
agent: freddie
task: "Implement orchestrator pattern for variation-loop skills"
workflow: suno-generation
outcome: success
scope: self
confidence: high
tags: [architecture, context_management, orchestrator_pattern, subagents, skill_design]
session: 2026-03-28T01:38:29
---

## Observation
User identified context window consumption as a problem with inline skill execution for variation-loop skills. An orchestrator pattern was implemented as a solution.

## Learning
For skills involving iterative processes, variation loops, or complex multi-turn interactions that risk context window overflow, an orchestrator pattern with explicit subagent dispatch is an effective solution to manage context and prevent the main agent's context from being consumed by repetitive internal loops.

## Evidence
User identified context window consumption as a problem with inline skill execution. Brainstormed 5 strategies. User chose Strategy 2 (Orchestrator pattern). Implemented across 3 skills: suno, lyrics, minimax-music (all identified as having variation loops).
