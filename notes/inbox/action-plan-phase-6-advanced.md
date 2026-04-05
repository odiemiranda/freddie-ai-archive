---
title: Phase 6 — Advanced Improvements
created: 2026-03-28 02:24:20 AM
tags: [action-plan, phase-6, improvements, polish]
parent: "[[notes/inbox/action-plan-validation-and-improvements]]"
status: pending
---

# Phase 6: Advanced Improvements

> Polish and optimize based on everything learned in Phases 1-5.

## Prerequisites
- Phases 1-5 complete — system validated, learning loop proven

## Tasks

### 6.1 — FTS5 ranking tuning
Boost title field weight in brain.db queries. Test with 10 common queries, measure precision improvement.

### 6.2 — Negative learning loop
When an agent queries decisions and finds a past rejection:
- Surface it as a warning: "Last time you used [X] with [Y], user rejected it because [Z]"
- Agent can override with justification, but must acknowledge the warning
- Implementation: new field in decisions table `outcome_type: success|rejected|partial`

### 6.3 — Multi-model reflection validation
After Gemini extracts learnings, send them to Grok for validation:
- "Are these learnings specific and actionable, or generic noise?"
- Filter out low-quality learnings before they hit inbox
- Cost: ~$0.01 extra per reflection

### 6.4 — Split workflow.md for subagents
Create separate files for subagent-scoped nodes:
- `workflow-planning.md` — nodes for main conversation
- `workflow-build.md` — nodes for subagent
Subagent only loads the build file, reducing context pollution.

### 6.5 — /usage skill
Manual command for session-level usage summary:
- Total tokens by model (Gemini, MiniMax, Grok, Claude)
- Cost breakdown
- Calls count
- Session duration
- Comparison with average session

### 6.6 — Windows terminal popup fix
Investigate why SessionStart/Stop hooks flash a console window on Windows. Possible fixes:
- `start /B` prefix
- PowerShell `-WindowStyle Hidden`
- Node child_process with `windowsHide: true`

## Definition of done
- [ ] Discovery accuracy measurably improved
- [ ] Past rejections surface as warnings
- [ ] Reflection quality improves with multi-model validation
- [ ] Subagent context is leaner
- [ ] /usage command works
- [ ] No terminal popup on Windows
