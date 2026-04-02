# AI Agent Persona Design in Production Systems

## 1. Generative Agents and Persona Research

### Stanford "Generative Agents" (Park et al., 2023)

The foundational paper is *"Generative Agents: Interactive Simulacra of Human Behavior"* by Joon Sung Park et al. (Stanford/Google). It introduced 25 agents living in a sandbox environment, each with believable behavior emerging from three architectural components:

- **Memory stream**: A timestamped log of all observations and actions, stored as natural language records with recency/importance/relevance scoring.
- **Reflection**: Periodic synthesis of memories into higher-level abstractions. Triggered by an importance accumulator threshold.
- **Planning**: Agents generate day-level plans, decompose into hour-level actions, and re-plan reactively when unexpected events occur.

Persona is seeded with a one-paragraph description, but *identity emerges from the memory/reflection loop* rather than from the seed alone. The seed is shallow; depth comes from accumulated experience.

**Key Insight for Coding Agents:** The Stanford work proves that a thin persona seed + structured memory + reflection produces more believable behavior than an elaborate static prompt. This directly validates the Freddie architecture pattern: soul files (seed) + inbox/knowledge (memory) + reflect.ts (reflection) + distill-soul.ts (synthesis).

## 2. Persona Engineering Techniques

### Shallow vs. Deep Persona

| Layer | Example | Effect |
|-------|---------|--------|
| **Name + title** | "You are Alex, a senior engineer" | Minimal behavior change; model defaults dominate |
| **Experience + opinions** | "You've shipped 3 distributed systems. You distrust ORMs." | Measurably shifts code style, review tone, and architectural recommendations |
| **Instincts + failure memory** | "You once lost a production database to an unguarded migration." | Creates consistent behavioral heuristics that persist across tasks |

Key findings from prompt engineering research:

- **Expertise encoding works best through *specific opinions*, not credentials.** "Senior engineer" changes little. "You believe premature abstraction is worse than duplication" changes decisions.
- **Negative experiences are more behaviorally sticky than positive ones.** An agent told "you were burned by X" will actively avoid X more reliably than one told "you prefer Y."
- **Consistency requires constraints, not just description.** Telling an agent its communication style produces more uniform output than describing personality traits.

### Senior vs. Junior Engineer Simulation

The most effective technique is encoding *decision heuristics* rather than skill level:

- **Senior**: "When facing ambiguity, you make a decision and document the tradeoff. You never block on missing requirements — you state your assumption and proceed."
- **Junior**: "When uncertain, you ask clarifying questions before proceeding. You follow established patterns exactly."

## 3. Multi-Agent Frameworks

### CrewAI

Agents defined with four fields — the most explicit persona model:

```python
Agent(role, goal, backstory, tools)
```

Backstory quality is the primary driver of agent output quality.

### MetaGPT

Assigns software-engineering roles with SOPs. Persona defined by *process constraints* — what the agent must produce and in what order.

### AutoGen (Microsoft)

Lightweight — persona is entirely in the system prompt. Multi-agent coordination through conversation patterns.

### LangGraph

Agent identity is implicit in the graph structure. No first-class persona abstraction.

## Summary: What Works

| Technique | Evidence | Freddie Analog |
|-----------|----------|----------------|
| Thin seed + emergent memory | Stanford Generative Agents | Soul files + memory tiers |
| Opinions over credentials | Prompt engineering research | Agent files encode specific rules |
| Behavioral constraints | CrewAI backstory, MetaGPT SOPs | `.claude/agents/` define WHAT |
| Negative experience anchors | Community consensus | Decision log with confidence decay |
| Reflection loop | Stanford, self-learning | reflect.ts + consolidate + distill-soul |

**Convergent finding: persona depth comes from structured memory and behavioral constraints, not from elaborate character descriptions.**
