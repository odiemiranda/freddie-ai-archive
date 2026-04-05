---
title: Phase 1 — Orchestrator Pattern Validation
created: 2026-03-28 02:24:20 AM
tags: [action-plan, phase-1, orchestrator, validation]
parent: "[[notes/inbox/action-plan-validation-and-improvements]]"
status: pending
---

# Phase 1: Orchestrator Pattern Validation

> Prove that subagent dispatches work for all 3 variation-loop skills.

## Context

The orchestrator pattern was implemented on 2026-03-28 (commit 7974cbe). It separates:
- **Planning (inline):** Nodes that involve user interaction, confirmations, lightweight processing
- **Variation build (subagent):** Heavy ref loading, critique loops, track name generation
- **Review (inline):** User approval, disk writes, API calls

This has never been tested with a real subagent dispatch.

## What to test

### 1.1 — /suno full generation

**Run:** `/suno [any genre request]`

**Watch for:**
- Does the main conversation correctly dispatch `subagent_type: "tala"` at node 012?
- Does the subagent prompt include all required context (variation lean, locked ingredients, discovery output, music card constraints)?
- Does the subagent return complete variation content (Track Names 18, Style, Exclude, Settings, Lyrics, Notes)?
- Does the main conversation correctly present the returned variation at node 020?
- Does the user review cycle work (approve → write to disk)?
- Does "next variation" correctly dispatch a new subagent for V2, V3?

**Potential friction:**
- Subagent may not have access to MCP tools (uses CLI only — per feedback memory)
- Subagent may not find the right reference files if discovery output isn't passed clearly
- Track names tool requires 3 API calls (MiniMax + Gemini + Grok) — may be slow in subagent
- charcount validation needs to work inside the subagent

### 1.2 — /lyrics full generation

**Run:** `/lyrics [Style block or direction]`

**Watch for:**
- Subagent dispatched with `subagent_type: "rune"` at node 007
- Tri-critic loop (3 API calls: Gemini, MiniMax, Grok) runs inside subagent
- Reconciliation table built correctly
- Complete variation returned to main conversation

**Potential friction:**
- Tri-critic is the heaviest part — 3 sequential API calls inside one subagent
- Reconciliation table format needs to survive the subagent→main handoff

### 1.3 — /minimax-music full generation

**Run:** `/minimax-music [any track request]`

**Watch for:**
- Subagent dispatched with `subagent_type: "sol"` at node 005
- Style block and Lyrics returned to main conversation
- Main conversation handles file creation (node 009) and API generation (node 010)
- API generation ($0.15/track) only happens in main conversation, never in subagent

**Potential friction:**
- Method selection (NL vs NL-6S vs JSON) needs to be decided in subagent and returned clearly
- Main conversation needs enough context from subagent to write correct files

### 1.4 — Adjust path test

**Run:** Complete a variation in /suno, then say "adjust" with specific changes.

**Watch for:**
- New subagent dispatched with original prompt + adjustment instructions
- Subagent rebuilds from node 013 (not from scratch)
- Returned variation reflects the requested changes
- No duplication or loss of context between original and adjusted version

### 1.5 — Document friction

After 1.1–1.4, create a friction log:

| Issue | Skill | Severity | Fix |
|-------|-------|----------|-----|
| ... | ... | ... | ... |

### 1.6 — Fix issues

Address all high/medium severity issues before moving to Phase 2.

## Definition of done

- [ ] All 3 skills produce complete output through orchestrator pattern
- [ ] Adjust path works for at least 1 skill
- [ ] No data loss between subagent and main conversation
- [ ] Friction log created and high-severity issues fixed
