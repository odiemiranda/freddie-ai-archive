---
title: "Autonomous Agent Architecture Research"
type: brief
project: autonomous-agent-research
project-code: AAR
status: approved
created: 2026-04-05
author: holmes
notebook: 72e0f61c
---

# Autonomous Agent Architecture Research

## Problem

**Hypothesis:** We believe that systematically studying production autonomous agent architectures will produce a decision framework for Freddie AI's next evolution — specifically around coordination protocols, autonomy levels, quality gates, and self-improvement — because our current agent dispatch model was designed organically and we have no principled basis for deciding what to automate, what to gate, and what to isolate.

**Root cause (Five Whys):** Freddie's agent system works, but its design boundaries are instinct-based, not evidence-based. We chose fire-and-forget subagent dispatch because it was available. We chose phase-gated skills because the user wanted control. We chose tri-critic because three models felt robust. None of these decisions were informed by how production agent systems actually solve these problems. The result is that we cannot confidently answer: "Should agents coordinate via shared state or message passing? Should quality gates be independent or embedded? Should self-learning be autonomous or supervised?" Without studying what exists, every architecture decision is a guess dressed up as a design choice.

**Who feels this:** The user (as the system operator) — when workflows require excessive manual intervention for decisions that could be safely automated. And the system itself — when context is wasted, coordination is ad-hoc, and learning requires manual consolidation triggers.

## Goals

These are research outcomes. No code ships from this project.

- **G1: Pattern catalog.** Produce a structured comparison of agent coordination patterns (direct dispatch, message board, DAG, orchestrator-worker, blackboard) with documented tradeoffs and applicability criteria.
- **G2: Autonomy framework.** Define a decision matrix for classifying tasks by autonomy level (fully autonomous, human-gated, configurable) based on reversibility, blast radius, and confidence. Map Freddie's current workflows against it.
- **G3: Quality architecture assessment.** Evaluate whether an independent Grader agent (IronClaude model) would improve on Freddie's embedded tri-critic. Document evidence for and against.
- **G4: Self-learning evolution path.** Identify how Freddie's reflect-consolidate-distill pipeline compares to Phantom's self-evolution engine and Stanford generative agent memory. Produce a gap analysis with prioritized improvements.
- **G5: SDK evaluation.** Determine whether the Claude Agent SDK offers concrete advantages over Claude Code's native Agent tool dispatch for Freddie's use case. Produce a build-vs-stay recommendation with evidence.
- **G6: Cross-project architecture direction.** Assess how the new MCP external routes should evolve — particularly whether external sessions should use Agent SDK programmatic access, MCP tool passthrough, or a hybrid.
- **G7: Isolation model recommendation.** Evaluate VM isolation (Phantom), worktree isolation (current), and container isolation against Freddie's constraints. Produce a recommendation.

## Constraints

- **Pure research.** No code changes to freddie-ai during this project. Findings inform future implementation projects.
- **Single operator.** Freddie AI is a single-user system on Windows 11. Patterns designed for team-scale or cloud-native environments must be evaluated against this reality.
- **Claude Code dependency.** Freddie runs inside Claude Code. Any pattern that requires a different runtime (custom terminal harness, dedicated VM, Docker containers) must be assessed for compatibility or adaptation cost.
- **Existing investment.** brain.db, soul/agent/critic architecture, raw context capture, MCP server, and worktree isolation are shipping and working. Research should evaluate evolution of these, not replacement.
- **NotebookLM as primary research tool.** All deep analysis goes through NotebookLM (notebook 72e0f61c). Web search is for source discovery only.
- **Budget: time, not money.** The cost here is research hours, not compute. The user's time is the scarce resource — research must be structured to produce decisions, not just knowledge.

## Assumptions

