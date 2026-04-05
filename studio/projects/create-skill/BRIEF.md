# BRIEF: Skill Specification and Scaffolding

> Owner: Holmes | Status: draft

## Problem Statement

**We believe** codifying the skill spec and building a scaffold + validator **will** eliminate silent skill failures and reduce new skill creation from "study 3 existing skills and hope you got the frontmatter right" to a single command **for** the user (solo developer) **because** the current system has 26 skills with no formal spec, 3 implicit structural tiers, and failure mode is silent non-discovery by Claude Code.

## Investigation Notes

### Five Whys (Root Cause)

| # | Question | Answer |
|---|----------|--------|
| 1 | Why do we need `/create-skill`? | Building new skills requires knowing exact structure — frontmatter, workflow.md node pattern, mermaid conventions. |
| 2 | Why is that a problem? | With 26 skills across 3 structural tiers (simple/medium/complex), conventions are implicit — learned by reading existing skills, not documented. |
| 3 | Why are they implicit? | Skills grew organically. `morning` is 16 lines. `create-track` is 320 + 750 (workflow). No spec document exists. |
| 4 | Why does that matter now? | System is scaling — new agent roles (Holmes, Ryan), new workflows added regularly. Each new skill risks structural drift. |
| 5 | Why is drift the real risk? | Skills are the primary user-agent interface. A malformed skill (bad frontmatter, truncated description, dead globs) **fails silently** — Claude Code just doesn't discover or trigger it. |

**Root cause:** There is no formal skill spec. Correctness is unenforceable and failure is silent. The scaffold is the secondary deliverable — the spec and validator are the primary ones.

### Falsification Attempt

**Hypothesis to disprove:** "The current manual approach is inadequate and a tool is needed."

**Counter-evidence considered:**
1. *"26 skills already exist and work fine without tooling."* True, but they were written by the same person in a short timeframe with full context. The question is whether this scales — new skills in future sessions when context has been lost. The `daily` skill (21 lines) and `dev-start` skill (214 lines + 571-line workflow) follow fundamentally different structural patterns with no documented reason why.
2. *"Just use an existing skill as a template."* This is the current approach. The problem is: which skill do you copy? The answer depends on the complexity tier, whether the skill delegates to a sub-agent, whether it needs a workflow.md, and whether it has assets. That selection logic is itself undocumented.
3. *"A simple README documenting the spec would suffice."* Partially valid. A spec document alone would solve the "implicit conventions" problem. But it would not catch silent failures (dead globs, truncated descriptions, broken internal links). The validator is what makes the spec enforceable.

**Verdict:** The hypothesis holds. A spec doc alone is insufficient because it does not prevent silent failure. The validator is the critical piece. The scaffold is a convenience layer on top of the validated spec.

### Triangulation

| Signal | Source | Finding |
|--------|--------|---------|
| Structural variance across 26 skills | Direct codebase analysis | 3 tiers: (1) SKILL.md only, 16-40 lines, no workflow (morning, daily, weekly, save-raw, brain, gemini, minimax-art, minimax-video, meeting); (2) SKILL.md + workflow.md, moderate (tracknames, create-track-variant, minimax-art/video with workflows); (3) SKILL.md + workflow.md + optional assets, complex (create-track, dev-start, architect, create-prd, create-task, create-brief, code-review, security-audit, lyrics, pr-review) |
| Claude Code skill discovery mechanism | Official docs (code.claude.com) | Discovery relies on frontmatter: `name`, `description` (truncated at 250 chars), `user-invocable`, `argument-hint`. Broken frontmatter = invisible skill. |
| External tooling patterns | Research summary (Skill Forge, Build Skill CLI, Agent Skill Creator) | All 3 external tools validate after scaffolding. Skill Forge has audit + benchmark. Build Skill has CI/CD. All validate frontmatter as minimum. |
| Existing meta-skills in this codebase | create-brief, create-prd, create-task, architect | Established pattern: a meta-skill that generates structured documents using a template + validation. create-skill would follow this exact pattern. |
| No prior decisions in brain.db | brain.db --search-all, --recall-all | Zero results for skill creation/scaffold/validation. This is genuinely new territory — no prior architecture decisions to respect or contradict. |

