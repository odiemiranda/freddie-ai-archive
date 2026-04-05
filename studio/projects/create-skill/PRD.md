# PRD: Skill Specification, Validation, and Scaffolding

> Owner: Holmes | Source: [[BRIEF]] | Status: draft

## Overview

Build three artifacts that formalize the implicit skill spec governing 26 Claude Code skills, make structural failures detectable, and reduce new skill creation to a single guided command. The core deliverable is a **formal skill spec** backed by a **validator** that catches the silent failures that today go unnoticed (bad frontmatter, dead globs, description truncation). The scaffold is a convenience layer on top -- it generates correct structure, but the spec and validator are what make correctness enforceable.

Dependency order: **Skill Spec** (reference doc) --> **validate-skill** (CLI tool) --> **create-skill** (CLI tool + skill).

## Goals & Non-Goals

### Goals

1. **Every structural convention is documented.** The 3 implicit tiers (simple, medium, complex), frontmatter rules, workflow.md node conventions, and asset patterns exist in a single reference document.
2. **Silent failures are detectable.** A validator catches: broken frontmatter, description > 250 chars, dead path globs, missing workflow.md references, name collisions, broken internal links.
3. **New skill creation is guided.** A single command collects intent, determines tier, confirms plan, scaffolds files, and validates the output.
4. **Existing skills are auditable.** The validator runs retroactively on all 26 skills with a `--all` flag, surfacing any current violations.

### Non-Goals

- **Cross-platform skill format.** Claude Code only. No adapters for Cursor, Windsurf, or other AI coding tools.
- **Content quality scoring.** The validator checks structure, not whether the skill's instructions are effective. Content quality is human judgment.
- **Staleness detection.** `last_reviewed` tracking and dependency health monitoring are v2 concerns. (Evidence: BRIEF Non-Goals, research identified these as useful but deferred.)
- **Auto-classification of existing skills into tiers.** The validator reports what it finds; it does not retroactively label existing skills.
- **Team registries or skill sharing.** Single-user system. (Evidence: BRIEF Constraints.)
- **Skill versioning or migration tooling.** If the spec changes, the validator catches drift. No automated migration.
- **Mermaid syntax validation.** Checking that mermaid diagrams are syntactically valid adds parser complexity with low payoff. Existence checks only. (Resolution of BRIEF Unresolved Question 2.)
- **Auto-detected tier selection.** Rule-based heuristic ("keywords like sub-agent, critic, multi-phase") is unreliable without training data. Explicit user selection for v1. (Resolution of BRIEF Unresolved Question 3.)

## User Stories

### Must Have

**US-1: Validate an existing skill for structural correctness**

**As a** skill author, **I want to** run a validation command on any skill directory **so that** I discover structural problems that would cause silent failure in Claude Code discovery.

- [ ] AC-1: `bun src/tools/validate-skill.ts {skill-name}` accepts a skill name (looked up in `.claude/skills/{name}/`) or a path to a skill directory.
- [ ] AC-2: Validator checks all required frontmatter fields: `name` (present, lowercase, hyphens only, 1-64 chars, matches directory name), `description` (present, <= 250 chars), `user-invocable` (present, boolean), `argument-hint` (present, string).
- [ ] AC-3: Validator checks `agent` field if present -- value must be a recognized agent name or `none`.
- [ ] AC-4: Validator checks `name` does not collide with any other existing skill name.
- [ ] AC-5: Validator checks that if SKILL.md references `workflow.md` (text match), the file exists in the same directory.
- [ ] AC-6: Validator checks that path globs referenced in skill files resolve to at least one existing file on disk.
- [ ] AC-7: Validator checks that internal markdown links (e.g., `[PX-NNN]` referencing sections in workflow.md) have corresponding section headers.
- [ ] AC-8: Validator checks that mermaid code blocks exist in SKILL.md for skills that have a workflow.md (medium/complex tiers).
- [ ] AC-9: Output is structured as three severity levels: **error** (will cause silent failure -- blocks), **warning** (deviates from convention but may be intentional -- informational), **info** (suggestions).
- [ ] AC-10: Exit code 0 when no errors. Exit code 1 when errors exist.