- **A1:** The 10 NotebookLM sources provide sufficient breadth across the autonomous agent landscape. *Confidence: 75%.* Gap: no coverage of LangGraph, AutoGen, or MetaGPT architectures in the notebook (though brain.db has prior research on these from the agent-improvement-plan project).
- **A2:** Claude Agent SDK is stable enough to evaluate meaningfully. *Confidence: 65%.* Risk: the SDK is new and its API may shift. Evaluation should focus on architectural patterns it enables, not specific API calls.
- **A3:** Patterns from team-scale systems (IronClaude's Slack Commander, Phantom's VM hosting) can be meaningfully adapted to single-operator use. *Confidence: 60%.* This is the riskiest assumption — some patterns may simply not apply.
- **A4:** The user wants dual-mode: human-gated by default, autonomous opt-in when needed. *Confirmed.* Research should design for switchable autonomy, not push toward full autonomy.
- **A5:** Findings from this research will feed into The Foundry project (multi-agent Docker platform). *Confidence: 80%.* brain.db confirms 70-80% overlap between Shirozen and The Foundry vision. Dual-use findings are more valuable.

## Success Criteria

Research is "done" when all seven goals have a written deliverable:

- [ ] **SC1:** Pattern catalog document comparing 5+ coordination patterns with applicability matrix (which patterns fit which workflow types)
- [ ] **SC2:** Autonomy decision matrix with Freddie's current 10+ workflows mapped and classified
- [ ] **SC3:** Grader pattern assessment — adopt, adapt, or reject — with evidence chain
- [ ] **SC4:** Self-learning gap analysis with prioritized improvement list (effort vs impact)
- [ ] **SC5:** Agent SDK build-vs-stay recommendation with proof-of-concept scope defined (if "build")
- [ ] **SC6:** Cross-project MCP architecture direction document
- [ ] **SC7:** Isolation model recommendation with migration cost estimate (if changing)
- [ ] **SC8:** Master findings document that synthesizes all seven assessments into a prioritized roadmap of implementation projects

## Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Research scope creep — 10 sources generate unbounded investigation threads | High | Medium | Time-box each goal to 1 research session. If a thread opens, log it as a future question, don't chase it. |
| Analysis paralysis — too many patterns, no clear winner | Medium | High | Force a recommendation for every assessment. "We don't know yet" is not a valid conclusion — state what evidence would resolve it and move on. |
| Claude Agent SDK immaturity — evaluation may not reflect production readiness | Medium | Medium | Focus on architectural patterns the SDK enables, not API stability. Note version and date of evaluation. |
| Phantom/IronClaude over-indexing — impressive demos that don't transfer to our constraints | Medium | Medium | Every pattern must pass the "single operator on Windows" filter before inclusion. |
| Research becomes shelfware — findings never translate to implementation | Low | High | SC8 (prioritized roadmap) is a mandatory deliverable. Each finding must state: adopt, adapt, reject, or defer — with next action. |
| Prior research staleness — agent-improvement-plan findings (2026-04-02) may conflict with new sources | Low | Low | Cross-reference explicitly. Note where new evidence confirms or contradicts prior findings. |

## Key Questions to Investigate

