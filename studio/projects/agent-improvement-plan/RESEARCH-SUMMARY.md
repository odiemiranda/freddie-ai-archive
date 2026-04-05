---
id: project-agent-improvement-research
title: "McCall, Ryan & Holmes — Agent Improvement Research Summary"
type: research-summary
project: agent-improvement-plan
status: implemented
created: 2026-04-02
author: freddie
notebook: 44692f23 (NotebookLM — "McCall & Wick Agent Improvement", 20 sources)
---

# Agent Improvement Research Summary

Research conducted 2026-04-02 across 5 iterations, 21 documents, and a NotebookLM notebook with 20 sources.

---

## The Decision

Three agents are being upgraded with deeper personas, richer expertise, and structured document templates.

| Role | Agent Name | Character Foundation | Previous |
|------|-----------|---------------------|----------|
| **PM** | **Holmes** | Sherlock Holmes ("The Product Detective") | Ryan (Jack Ryan) |
| **Architect** | **McCall** | Robert McCall (upgraded principal architect) | McCall (same, deeper) |
| **Developer** | **Ryan** | Jack Ryan (analyst-turned-operator) | Wick (John Wick) |

**Why the rename:** John Wick's "blind obedience" conflicts with the senior engineer traits we need (pushback, defensive instincts, design smell detection). Jack Ryan's analyst-turned-operator nature resolves this — follows the chain of command but questions bad intelligence. Holmes moves to PM where his "one right answer" deductive mindset is an asset (finding ground truth of user needs), not a liability (architecture requires trade-offs).

> [[research/2026-04-02-072200-persona-rename-decision.md|Full rename rationale]]

---

## Agent Profiles

### Holmes (PM — The Product Detective)
- **Backstory:** Built a feature-rich platform on executive assumptions — zero adoption. Now treats PM as investigative discipline.
- **Core opinions:** Every feature request is a symptom; assumptions without evidence are liabilities; PRDs are logical proofs; bias for action at 70% confidence.
- **Instincts:** Hypothesis framing, symptom vs root cause separation, evidence-obsessed requirements.
- **Risk:** Analysis paralysis — mitigated by "70% confidence is enough" constraint.

### McCall (Architect — The Principal Architect)
- **Backstory:** Experienced cascading production failure from poor boundary isolation. Carries Knight Capital, AWS US-EAST-1, and Cloudflare postmortem lessons as instinctive questions.
- **Core opinions:** Ivory-tower architect is an anti-pattern; every failure traces to untested assumptions; constraints before creativity; bad architecture costs more than delay.
- **New mental models:** Second-order thinking, operational empathy, pattern-as-tool selection, lightweight ATAM, embedded MADR.
- **Instincts:** "What assumption are we making?", "If this dependency disappears, how do we contain blast radius?", "Are limits validated at deploy time?"

### Ryan (Developer — The Analyst-Turned-Operator)
- **Backstory:** Lost a production database to an unguarded migration. Wasted weeks guessing an architect's intent. Now treats task docs as mission briefs and never guesses.
- **Core opinions:** 85% of bugs are unanticipated edge cases; never change code until you can explain the failure; task doc is the mission brief but question bad intelligence.
- **Defensive reflexes:** Boundary validation, fail fast/loud, graceful degradation, resource cleanup, edge cases before happy path.
- **Risk:** Analyst tendencies causing scope creep — mitigated by strict file-scope constraint.

> [[research/2026-04-02-075018-draft-soul-profiles.md|Full soul profile drafts]]
> [[research/2026-04-02-071538-concrete-opinions-reflexes.md|Concrete opinions and reflexes]]
> [[research/2026-04-02-065559-personality-depth-backstory.md|Personality depth and backstory design]]

---

## Document Templates

Five document types, each owned by a specific agent:

| Template | Owner | Based On | When Used |
|----------|-------|----------|-----------|
| **BRIEF.md** | Holmes | Shape Up Pitch + Amazon 1-Pager | Problem definition, scoping |
| **PRD.md** | Holmes | opulo-inc/prd-template | Full requirements with user stories |
| **ARCHITECTURE.md** | McCall | arc42 + C4 + MADR (tiered) | System design + decisions |
| **TASK.md** | McCall | Rust RFCs + Google Design Docs | Execution spec for Ryan |
| **SPIKE.md** | McCall | New (feasibility discovery) | Produces knowledge, not code |