**Priority:** Must

---

**US-2: Batch-validate all existing skills**

**As a** skill author, **I want to** validate all 26 skills in a single command **so that** I can audit the current state and identify any existing violations before adopting the spec.

- [ ] AC-1: `bun src/tools/validate-skill.ts --all` discovers all skill directories under `.claude/skills/` and runs validation on each.
- [ ] AC-2: Output is a summary table: skill name, error count, warning count, info count, pass/fail status.
- [ ] AC-3: Detailed findings follow the summary, grouped by skill.
- [ ] AC-4: Final line reports total pass/fail count (e.g., "22/26 passed, 4 with errors").

**Priority:** Must

---

**US-3: Formal skill spec document exists as a reference**

**As a** skill author, **I want to** consult a single document that codifies all structural conventions for skills **so that** I understand the rules the validator enforces and can make informed structural decisions.

- [ ] AC-1: Spec document exists at `.claude/references/skill-spec.md`. (Resolution of BRIEF Unresolved Question 1 -- see Assumptions A-3.)
- [ ] AC-2: Spec defines the three structural tiers with criteria for when each applies:
  - **Simple:** SKILL.md only, 15-50 lines, no workflow.md, no sub-agent dispatch. Examples: `morning`, `daily`, `brain`, `gemini`, `bye`.
  - **Medium:** SKILL.md + workflow.md, moderate complexity, may have sub-agent dispatch. Examples: `tracknames`, `create-music-card`, `create-track-variant`.
  - **Complex:** SKILL.md + workflow.md + optional `assets/` or `scripts/`, multi-phase with review gates, sub-agent dispatch, mermaid diagrams. Examples: `create-track`, `dev-start`, `architect`, `lyrics`.
- [ ] AC-3: Spec defines all frontmatter fields, their types, constraints, and which are required vs optional.
- [ ] AC-4: Spec defines workflow.md conventions: TOC format, node ID pattern (`[PX-NNN]`), phase naming, mermaid diagram expectations.
- [ ] AC-5: Spec defines the `assets/` and `scripts/` subdirectory conventions and when each is appropriate.
- [ ] AC-6: Spec defines description length limit (250 chars) with rationale (Claude Code truncation).
- [ ] AC-7: Spec defines internal link conventions (SKILL.md referencing workflow.md sections).

**Priority:** Must

---

**US-4: Scaffold a new skill from intent**

**As a** skill author, **I want to** describe what a new skill should do and have the system generate the correct file structure **so that** I start from a valid skeleton instead of copying and adapting an existing skill.

- [ ] AC-1: `bun src/tools/create-skill.ts` is a CLI tool that accepts structured input (name, description, tier, agent) and scaffolds files.
- [ ] AC-2: `/create-skill` is a skill (`.claude/skills/create-skill/SKILL.md`) that orchestrates the interactive flow: collect intent, determine tier, confirm plan, scaffold, validate.
- [ ] AC-3: User provides natural language description of what the skill should do.
- [ ] AC-4: System asks user to select tier: simple, medium, or complex -- with guidance on when each applies (drawn from the skill spec).
- [ ] AC-5: System presents a plan before generating: proposed name, tier, files to create, frontmatter values, structural decisions.
- [ ] AC-6: User confirms before any files are written to disk. (Evidence: BRIEF Constraints -- generation safety rule.)
- [ ] AC-7: Scaffold writes files to `.claude/skills/{name}/`.
- [ ] AC-8: Scaffold generates structural skeleton only -- no placeholder instructions or filler text. (Resolution of BRIEF Unresolved Question 4.)
  - Simple tier: SKILL.md with valid frontmatter + `## Workflow` heading (empty).
  - Medium tier: SKILL.md with frontmatter + execution model table + mermaid skeleton + phase headings; workflow.md with TOC + node headings (empty bodies).
  - Complex tier: Same as medium + `assets/` directory (if applicable) + richer mermaid skeleton with review gates.