### Coordination & Dispatch (G1)
1. What are the concrete tradeoffs between direct dispatch (Freddie's current model) and message-board coordination (AgentHub model)?
2. Does SoulForge's shared file cache pattern solve a problem Freddie actually has, or is it optimizing for a scale we don't operate at?
3. How does the orchestrator-worker pattern (Anthropic's guide, Azure patterns) compare to Freddie's McCall-dispatches-Ryan model?

### Autonomy & Control (G2)
4. What is the right classification for each of Freddie's workflows on the autonomy spectrum? Which ones could safely run without user approval?
5. How does pi-mono's "user steering queue" compare to Freddie's phase-gated skills? Is there a middle ground?
6. What would a configurable autonomy system look like — where the user sets per-task autonomy levels?

### Quality & Grading (G3)
7. Does IronClaude's independent Grader agent produce measurably better outcomes than embedded critics? What evidence exists?
8. Would a Grader pattern complement or replace Freddie's tri-critic? Could the Grader be the reconciliation layer?
9. What is the cost (latency, tokens, complexity) of adding an independent Grader versus the current inline critic approach?

### Self-Learning & Memory (G4)
10. How does Phantom's 3-tier episodic memory compare to Freddie's raw/inbox/knowledge/archive structure?
11. What specific mechanisms make Phantom's self-evolution engine work — and which of those mechanisms could operate within Claude Code?
12. Stanford generative agents used reflection + importance scoring. How does Freddie's confidence decay compare?

### SDK & Platform (G5, G6)
13. What does the Claude Agent SDK enable that Claude Code's native Agent tool does not? (Concrete capabilities, not marketing.)
14. For cross-project MCP routes: should external sessions dispatch agents via SDK, or continue using MCP tool passthrough?
15. Does the Agent SDK's session/hook model map to Freddie's skill/workflow structure?

### Isolation (G7)
16. What specific problems does VM isolation (Phantom) solve that worktree isolation (Freddie) does not?
17. Is there a lightweight isolation model between worktrees and full VMs that fits the single-operator Windows constraint?
18. How does isolation interact with brain.db access — can isolated agents still read/write the shared knowledge store?

## Research Sources

### Primary (NotebookLM Notebook 72e0f61c)

| # | Source | Key Contribution |
|---|--------|-----------------|
| 1 | Anthropic "Building Effective Agents" | 6 canonical patterns: prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer, autonomous agents |
| 2 | Claude Agent SDK | Programmatic agent loop, built-in tools, subagents, hooks, MCP, sessions |
| 3 | 15 Agentic AI Design Patterns | ReAct, planning, reflection, multi-agent, tree-of-thoughts, semantic memory, self-improvement, ensemble, dry-run, graph memory, mental model, blackboard, meta-controller, reflexive agent |
| 4 | SoulForge | SQLite dependency graph (Soul Map), parallel multi-agent dispatch, surgical code reads, shared file cache |
| 5 | IronClaude | Hook-enforced brainstorm-plan-execute pipeline, Slack Commander daemon, independent Grader agent |
| 6 | Phantom | VM isolation, self-evolution engine with LLM judge panel, 3-tier episodic memory, dynamic MCP tool creation |
| 7 | pi-mono coding-agent | Minimalist terminal harness, no sub-agents, user steering queue, session trees with fork/rewind |
| 8 | Azure Agent Design Patterns | Sequential, concurrent, group chat, handoffs, orchestrator-worker |
| 9 | Redis AI Agent Architecture | Production agent architecture guide |
| 10 | Nader's Agent SDK Guide | Complete tutorial on building with Claude Agent SDK |

### Secondary (brain.db — Prior Research)

| Source | Date | Key Finding |
|--------|------|-------------|
| AgentHub deep dive | 2026-04-02 | Message board coordination, DAG exploration, platform/culture separation |
| AI Agent Persona Systems | 2026-04-02 | CrewAI backstory, MetaGPT SOPs, AutoGen conversation patterns, LangGraph implicit identity |
| Gemini Architecture Review | pre-2026-04-05 | Multi-agent architecture evaluation of freddie-ai |
| Cross-Session Architecture | pre-2026-04-05 | Worktree execution, MCP silent operations, Bifrost multi-provider routing |

## Out of Scope

- **Implementation.** This project produces documents, not code. Implementation projects will be scoped separately from the findings.
- **The Foundry.** While findings will inform The Foundry, that project has its own scope (Shirozen, multi-agent Docker). We note applicability but don't design for it.
- **Model evaluation.** We are not comparing Claude vs Gemini vs GPT for agent tasks. The question is architectural patterns, not model capabilities.
- **Cost optimization.** Token costs, API pricing, and compute budgets are not in scope. If a pattern is architecturally sound but expensive, we note the cost concern and move on.
- **Non-Claude platforms.** We study patterns from any source, but recommendations must be implementable within Claude Code or via its extension points (MCP, hooks, Agent tool, Agent SDK).

## Resolved Questions

1. **Autonomy preference:** Human-gating is the default mode, with the ability to switch to autonomous when needed. The research should design a dual-mode system (gated default + autonomous opt-in), not push toward full autonomy. Assumption A4 updated accordingly.

2. **Research depth:** Top 3 for deep investigation: G1 (coordination patterns), G4 (self-learning), G5 (SDK evaluation). Remaining goals (G2, G3, G6, G7) get survey-level treatment with "investigate further" tags. Can be promoted later.

3. **Foundry overlap:** Primary lens is Freddie. This research is a proof-of-concept toward The Foundry — findings should be portable but not designed for multi-tenant/Docker from the start. Tag Foundry-applicable findings as secondary.

4. **Agent SDK timing:** Evaluate now. Focus on architectural patterns and capability gaps (what Agent SDK enables vs what Agent tool provides), not API stability. Version-pin the evaluation. The goal is understanding leverage, not committing to migration.