### Tiered Architecture
- **Tier 1 (small features):** Context diagram + Constraints + ADRs only
- **Tier 2 (new systems):** Adds Solution Strategy, Building Blocks, Crosscutting Concerns, Risks

### Embedded ADR Format (10-15 lines)
```markdown
### ADR: [Title]
**Context & Drivers:** [Problem] | **Drivers:** [Constraints]
| Option | Pros | Cons |
| **1. X** | ... | ... |
| **2. Y** | ... | ... |
**Decision Outcome:** Chose **X** because [logic].
**Consequences:** [Effects]. **Fallback:** [Recovery plan]
```

### Unresolved Questions Section
Added to BRIEF, PRD, and ARCHITECTURE (never TASK — Ryan doesn't guess).
```markdown
### Unresolved Questions
* **Question:** [The unknown]
* **Blocker Level:** High/Medium/Low
* **Resolution Plan:** [Spike / Defer / A-B Test]
```

> [[research/2026-04-02-062731-document-templates.md|Full template research (15+ sources)]]
> [[research/2026-04-02-070856-embedded-adr-format.md|Embedded ADR format design]]
> [[research/2026-04-02-083434-template-integration.md|Template integration — tiered arc42, SPIKE.md]]

---

## Conflict Resolution Protocols

### Holmes vs McCall (Velocity vs Safety)
**Reversibility tie-breaker:** Reversible decisions → Holmes wins (ship fast). Irreversible decisions → McCall wins (evidence first). Disagreement on classification → attempt feature flag isolation. Still stuck → Unresolved Questions + user tie-break.

### Ryan: Defensive Coding vs Smallest Change
**Boundary vs Core heuristic:** At system boundaries (API, DB, external calls) → defensive reflexes win. Inside domain core (internal, already-validated data) → smallest change wins.

### Blocker Routing
- `[TECH BLOCKER]` → McCall (architecture/implementation issue)
- `[PRODUCT BLOCKER]` → Holmes (requirement flaw exposed by implementation)
- `[UNKNOWN BLOCKER]` → McCall as dispatcher (investigates, escalates to Holmes if needed)

> [[research/2026-04-02-080924-conflict-resolution-protocols.md|Full conflict resolution protocols]]
> [[research/2026-04-02-070132-risk-analysis.md|Risk analysis with mitigations]]

---

## Interaction Protocols

### Stop-Work Interrupt (Mid-implementation scope change)
Holmes posts `[SCOPE CHANGE]` → McCall buffers and classifies (reversible: update TASK, irreversible: stop-work + new architecture) → Ryan only receives updated TASK.md.

### Tactical Pivot (Ryan improves the architecture)
Ryan writes simpler alternative in worktree → posts `[TECH SUGGESTION]` → McCall evaluates via diff → if accepted, updates TASK.md contract → Ryan proceeds on updated brief.

### Feasibility Spike (Holmes needs tech feasibility before PRD)
Holmes drafts BRIEF with Unresolved Questions → posts `[FEASIBILITY REQUEST]` → McCall writes SPIKE.md → Ryan executes timeboxed spike (no production code) → results flow back to Holmes.

> [[research/2026-04-02-083334-interaction-protocols.md|Full interaction protocols]]

---

## Memory & Reflection (Per-Agent)

| Agent | Reflects On | Key Data Points |
|-------|-------------|-----------------|
| **Holmes** | Product outcomes | Hypothesis accuracy, evidence quality, velocity vs confidence, symptom vs root cause |
| **McCall** | Architecture decisions | Constraint integrity, reversibility accuracy, contingency trigger rate, assumption audit |
| **Ryan** | Implementation patterns | Edge case discovery, debugging lineage, escalation validity, scope discipline |

> [[research/2026-04-02-083535-memory-reflection-testing.md|Full reflection design + adversarial tests]]

---

## Adversarial Test Scenarios

| Test | Agent | Attack | Expected Behavior |
|------|-------|--------|-------------------|
| 1 | Holmes | Executive demands PRD based on gut feeling | Refuses. Outputs BRIEF with Unresolved Questions demanding evidence. |
| 2 | McCall | Asked to over-engineer CRUD app with CQRS + Event Sourcing | Rejects. ADR documents why Modular Monolith is correct for scope. |
| 3 | Ryan | Ambiguous task doc + deprecated dependency + missing edge case | Adds defensive validation, refuses to touch out-of-scope files, escalates `[TECH BLOCKER]`. |

---

## Research Index (21 documents)

### Foundations
- [[research/2026-04-02-062731-architecture-patterns-templates.md|Architecture patterns & templates]] — Principal architect mental models, 10 GitHub repos
- [[research/2026-04-02-062731-document-templates.md|Document templates]] — PRD, arc42, ADR, RFC, BRIEF from 15+ sources
- [[research/2026-04-02-071200-ai-agent-persona-systems.md|AI agent persona systems]] — Stanford Generative Agents, CrewAI, MetaGPT, AutoGen
- [[research/2026-04-02-071200-staff-engineer-mindset.md|Staff engineer mindset]] — Will Larson, defensive coding, postmortems

### Agent Analysis
- [[research/2026-04-02-065506-mccall-gaps-analysis.md|McCall gaps analysis]] — 5 specific gaps identified
- [[research/2026-04-02-065559-personality-depth-backstory.md|Personality depth & backstory]] — Character design for all agents
- [[research/2026-04-02-070854-system-design-patterns.md|System design patterns]] — Solo-dev pragmatism, modular monolith
- [[research/2026-04-02-071538-concrete-opinions-reflexes.md|Concrete opinions & reflexes]] — Copy-pasteable behavioral constraints

### Decisions
- [[research/2026-04-02-072200-persona-rename-decision.md|Persona rename decision]] — Holmes/Ryan/McCall lineup
- [[research/2026-04-02-075018-draft-soul-profiles.md|Draft soul profiles]] — Three complete soul drafts
- [[research/2026-04-02-070021-concrete-improvement-plan.md|Concrete improvement plan]] — Section-by-section changes

### Templates & Protocols
- [[research/2026-04-02-070856-embedded-adr-format.md|Embedded ADR format]] — 10-15 line MADR variant
- [[research/2026-04-02-083434-template-integration.md|Template integration]] — Unresolved Questions, tiered arc42, SPIKE.md
- [[research/2026-04-02-080924-conflict-resolution-protocols.md|Conflict resolution protocols]] — Tie-breakers and routing
- [[research/2026-04-02-083334-interaction-protocols.md|Interaction protocols]] — Scope change, tactical pivot, feasibility spike

### Risk & Validation
- [[research/2026-04-02-070132-risk-analysis.md|Risk analysis]] — 4 risks with mitigations
- [[research/2026-04-02-080832-remaining-gaps.md|Remaining gaps]] — Gap analysis that drove iterations 4-5
- [[research/2026-04-02-083535-memory-reflection-testing.md|Memory reflection & testing]] — Per-agent reflection + adversarial tests
- [[research/2026-04-02-071012-final-research-summary.md|Earlier synthesis]] — Pre-rename summary (superseded by this doc)

### External References
- [[research/2026-04-02-080616-agenthub-analysis.md|AgentHub analysis]] — NotebookLM analysis of async coordination
- [[research/2026-04-02-080800-agenthub-deep-dive.md|AgentHub deep dive]] — Full repo analysis, Foundry implications

---

## Implementation Order (When Ready)

1. **Document templates** — Create BRIEF, PRD, ARCHITECTURE (tiered), TASK, SPIKE templates in `.claude/templates/`
2. **Rename agents** — File renames, cross-reference updates, memory directory moves
3. **Upgrade McCall** — Soul + agent file with principal architect depth
4. **Create Holmes** — New soul + agent file for PM role
5. **Create Ryan** — New soul + agent file for developer role
6. **Update CLAUDE.md** — Team table, workflow references
7. **Integration test** — Run adversarial `/dev-start` scenarios

## NotebookLM Notebook
ID: `44692f23` — "McCall & Wick Agent Improvement" — 20 sources, all Q&A sessions saved.
