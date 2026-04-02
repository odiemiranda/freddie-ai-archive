---
source: reflection
agent: freddie
project: freddie-ai
projectPath: "D:\\PROJECTS\\freddie-ai"
task: "Fix project doc sync in brain.db — projects now included in --sync and --check auto-sync"
workflow: dev
outcome: success
scope: self
confidence: high
memoryType: semantic
tags: [brain.db, project_management, sync, entity_tracking, data_integrity, workflow_management, infrastructure]
session: 2026-04-02T00:01:49
---

## Observation
The `brain.db` project entity tracking system, specifically `syncProjects`, was not consistently updating project information because its execution was tied to the `--embed` flag, causing projects to be missed during regular `--sync` and `--check` operations.

## Learning
For robust and continuous project-aware context management, the `syncProjects` mechanism must be decoupled from the `--embed` flag and executed as part of all standard `brain.db` synchronization and checking workflows (`--sync` and `--check`). This ensures comprehensive and up-to-date project entity registration.

## Evidence
brain.db missed project BRIEF (not in sync path); syncProjects was behind --embed gate; promoted to run on every --sync and --check; verified end-to-end
