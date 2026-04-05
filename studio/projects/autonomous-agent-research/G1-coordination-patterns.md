---
title: "G1: Coordination Pattern Assessment"
type: research-finding
project: autonomous-agent-research
project-code: AAR
goal: G1
status: complete
created: 2026-04-05
author: freddie
recommendation: keep-and-extend
---

# G1: Coordination Pattern Assessment

## Summary

Freddie's direct dispatch model (Freddie→Holmes/McCall/Ryan via Agent tool) is the right pattern for our scale. No migration needed. Extend with an optional Grader layer for autonomous mode.

## Pattern Comparison

| Pattern | Problem Solved | Cost | Freddie Fit |
|---------|---------------|------|-------------|
| **Direct dispatch** (current) | Separation of concerns, specialized agents | Low complexity, sequential latency | **Keep** — simple, proven |
| **Shared file cache** (SoulForge) | Parallel agents on same codebase | Very high complexity (graph DB, event bus) | **Skip** — we serialize via worktrees |
| **Commander→Brain→Grader** (IronClaude) | Trust in autonomous decisions | High latency (dual-model gate) | **Adopt Grader concept** for autonomous mode |
| **Blackboard** (shared state) | Unpredictable collaboration order | Very high complexity (state management) | **Skip** — too complex for single-operator |
| **Orchestrator-workers** (Anthropic) | Dynamic task decomposition | Moderate complexity | **Already doing this** — McCall→Ryan |

## Recommendation: Keep and Extend

### Keep
- Direct dispatch via Agent tool with subagent_type
- McCall as orchestrator, Ryan as worker (orchestrator-workers pattern)
- Worktree isolation for Ryan execution
- Phase-gated skills with user approval

### Extend (for autonomous mode)
- **Grader pattern**: When running in autonomous mode, add an independent review step before committing decisions. The Grader is a separate model call (Gemini or a second Claude) that evaluates McCall's architectural decisions against a checklist before Ryan executes.
- **Not a blocker**: The Grader is optional — only active when autonomy level > default. Human-gated mode doesn't need it because the user IS the grader.

### Skip
- SoulForge's graph DB and shared cache — solving for a scale we don't have
- Blackboard architecture — coordination complexity not justified
- Full Commander daemon — we don't need Slack-based async management

## Applicability to The Foundry
- The Grader pattern is directly portable to multi-agent Docker setups
- Orchestrator-workers pattern scales naturally to The Foundry's multi-tenant model
- Tag: **Foundry-applicable**
