---
name: Agent Architecture and Workflow
description: Guidelines for designing agent workflows, managing context in iterative skills, establishing internal agent hierarchies, standardizing agent initialization, optimizing documentation for context, and employing effective development methodologies.
type: knowledge
agent: freddie
tags: [architecture, context_management, orchestrator_pattern, subagents, skill_design, workflow_management, delegation, infrastructure, initialization, boot_flow, agent_soul, documentation_management, context_optimization, development_methodology, parallel_execution, worktrees, agent_hierarchy, review_process, dev_session]
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
- **Formalized Development Workflow:** The Freddie→McCall→Wick→McCall chain, including iterative McCall→Wick→McCall review loops, is a proven and formalized development workflow for complex tasks, effectively leveraging the internal agent hierarchy for structured iteration and refinement. This workflow is particularly effective for managing and reviewing focused, contained changes, such as updates to core infrastructure components like the reflection engine.

## Agent Initialization
- **Standardized Boot Flow:** A dedicated skill (e.g., `/morning`) invoking an `init.ts` hook can effectively manage agent initialization.
- **Core Functionality:** This includes loading the agent's core personality/context ('soul') and providing initial user briefings upon session boot.

## Documentation for Context Optimization
- **Strategy:** Extract architectural rules into separate files and remove content derivable from other sources from core documentation (e.g., `CLAUDE.md`).
- **Benefit:** This significantly reduces context window consumption, improves clarity, and maintains a lean primary reference for the agent.

## Development Methodology for Complex Tasks
- **Approach:** For complex, multi-component development tasks, utilize parallel execution across multiple worktrees.
- **Benefit:** This technique accelerates development, manages concurrent changes effectively, and streamlines integration processes.

**Why:** These architectural patterns prevent main agent context overflow, thereby improving efficiency and reliability. Clear documentation, including specific standards for multi-agent workflows and general context optimization, enhances clarity, debuggability, and maintainability. A structured internal hierarchy and formalized development workflows enable effective delegation, structured iteration, and management of complex tasks. Standardized initialization ensures a consistent starting state and user interaction. Employing parallel worktrees for complex development further accelerates project delivery and improves change management. The formalized McCall→Wick→McCall workflow's proven effectiveness for contained infrastructure changes further validates its utility in maintaining core system integrity.

**How to apply:**
- Identify skills that involve iterative processes, variation loops, or complex multi-turn interactions and implement the orchestrator pattern with explicit subagent dispatch.
- Adhere to specified documentation standards across `SKILL.md`, `workflow.md`, and Mermaid diagrams for multi-agent workflows.
- Design internal workflows using a structured agent hierarchy with specialized roles and clear interaction protocols, such as the Ryan→McCall→Wick chain, and leverage the formalized Freddie→McCall→Wick→McCall development workflow for complex tasks, especially for focused infrastructure changes.
- Implement an `init.ts` hook, invoked by a dedicated skill, to standardize agent boot flow, including loading the agent's 'soul' and displaying initial briefings.
- Actively refactor core documentation by extracting architectural rules and removing derivable content to reduce context window consumption.
- For complex, multi-component development, leverage parallel execution across multiple worktrees to manage tasks and accelerate progress.

*Consolidated from: knowledge_orchestrator_architecture.md, 20260330-123330-a-structured-internal-agent-hierarchy-wi-1.md, 20260330-211929-a-dedicated-skill-and-an-init-ts-hook-ca-1.md, 20260401-020156-extracting-architectural-rules-and-remov-1.md, 20260401-020156-for-complex-multi-component-development--2.md, 20260401-080142-the-freddie-mccall-wick-mccall-chain-inc-1.md, 20260401-081439-the-formalized-mccall-wick-mccall-develo-2.md*
