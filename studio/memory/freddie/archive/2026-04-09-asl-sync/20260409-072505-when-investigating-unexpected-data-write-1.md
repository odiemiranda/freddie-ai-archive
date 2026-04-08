---
source: reflection
agent: freddie
project: freddie-ai
projectPath: "D:\\PROJECTS\\freddie-ai"
task: "Cleaned tala test pollution from soul_changes (1292 phantom rows + stray approved file). Fixed morning briefing spam — capped per-agent applied entries at 2 with rollup. Created Unbroken Roots variant of The Last Iron — 6 generations to land. Discovered descriptive-vs-prescriptive lesson for Suno prompts."
workflow: session
outcome: success
scope: self
confidence: high
memoryType: semantic
importance: 7
tags: [FTS5, database, bug_detection, data_integrity, platform_behavior]
session: 2026-04-09T07:25:05
---

## Observation
A perceived 'duplicate-write bug' in `soul_changes` was identified and verified to be an FTS5 trigger cascade, not a genuine data integrity issue.

## Learning
When investigating unexpected data writes or perceived duplication in `brain.db` related to full-text search, consider FTS5 trigger cascades as a potential cause rather than immediately assuming a data integrity bug.

## Evidence
ASL cleanup — deleted test pollution, verified duplicate-write 'bug' was FTS5 trigger cascade not real.