**Confidence: 85%.** Three independent signals converge: the codebase shows structural variance without a spec, Claude Code's discovery mechanism makes bad frontmatter a silent failure, and external tooling universally validates after scaffolding. The only uncertainty is scope — how much validation is enough for v1.

## Evidence

| Signal | Source | Confidence |
|--------|--------|------------|
| 26 skills with 3 implicit tiers, no documented spec | Codebase analysis (line counts, file structures) | High |
| Claude Code truncates descriptions > 250 chars, breaking discovery | Official Claude Code docs | High |
| Silent failure mode — bad frontmatter = invisible skill | Claude Code discovery mechanism | High |
| External tools all validate post-scaffold | Skill Forge, Build Skill CLI, Agent Skill Creator research | Medium |
| Existing meta-skill pattern (create-brief, create-prd, create-task) works well | Codebase analysis | High |
| No prior architecture decisions exist for this feature | brain.db search (5 queries, 0 relevant results) | High |
| Skills grew organically — `morning` (16 lines) to `create-track` (320 + 750 lines) | File analysis | High |

## Goals

1. **Formal skill spec exists.** Every structural convention currently implicit in the 26 skills is documented in a single reference.
2. **Silent failures are catchable.** A validator can detect: bad frontmatter, description > 250 chars, dead path globs, missing workflow.md references, name collisions.
3. **New skill creation is guided.** User provides intent (natural language), system determines the correct tier and scaffolds accordingly.
4. **Existing skills are validatable.** The validator runs retroactively on all 26 skills — any current violations are surfaced.

## Constraints

| Constraint | Source |
|-----------|--------|
| CLI-first — tools are `bun src/tools/...`, no MCP for local use | Project convention (CLAUDE.md) |
| Single user — no team registries, no multi-user scaffolding | Project scope |
| Skills are pure markdown — SKILL.md + workflow.md, no TypeScript runtime | Claude Code skill format |
| Must work within existing agent dispatch model (Freddie orchestrates, agents execute) | Architecture rule |
| Confirm before generating — user approves plan before scaffold writes to disk | Generation safety rule |
| No inline `bun -e` in workflow files — if logic is needed, create a CLI tool | Command execution rule |
| `out/` for drafts, `.claude/skills/{name}/` for final output | Project convention |
| Existing naming: lowercase, hyphens, 1-64 chars | Claude Code frontmatter spec |

## Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Over-engineered scaffold that imposes structure the user doesn't want | Medium | Medium | Tier detection — simple skills get minimal scaffold, complex skills get full structure. User reviews before write. |
| Validator too strict — flags legitimate structural choices as errors | Medium | Low | Separate errors (blocks) from warnings (informational). Errors = will silently fail. Warnings = deviates from convention but may be intentional. |
| Generated workflow.md is too generic to be useful | Medium | Medium | The scaffold provides structure (phases, nodes, mermaid skeleton), not content. Content is always skill-specific — the scaffold should not pretend otherwise. |
| Spec document drifts from actual skill patterns over time | Low | Medium | Validator enforces the spec. If a new pattern emerges, the spec and validator are updated together. They are coupled by design. |
| Scaffold becomes the only way to create skills, reducing flexibility | Low | Low | Scaffold is additive — manual creation still works. Validator is the enforcement layer, not the scaffold. |

## Non-Goals

- **Cross-platform skill format.** This is Claude Code only. No adapters for other AI coding tools.
- **Benchmarking or quality scoring of skill content.** The validator checks structure, not whether the skill's instructions are good. Content quality is a human judgment.
- **Staleness checker for v1.** Research identified `last_reviewed` tracking and dependency health as useful, but these are v2 concerns. The spec and validator are the foundation.
- **Auto-detection of tier from existing skills.** The validator reports what it finds; it does not retroactively classify existing skills into tiers.
- **Team registries or skill sharing.** Single user system.
- **Skill versioning or migration.** If the spec changes, the validator catches drift. No automatic migration.

## Deliverables

Three artifacts, in dependency order:

```
1. Skill Spec (reference doc)
   └── 2. validate-skill (CLI tool — enforces the spec)
       └── 3. create-skill (CLI tool + skill — uses the spec, runs the validator)
```

### 1. Skill Spec Document

