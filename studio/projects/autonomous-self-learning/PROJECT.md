---
id: project-autonomous-self-learning
title: Autonomous Self-Learning (ASL) — Phase 2 Productionization
project-code: ASL
status: open
created: 2026-04-07
tags: [project, daemon, pipeline, infrastructure]
---

## Summary

Phase 2 of the self-learning pipeline. AAR Phase 1 built the research prototype — distill + gates + judges + staged proposals + auto-apply. It works but is synchronous, blocks sessions for 8+ minutes, has a corrupted judge signal from MiniMax parser failures, and cannot be safely exposed via external MCP.

Phase 2 makes it production-grade: decouple the slow path (distill + gates + judges) from user sessions via a daemon + task queue, track agent state explicitly in brain.db, fix the judge panel, introduce local LLMs for cheap classification, and gate external MCP deployment behind a readiness checklist.

## Relationship to AAR Phase 1

AAR Phase 1 shipped as 7 task docs (AAR-0001 through AAR-0007) plus the post-shipment fixes in AAR-0008, AAR-0008B, and AAR-0009. Those are complete. ASL Phase 2 does NOT modify the Phase 1 pipeline logic — it restructures how and when that logic runs.

## Notes

- Triggered by experiential pain points surfaced during the AAR-0008 bug fix session on 2026-04-07
- BRIEF.md captures the full design conversation
- Task numbering: ASL-0001 onwards
- Folder: `vault/studio/projects/autonomous-self-learning/`
- Source code convention: `src/services/{service-name}/` for daemons (distinct from `src/servers/` which is MCP-only)
