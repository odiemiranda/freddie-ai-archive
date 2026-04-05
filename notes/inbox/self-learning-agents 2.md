---
title: Self-Learning Agents — Autonomous Reflection & Soul Evolution
created: 2026-03-25
tags: [idea, architecture, self-learning, agents, souls, memory, reflection]
status: implemented
inspired_by: [hermes-agent (NousResearch), forge-agent (prototype)]
---

## Overview

Agents learn from every workflow, consolidate knowledge automatically, and evolve their own soul files over time. The system is fully autonomous except for soul commits, which require user approval.

## Core Metaphor

- **Soul** = the brain. Core identity, proven techniques, fundamental beliefs. Authoritative — if it's in the soul, the agent lives by it.
- **Knowledge** = the notebook. Verified learnings the agent consults. External, searchable, can be wrong or outdated.
- **Inbox** = scratch paper. Raw observations from a single workflow. Unverified, may contradict existing knowledge.

## The Learning Lifecycle

```
Task completes
     │
     ▼
┌─────────────────────────┐
│  Agent Reflection (auto) │  ← internal deliberation after every workflow
│  "What surprised me?     │
│   What failed?           │
│   What technique worked  │
│   that wasn't in my      │
│   knowledge?"            │
└────────┬────────────────┘
         ▼
   inbox/{agent}/           ← raw observation (auto)
         │
         ▼
┌─────────────────────────┐
│  Consolidation (auto)    │  ← reconcile inbox + knowledge
│  Deduplicate, resolve    │
│  contradictions, archive │
│  stale entries           │
└────────┬────────────────┘
         ▼
   knowledge/{agent}/       ← verified, current (auto)
         │
         ▼
┌─────────────────────────┐
│  Soul Distillation       │  ← propose diff to soul file
│  (auto-propose,          │
│   user-approve)          │
│                          │
│  Criteria:               │
│  - Proven (repeated,     │
│    never contradicted)   │
│  - Fundamental (changes  │
│    approach, not a tip)  │
└────────┬────────────────┘
         ▼
   .claude/souls/{agent}.md ← core identity (user reviews)
```

## Three Layers

### Layer 1: Post-Task Reflection

**Trigger:** End of every creative workflow (Tala, Rune, Sol, Echo)
**How:** One additional LLM call after the workflow completes
**What it asks:**
- What surprised me about this generation?
- What technique worked that wasn't already in my knowledge?
- What failed or produced unexpected results?
- What would I do differently next time?
- Is this learning specific to me, or should other agents know?

**Output:** A learning file in `memory/{agent}/inbox/` with:
```markdown
---
source: reflection
task: "lo-fi shamisen track with vinyl texture"
session: 2026-03-25T20:15:00
scope: self | shared | {specific-agent}
confidence: high | medium | low
---

## Observation
{what happened}

## Learning
{the generalizable takeaway}

## Evidence
{specific generation details that support this}
```

**Scope tagging:**
- `self` — only relevant to this agent (e.g., Tala's Suno-specific trick)
- `shared` — cross-cutting (e.g., instrument behavior all agents should know)
- `{agent-name}` — relevant to a specific other agent (e.g., Tala discovers something Rune needs)

Shared-scope learnings are copied to `memory/shared/inbox/`.

### Layer 2: Knowledge Consolidation

**Trigger:** Automatic when inbox has 3+ items, or on-demand via `bun tools/consolidate-memory.ts --agent {name}`
**How:** Gemini analyzes inbox items against existing knowledge files
**What it does:**
- Merges compatible observations into existing knowledge entries
- Creates new knowledge entries for novel learnings
- Resolves contradictions (newer evidence wins, flags for review if ambiguous)
- Archives processed inbox items with date-stamped reason
- Cross-agent propagation: shared-scope items are consolidated into `memory/shared/knowledge/`

**Existing tool:** `tools/consolidate-memory.ts` already does most of this. Needs:
- Auto-trigger when inbox threshold is met
- Scope-aware routing (shared items → shared knowledge)
- Cross-agent notification ("Tala learned X, flagging for Rune")

### Layer 3: Soul Distillation

**Trigger:** On-demand via `bun tools/distill-soul.ts --agent {name}`, or suggested when knowledge accumulates
**How:** LLM reads knowledge files + current soul file, proposes changes
**Criteria for soul-worthy learnings:**
- **Proven:** Appeared in 2+ separate workflows, never contradicted
- **Fundamental:** Changes how the agent approaches work (not a one-off tip)
- **Durable:** Won't become stale when platforms update (unless it's a platform truth)

