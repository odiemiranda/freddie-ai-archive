---
name: Agent Architecture and Workflow
description: Guidelines for designing agent workflows, managing context in iterative skills, establishing internal agent hierarchies, and standardizing agent initialization.
type: knowledge
agent: freddie
tags: [architecture, context_management, orchestrator_pattern, subagents, skill_design, workflow_management, delegation, infrastructure, initialization, boot_flow, agent_soul]
---

## Orchestrator Pattern for Iterative Skills
- **Problem:** Context window consumption is a significant issue for skills involving iterative processes, variation loops, or complex multi-turn interactions.
- **Solution:** The orchestrator pattern with explicit subagent dispatch effectively manages context and prevents the main agent's context from being consumed by repetitive internal loops.
- **Application Scope:**
    - **Beneficial for:** Skills characterized by iterative processes or variation loops where context accumulation is a significant risk (e.g., `suno`, `lyrics`, `minimax-music`).
    - **Not required for:** Skills with a linear, single-pass execution flow that do not have variation loops (e.g., `musiccard`, `gemini`, `minimax-art`, `minimax-video`). These should maintain a simpler, non-orchestrated structure.
- **Documentation Standards:** When adopting an orchestrator pattern, establish clear documentation standards:
    - **`SKILL.md`:** Add an 'Execution Model' section detailing phases and subagent dispatch instructions.
    - **`workflow.md`:** Use `★SUBAGENT/★MAIN` markers in the Table of Contents and execution context banners on boundary nodes.
    - **Mermaid Diagrams:** Update with separate subgraphs to visually represent subagent interactions versus the main conversation flow.

## Structured Internal Agent Hierarchy
- **Effective Approach:** A structured internal agent hierarchy with specialized roles and defined interaction patterns is effective for managing complex internal workflows and delegation.
- **Example Hierarchy (Ryan, McCall, Wick):**
    - **Ryan (PM/Analyst):** Jack Ryan-inspired analyst, structured assessments, risk/reward options, scope guard, confidence levels.
    - **McCall (Architect/Planner):** Equalizer-inspired architect, diagrams-first communication, council dispatch pattern, task slicing for Wick, decision documentation.
    - **Wick (Dev/Executor):** John Wick-inspired executor, task-doc-as-contract, escalate-don't-guess (doc gets updated before continuing), worktree isolation, terse reporting.
- **Key Interaction Protocol:** A chain like Ryan→McCall→Wick, where McCall updates the task document when Wick encounters an issue, ensures clear delegation and issue resolution.

## Agent Initialization
- **Standardized Boot Flow:** A dedicated skill (e.g., `/morning`) invoking an `init.ts` hook can effectively manage agent initialization.
- **Core Functionality:** This includes loading the agent's core personality/context ('soul') and providing initial user briefings upon session boot.

**Why:** These architectural patterns prevent main agent context overflow, thereby improving efficiency and reliability. Clear documentation enhances clarity, debuggability, and maintainability of complex, multi-agent workflows. A structured internal hierarchy enables effective delegation and management of complex tasks. Standardized initialization ensures a consistent starting state and user interaction.

**How to apply:**
- Identify skills that involve iterative processes, variation loops, or complex multi-turn interactions and implement the orchestrator pattern with explicit subagent dispatch.
- Adhere to specified documentation standards across `SKILL.md`, `workflow.md`, and Mermaid diagrams for multi-agent workflows.
- Design internal workflows using a structured agent hierarchy with specialized roles and clear interaction protocols, such as the Ryan→McCall→Wick chain.
- Implement an `init.ts` hook, invoked by a dedicated skill, to standardize agent boot flow, including loading the agent's 'soul' and displaying initial briefings.

*Consolidated from: knowledge_orchestrator_architecture.md, 20260330-123330-a-structured-internal-agent-hierarchy-wi-1.md, 20260330-211929-a-dedicated-skill-and-an-init-ts-hook-ca-1.md*