A reference document (likely in `.claude/references/` or `.claude/docs/`) that codifies:
- Required frontmatter fields and their constraints
- The three structural tiers and when each applies
- Workflow.md conventions: TOC format, node naming (`[PX-NNN]`), mermaid diagram requirements
- The `assets/` convention
- Description length limits
- Internal link conventions (SKILL.md references workflow.md sections)

### 2. `validate-skill` CLI Tool

`bun src/tools/validate-skill.ts` — standalone, runs on any skill directory.

**Input:** skill name or path to skill directory
**Output:** structured validation report (errors, warnings, info)

**Checks (minimum):**
- Frontmatter: all required fields present, `name` matches directory name, `description` <= 250 chars
- Name: lowercase, hyphens only, 1-64 chars, no collision with existing skill or command
- Structure: workflow.md exists if SKILL.md references it, internal links resolve
- Workflow.md: TOC present, node IDs match section headers, mermaid diagram present for non-trivial skills
- Dead rules: path globs in skill files that don't match actual files on disk

**Batch mode:** `bun src/tools/validate-skill.ts --all` — runs on all 26 skills, reports results.

### 3. `create-skill` CLI Tool and Skill

**CLI tool:** `bun src/tools/create-skill.ts` — takes structured input, scaffolds files.

**Skill:** `.claude/skills/create-skill/SKILL.md` — the `/create-skill` slash command that orchestrates the user interaction (intent collection, tier detection, confirmation, scaffold, validation).

**Flow:**
1. User provides intent (natural language description of what the skill should do)
2. System determines tier (simple / medium / complex) based on: sub-agent delegation needed? multi-phase workflow? user review gates?
3. Present plan: tier, proposed files, frontmatter, structural decisions
4. User confirms
5. Scaffold files to `.claude/skills/{name}/`
6. Run `validate-skill` on the output
7. Report results

## Unresolved Questions

* **Question:** Where does the skill spec document live? Options: `.claude/references/skill-spec.md`, `.claude/docs/skill-spec.md`, or a rule file in `.claude/rules/skill-spec.md` (auto-loaded when editing skill files).
* **Blocker Level:** Low
* **Resolution Plan:** McCall decides during architecture. A rule file has the advantage of auto-loading when editing skills. A reference doc is more discoverable for human reading. Could be both — rule file references the full spec doc.

* **Question:** Should the validator check that mermaid diagrams in SKILL.md are syntactically valid, or just that they exist?
* **Blocker Level:** Low
* **Resolution Plan:** Start with existence check only. Mermaid syntax validation is a nice-to-have but adds parser complexity.

* **Question:** How should tier detection work for the scaffold? Rule-based heuristic (keywords like "sub-agent", "critic", "multi-phase") or explicit user selection?
* **Blocker Level:** Medium
* **Resolution Plan:** Start with explicit user selection ("simple / medium / complex") with guidance on when each applies. Auto-detection is a v2 enhancement if the heuristic proves reliable.

* **Question:** Should the scaffold pre-populate workflow.md with actual node content (like placeholder instructions) or just the structural skeleton (TOC + empty sections)?
* **Blocker Level:** Medium
* **Resolution Plan:** Structural skeleton only. Content is always skill-specific. Placeholder text risks being left in place and creating misleading instructions. The skeleton gives the author the correct structure to fill in.

* **Question:** The `create-brief` SKILL.md references a "Ryan sub-agent" but the research summary renames Ryan to the developer role. Is the agent dispatch model stable enough to encode in the scaffold, or will it change again?
* **Blocker Level:** Low
* **Resolution Plan:** The scaffold should use a generic `agent: none` default in frontmatter and let the skill author set the dispatch model. The scaffold documents the convention but does not force a specific agent.

## Confidence Assessment

**Overall: 85%.**

The problem is well-evidenced (silent failure from implicit spec, structural variance across 26 skills, no prior art in brain.db). The solution shape is clear (spec + validator + scaffold, in that dependency order). The external research confirms the approach (all 3 external tools validate post-scaffold).

The 15% uncertainty is in scope calibration — specifically around tier detection heuristics and how much structure the scaffold should impose. These are design decisions for McCall, not product decisions for Holmes.

**What would move me to 95%:** Running `validate-skill --all` against the current 26 skills and seeing what fails. That empirical data would confirm whether the proposed validation checks are the right ones, or whether the current skills have patterns the spec should accommodate rather than flag.
