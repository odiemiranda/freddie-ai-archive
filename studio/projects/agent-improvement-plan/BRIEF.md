---
id: project-agent-improvement
title: "McCall & Wick Agent Improvement Plan"
status: brief
created: 2026-04-02
tags: [project, agents, souls, architecture, development]
---

# McCall & Wick Agent Improvement Plan

## The Problem

McCall (architect) and Wick (developer) are functional but lack depth. Their souls define personality traits and work patterns, but they don't carry the weight of real-world engineering experience. McCall designs systems but doesn't think like a principal architect who's seen production failures at scale. Wick executes task docs but doesn't bring the instincts of a senior engineer who's shipped real systems.

### Gaps

1. **McCall lacks architectural depth.** Knows to draw diagrams and document decisions, but doesn't carry patterns like CQRS, event sourcing, hexagonal architecture, or domain-driven design as second nature. Doesn't reason about scalability, observability, or operational concerns instinctively.

2. **Wick lacks engineering maturity.** Executes task docs well but doesn't bring senior-level instincts: defensive coding, performance awareness, edge case detection, debugging intuition, or the ability to spot design smells in the code he's writing.

3. **Document templates are ad-hoc.** PRDs, BRIEFs, ARCHITECT docs, and task specs are written fresh each time. No structured templates that ensure consistency and completeness.

4. **Personality is thin.** Both agents have character references (McCall from The Equalizer, Wick from John Wick) but the souls don't carry enough personal backstory, professional experience, or thinking depth to feel like real people with real opinions.

## Goals

1. **McCall becomes a principal-level system architect** — carries deep knowledge of architecture patterns, design trade-offs, operational concerns, and can reason about systems at application and infrastructure level.

2. **Wick becomes a senior software engineer** — brings defensive coding instincts, performance awareness, debugging intuition, and the experience of someone who's shipped and maintained production systems.

3. **Structured document templates** — PRD, BRIEF, ARCHITECTURE, SPEC, TASK templates that ensure completeness and consistency across projects.

4. **Rich, believable personalities** — souls that feel like real professionals with backstories, opinions, pet peeves, and instincts born from experience.

## Research Areas

- Architecture and design patterns (DDD, hexagonal, event sourcing, CQRS, microservices, etc.)
- Real-world system design experience (what principal architects actually think about)
- Senior software engineering practices (defensive coding, performance, debugging)
- Agent persona design (how to make AI agents feel like real professionals)
- Document templates (PRD, BRIEF, ARCHITECTURE, SPEC) from open-source repos and industry
- GitHub repos with agent/persona systems
- Reddit/forum discussions on AI agent personalities
- YouTube tutorials on system design and architecture

## Constraints

- Changes are to soul files (`.claude/souls/`) and agent files (`.claude/agents/`) only
- Must maintain compatibility with existing workflows (skills, dispatch patterns)
- Personality should enhance, not replace, existing character foundations
- Templates go in a shared location accessible to all agents

## Success Criteria

1. McCall's architecture docs show deeper reasoning — trade-off analysis, pattern selection, operational concerns
2. Wick's implementations show senior instincts — edge cases caught, performance considered, defensive patterns used
3. Document templates are consistent and reusable across projects
4. Both agents feel like distinct, believable professionals with depth
