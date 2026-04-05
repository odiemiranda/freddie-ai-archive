---
title: "G5: Claude Agent SDK Evaluation"
type: research-finding
project: autonomous-agent-research
project-code: AAR
goal: G5
status: complete
created: 2026-04-05
author: freddie
recommendation: evaluate-poc
---

# G5: Claude Agent SDK Evaluation

## Summary

The Agent SDK provides capabilities we can't get from Claude Code's Agent tool: standalone processes, true parallel execution, programmatic hooks, persistent sessions, and HTTP service wrapping. Recommend a proof-of-concept before committing to migration.

## Capability Gap Analysis

| Capability | Agent Tool (current) | Agent SDK | Impact |
|-----------|---------------------|-----------|--------|
| Run outside Claude Code | No — runs inside session | Yes — standalone scripts | **High** — enables scheduled agents, CI/CD |
| True parallel agents | Sequential (worktrees) | Native async (Promise.all) | **High** — parallel McCall+Ryan |
| Programmatic hooks | File-based (.claude/hooks/) | Code-level callbacks | **Medium** — more flexible, same concept |
| Persistent sessions | Per-conversation only | Cross-invocation resume | **High** — multi-session workflows |
| HTTP service agents | Not possible | Wrap in Express/FastAPI | **High** — agents as microservices |
| Subagent model | String-based subagent_type | Full AgentDefinition objects | **Medium** — richer agent definitions |
| Skill/CLAUDE.md loading | Native | Via settingSources: ['project'] | **Same** — compatible |
| MCP integration | Native | Native | **Same** — compatible |

## What the SDK Enables That We Can't Do Today

1. **Scheduled agents**: McCall reviewing PRs on a cron, Holmes monitoring a project inbox
2. **Agent-as-service**: Expose McCall as an HTTP endpoint that external CI can call
3. **True parallelism**: Run Holmes (brief) + McCall (architecture) simultaneously without worktree serialization
4. **Cross-session memory**: Resume a dev session days later with full context preserved programmatically
5. **The Foundry runtime**: Multi-agent Docker containers, each running an Agent SDK instance

## What We'd Lose or Need to Rebuild

1. Claude Code's native tool permissions UI (would need custom permission handler)
2. Automatic CLAUDE.md/rules loading (need explicit settingSources config)
3. Interactive user experience in terminal (SDK is programmatic, not interactive)
4. Current hook files would need conversion to programmatic callbacks

## Recommendation: Proof-of-Concept

**Don't migrate. Build one POC to test leverage.**

Proposed POC: Wrap McCall as an Agent SDK service for automated PR review.
- Input: GitHub PR URL
- Process: McCall loads soul, queries brain.db, dispatches council, reconciles
- Output: Structured review posted as PR comment
- Trigger: GitHub webhook or cron schedule

This tests: standalone execution, programmatic hooks, MCP integration (brain.db), and HTTP service wrapping — all the high-impact gaps.

**If POC succeeds**: Evaluate migration path for scheduled/autonomous workflows while keeping Claude Code for interactive sessions.
**If POC fails**: Stay on current model, revisit when SDK matures.

## Applicability to The Foundry
- Agent SDK is THE runtime for The Foundry — each container runs an SDK instance
- POC directly validates the Foundry's core architecture
- Tag: **Foundry-critical**