**Output:** A proposed diff to the soul file, shown to the user:
```
Soul distillation for Tala:

+ ## Instrument Density Rule
+ When combining shamisen with lo-fi, always specify "sparse phrasing"
+ in the Style block. Without it, Suno defaults to rapid picking which
+ overwhelms the lo-fi warmth.
+
+ Evidence: 4 generations (2026-03-20 through 2026-03-25), consistent
+ across different BPM ranges. Contradicts earlier assumption that
+ "gentle" was sufficient — "gentle" produces inconsistent results.

Apply to .claude/souls/tala.md? [approve / adjust / reject]
```

**What gets committed to the soul:**
- The learning itself (concise, actionable)
- NOT the evidence trail (that stays in knowledge/)
- NOT timestamps or session details (soul is timeless)

**What stays in knowledge:**
- Evidence, examples, edge cases
- Learnings that are useful but not fundamental
- Platform-specific details that may change

## Cross-Agent Learning

### How it flows

```
Tala reflects → tags learning as scope:shared
     │
     ▼
memory/shared/inbox/        ← lands here
     │
     ▼
consolidate-memory.ts       ← merges into shared knowledge
     │
     ▼
memory/shared/knowledge/    ← all agents can read
     │
     ▼
distill-soul.ts             ← can propose changes to any relevant soul
```

### Agent-specific routing

When Tala tags a learning as `scope: rune`, it goes to:
- `memory/shared/inbox/` (for consolidation)
- AND flagged in consolidation output: "Tala→Rune: {learning}"

Rune's next workflow picks this up from shared knowledge.

## Implementation Plan

### Phase 1: Reflection Engine — DONE (2026-03-25)
- ~~`libs/reflect.ts` — shared reflection library~~ done
- ~~Reflection prompt template (what to ask, how to format)~~ done — Gemini-powered, scope/confidence tagging
- ~~Integration points in agent workflows (post-generation hook)~~ done — `tools/reflect.ts` CLI wrapper
- ~~Inbox file writer with proper frontmatter~~ done — structured YAML frontmatter + observation/learning/evidence sections

### Phase 2: Auto-Consolidation — DONE (2026-03-25)
- ~~Extend `tools/consolidate-memory.ts` with:~~ done
  - ~~Inbox threshold auto-trigger~~ done — `--auto` flag, default threshold 3
  - ~~Scope-aware routing~~ done — shared-scope learnings written to `memory/shared/inbox/`
  - ~~Cross-agent propagation~~ done — agent-targeted scope writes to target agent's inbox
- Shared knowledge directory structure — ready (created on first write)

### Phase 3: Soul Distillation — DONE (2026-03-25)
- ~~`tools/distill-soul.ts` — new tool~~ done
- ~~Soul file parser (read current soul, identify sections)~~ done
- ~~Knowledge-to-soul proposal generator~~ done — Gemini analyzes proven/fundamental criteria
- ~~Interactive approval flow (show diff, accept/adjust/reject)~~ done — dry-run default, `--apply` to commit
- ~~Soul file writer (apply approved changes)~~ done

### Phase 4: Integration & Polish — PARTIAL (2026-03-25)
- ~~Agent workflow hooks (auto-reflect on completion)~~ done — Tala (step 10), Sol (step 8), Rune (step 10), Echo (post-task section)
- Dashboard: `bun tools/learning.ts --status` — show inbox/knowledge/soul state per agent (TODO)
- Consolidation scheduling (or hook into existing workflows) (TODO)

## Key Design Decisions

- **Soul files are sacred.** Only user-approved changes. No auto-commits to souls.
- **Knowledge is disposable.** Can be rebuilt from inbox history if needed.
- **Reflection is cheap.** One LLM call per workflow — don't over-engineer.
- **Cross-agent learning is opt-in.** Agents tag scope explicitly. No surprise knowledge injection.
- **Gemini for consolidation, not Claude.** Keep Claude for orchestration. Gemini is free for analysis work.

## Open Questions

- [x] Should reflection run even on failed/rejected generations? → Yes — outcome param accepts success/partial/failed/rejected, all trigger reflection
- [ ] How often should distill-soul be suggested? After every consolidation, or on a cadence?
- [ ] Should souls have a changelog section tracking when/why they evolved?
- [ ] Can we use the forge-agent SQLite+FTS5 approach alongside markdown? (Faster search, same data)
- [x] Should Echo (Gemini) be the reflector for all agents, or should each agent self-reflect? → Gemini reflects on behalf of all agents (via libs/reflect.ts). Each agent builds its own trace, Gemini analyzes it. Echo skips reflection when called as sub-agent
