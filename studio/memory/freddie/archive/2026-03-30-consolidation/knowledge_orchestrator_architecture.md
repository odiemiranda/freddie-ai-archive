---
name: Orchestrator Pattern for Iterative Skills
description: Best practices for implementing and documenting the orchestrator pattern to manage context in iterative or variation-loop skills.
type: knowledge
agent: freddie
tags: [architecture, context_management, orchestrator_pattern, subagents, skill_design, documentation, workflow_management, optimization, skill_categorization]
---

## Problem
Context window consumption is a significant issue for skills involving iterative processes, variation loops, or complex multi-turn interactions, leading to inefficiencies and potential failures.

## Solution
The orchestrator pattern with explicit subagent dispatch is an effective architectural solution to manage context and prevent the main agent's context from being consumed by repetitive internal loops.

## Application Scope
- **Beneficial for:** Skills characterized by iterative processes or variation loops where context accumulation is a significant risk (e.g., `suno`, `lyrics`, `minimax-music`).
- **Not required for:** Skills with a linear, single-pass execution flow that do not have variation loops (e.g., `musiccard`, `gemini`, `minimax-art`, `minimax-video`). These should maintain a simpler, non-orchestrated structure.

## Documentation Standards
When adopting an orchestrator pattern, establish clear documentation standards:
- **`SKILL.md`:** Add an 'Execution Model' section detailing phases and subagent dispatch instructions.
- **`workflow.md`:** Use `★SUBAGENT/★MAIN` markers in the Table of Contents and execution context banners on boundary nodes.
- **Mermaid Diagrams:** Update with separate subgraphs to visually represent subagent interactions versus the main conversation flow.

**Why:** This pattern prevents main agent context overflow, thereby improving efficiency and reliability. Clear documentation enhances clarity, debuggability, and maintainability of complex, multi-agent workflows.

**How to apply:**
1.  Identify skills that involve iterative processes, variation loops, or complex multi-turn interactions.
2.  Implement the orchestrator pattern for these identified skills, ensuring explicit subagent dispatch.
3.  Adhere to the specified documentation standards across `SKILL.md`, `workflow.md`, and Mermaid diagrams to clearly outline the multi-agent workflow.

*Consolidated from: knowledge_orchestrator_pattern.md*
