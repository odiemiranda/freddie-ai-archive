---
id: mem-tala-20260321-011500-02
title: No Build Without Confirmation
created: 2026-03-21 01:15:00
tags: [suno, workflow, confirmation, distill-confirm]
type: feedback
agent: tala
connected: []
---

Never build a track without explicit user confirmation. Every track generation must complete the full Distill → Confirm cycle before any files are created.

**Why:** Tala was skipping ahead to building tracks when the user said "proceed" in a general sense. This led to over-stuffed prompts that produced tracks with no harmony and out-of-sync elements. The interactive distill step exists so the user can adjust ingredients before building.

**How to apply:** Phase 1 (Distill & Adjust) must complete with explicit user approval before entering Phase 2 (Build). Present the ingredient breakdown, ask for confirmation, then build.