- [ ] AC-9: Scaffold sets `agent: none` as default in frontmatter. Author updates if sub-agent dispatch is needed. (Resolution of BRIEF Unresolved Question 5.)
- [ ] AC-10: After scaffolding, `validate-skill` runs automatically on the output. Results are displayed to the user.
- [ ] AC-11: If validation finds errors in the scaffold output, the tool reports them and does NOT silently succeed.

**Priority:** Must

---

### Should Have

**US-5: Validation checks for recognized agent names**

**As a** skill author, **I want** the validator to check that the `agent` field value (when present and not `none`) corresponds to a known agent in the system **so that** typos in agent names don't cause silent dispatch failures.

- [ ] AC-1: Validator discovers known agents by scanning `.claude/agents/` and/or `.claude/souls/` directories.
- [ ] AC-2: An `agent` value not matching any known agent produces a **warning** (not error -- the agent might be defined elsewhere or planned).

**Priority:** Should

---

**US-6: Rule file that auto-loads when editing skills**

**As a** skill author, **I want** key skill conventions to load automatically when I edit files under `.claude/skills/` **so that** I don't have to remember to consult the spec manually.

- [ ] AC-1: A rule file exists at `.claude/rules/skill-structure.md` with glob pattern matching `.claude/skills/**`.
- [ ] AC-2: Rule file contains a concise summary of the spec (frontmatter requirements, tier guidance, common mistakes) and references the full spec at `.claude/references/skill-spec.md`.
- [ ] AC-3: Rule file does NOT duplicate the full spec -- it summarizes and links.

**Priority:** Should

---

**US-7: Descriptive error messages with fix guidance**

**As a** skill author, **I want** validation errors to explain what is wrong and how to fix it **so that** I can resolve issues without consulting external documentation.

- [ ] AC-1: Each error/warning includes: the check name, what was found, what was expected, and a one-line fix suggestion.
- [ ] AC-2: Example: `ERROR [frontmatter.description-length]: Description is 287 chars (max 250). Shorten to avoid Claude Code truncation.`

**Priority:** Should

---

### Could Have

**US-8: JSON output mode for programmatic consumption**

**As a** tool author, **I want** the validator to optionally output results as JSON **so that** other tools (CI, brain.db sync, future dashboards) can consume validation results programmatically.

- [ ] AC-1: `bun src/tools/validate-skill.ts {name} --json` outputs structured JSON instead of human-readable text.
- [ ] AC-2: JSON schema includes: `skill`, `tier` (detected), `errors[]`, `warnings[]`, `info[]`, each with `check`, `message`, `location` (file + line if applicable).

**Priority:** Could

---

**US-9: Dry-run mode for scaffold**

**As a** skill author, **I want to** see what the scaffold would generate without actually writing files **so that** I can evaluate the plan before committing to disk.

- [ ] AC-1: `bun src/tools/create-skill.ts --dry-run` outputs the file tree and file contents to stdout without writing to disk.
- [ ] AC-2: The `/create-skill` interactive flow already has a confirm step (US-4 AC-6), so dry-run is primarily useful for the CLI tool in non-interactive usage.

**Priority:** Could

## Assumptions & Evidence

| # | Assumption | Evidence | Confidence |
|---|-----------|----------|------------|
| A-1 | Claude Code truncates skill descriptions at 250 characters, causing discovery failure for longer descriptions | BRIEF evidence table: "Claude Code truncates descriptions > 250 chars, breaking discovery" sourced from official Claude Code docs | High |
| A-2 | The three structural tiers (simple/medium/complex) are a valid categorization of the 26 existing skills | BRIEF investigation: codebase analysis of line counts and file structures across all 26 skills. Simple (9 skills, SKILL.md only, 16-40 lines), medium (several with SKILL.md + workflow.md, moderate), complex (10+ with SKILL.md + workflow.md + assets, multi-phase) | High |
| A-3 | The skill spec document should live at `.claude/references/skill-spec.md` with a companion rule file at `.claude/rules/skill-structure.md` | BRIEF Unresolved Question 1 suggested three options. Resolution: a reference doc is more discoverable for human reading and more appropriate for a comprehensive spec. A companion rule file (US-6) auto-loads the essential subset when editing skills. This gives both benefits without duplication. | Medium -- McCall should validate this placement during architecture |
| A-4 | Frontmatter fields `name`, `description`, `user-invocable`, and `argument-hint` are required by Claude Code for skill discovery | Observed in all 26 skills -- every skill has these four fields. `agent` appears in most but not all (some omit it). Official Claude Code docs confirm discovery relies on frontmatter. | High |
| A-5 | Existing skills will have some validation failures when audited retroactively | BRIEF confidence assessment: "Running validate-skill --all against the current 26 skills and seeing what fails" was identified as the key data point. The `save-raw` skill is deprecated, which suggests at minimum one structural edge case. | Medium |
| A-6 | The meta-skill pattern (create-brief, create-prd, create-task, architect) is the correct pattern for create-skill | BRIEF evidence: "Established pattern: a meta-skill that generates structured documents using a template + validation." All four follow the same shape: Freddie orchestrates, sub-agent drafts, user reviews, save. | High |
| A-7 | Skeleton scaffolding (no placeholder text) is better than pre-populated content | BRIEF Unresolved Question 4 resolution: "Placeholder text risks being left in place and creating misleading instructions." Observation: the 26 existing skills have highly specific, domain-dependent content that generic placeholders would not capture. | High |

## Success Metrics

| Metric | Baseline | Target |
|--------|----------|--------|
| Skills with documented structural spec | 0 (no spec exists) | 1 comprehensive spec covering all 3 tiers |
| Detectable silent failure modes | 0 (failures are invisible) | >= 5 distinct check categories (frontmatter, naming, structure, links, globs) |
| Time to create a new skill (from intent to valid skeleton) | ~30 min (study existing skills, copy, adapt, hope) | < 5 min (single command with guided flow) |
| Existing skills passing validation | Unknown | Measured via `--all` run. Target: identify and remediate all errors within one session. |
| Validation coverage | 0 checks | >= 10 distinct validation checks across error/warning/info |

## Out of Scope

- **Runtime skill behavior testing.** The validator checks structure, not whether the skill produces correct results when invoked. Testing skill behavior requires actually running the skill, which is outside this scope.
- **Skill dependency graph.** Some skills reference others (e.g., `dev-start` references `create-task`, `architect`). Mapping these dependencies is useful but is a separate concern from structural validation.
- **Automatic remediation.** The validator reports problems. It does not fix them. Auto-fix is a v2 feature if validation adoption proves the value.
- **Workflow.md content linting.** Checking that workflow.md instructions are complete, correct, or follow best practices is content quality -- not structural validation.
- **Skill analytics.** Tracking which skills are invoked, how often, success rates. This requires instrumentation that does not exist today.
- **Scaffold customization beyond tier.** The scaffold uses the spec's tier definitions. Custom structural templates (e.g., "scaffold like create-track but without Phase 3") are not supported in v1.

## Deliverables

Three artifacts, in dependency order:

```
1. Skill Spec (.claude/references/skill-spec.md)
   +  Rule file (.claude/rules/skill-structure.md)
   |
   v
2. validate-skill (src/tools/validate-skill.ts)
   |
   v
3. create-skill (src/tools/create-skill.ts + .claude/skills/create-skill/)
```

### Deliverable 1: Skill Spec + Rule File

| Attribute | Value |
|-----------|-------|
| Files | `.claude/references/skill-spec.md`, `.claude/rules/skill-structure.md` |
| Owner | McCall (architecture) |
| Dependencies | None |
| Acceptance | US-3 (all ACs), US-6 (all ACs) |

### Deliverable 2: validate-skill CLI

| Attribute | Value |
|-----------|-------|
| Files | `src/tools/validate-skill.ts` |
| Owner | Wick (implementation), McCall (spec + review) |
| Dependencies | Deliverable 1 (spec defines what to validate) |
| Acceptance | US-1 (all ACs), US-2 (all ACs), US-5 (all ACs), US-7 (all ACs) |

### Deliverable 3: create-skill CLI + Skill

| Attribute | Value |
|-----------|-------|
| Files | `src/tools/create-skill.ts`, `.claude/skills/create-skill/SKILL.md`, `.claude/skills/create-skill/workflow.md` |
| Owner | Wick (implementation), McCall (spec + review) |
| Dependencies | Deliverable 2 (scaffold runs validator on output) |
| Acceptance | US-4 (all ACs) |

## Validation Check Inventory

This table enumerates the specific checks the validator must perform. Each maps to a user story AC.

| # | Check | Severity | What it catches | User Story |
|---|-------|----------|----------------|------------|
| V-1 | `frontmatter.required-fields` | Error | Missing `name`, `description`, `user-invocable`, or `argument-hint` | US-1 AC-2 |
| V-2 | `frontmatter.name-format` | Error | `name` not lowercase, contains non-hyphen special chars, or exceeds 64 chars | US-1 AC-2 |
| V-3 | `frontmatter.name-matches-dir` | Error | `name` value does not match the skill directory name | US-1 AC-2 |
| V-4 | `frontmatter.description-length` | Error | `description` exceeds 250 characters (will be truncated by Claude Code) | US-1 AC-2 |
| V-5 | `frontmatter.name-collision` | Error | Another skill already uses this `name` | US-1 AC-4 |
| V-6 | `structure.workflow-ref` | Error | SKILL.md references `workflow.md` but the file does not exist | US-1 AC-5 |
| V-7 | `structure.dead-globs` | Warning | Path globs in skill files don't resolve to any existing files | US-1 AC-6 |
| V-8 | `structure.internal-links` | Warning | `[PX-NNN]` node IDs in SKILL.md don't have matching headers in workflow.md | US-1 AC-7 |
| V-9 | `structure.mermaid-exists` | Warning | SKILL.md has a workflow.md but no mermaid code block | US-1 AC-8 |
| V-10 | `frontmatter.agent-known` | Warning | `agent` value is not `none` and doesn't match a known agent | US-5 AC-1/2 |
| V-11 | `frontmatter.description-empty` | Error | `description` is empty or whitespace-only | US-1 AC-2 |
| V-12 | `frontmatter.user-invocable-type` | Error | `user-invocable` is not a boolean value | US-1 AC-2 |

## Unresolved Questions

* **Question:** Should the validator treat the deprecated `save-raw` skill differently? Its description starts with `[DEPRECATED]` which is a legitimate pattern but might trigger description-length or content warnings.
* **Blocker Level:** Low
* **Resolution Plan:** McCall decides during architecture. Likely: the validator treats all skills equally. Deprecation is a content concern, not a structural one. If `[DEPRECATED]` causes the description to exceed 250 chars, that is a real error.

* **Question:** Should the `--all` batch run have a `--fix` mode that auto-remediates trivial issues (e.g., trailing whitespace in descriptions)?
* **Blocker Level:** Low
* **Resolution Plan:** Defer to v2. The validator's job in v1 is to report, not to fix. Auto-remediation needs careful scoping to avoid unintended changes.

* **Question:** The `agent` field appears in most skills but not all (e.g., `brain` and `save-raw` omit it). Is `agent` a required field with default `none`, or truly optional?
* **Blocker Level:** Medium
* **Resolution Plan:** Analyze all 26 skills during Deliverable 1 (spec creation). If the majority include `agent`, the spec should require it with `none` as default. If omission is common and intentional, it stays optional. The `--all` run will provide empirical data.
