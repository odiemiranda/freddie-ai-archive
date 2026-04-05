# TASK: Create Skill Specification and Rule File

> Owner: McCall | Assigned: Ryan | Source: [[ARCHITECTURE]]

## What to Build

Two reference documents that codify the implicit structural conventions of the 26 existing Claude Code skills in this project.

**File 1: `.claude/references/skill-spec.md`** -- The full, authoritative specification. This is a human-readable reference document that skill authors consult when building or modifying skills. It does NOT execute at runtime. It does NOT get read by any tool. It is the single source of truth for "what makes a valid skill."

**File 2: `.claude/rules/skill-structure.md`** -- A concise auto-loading summary rule with glob `globs: [".claude/skills/**"]`. This rule auto-loads whenever anyone edits files inside `.claude/skills/`. It contains a quick-reference subset of the spec and a link to the full spec.

The `.claude/references/` directory does not exist yet. Create it.

### Spec Content (skill-spec.md)

The spec must contain these sections, in this order:

**1. Frontmatter Fields Table**

| Field | Type | Required | Constraints | Default |
|-------|------|----------|-------------|---------|
| `name` | string | Yes | Lowercase, hyphens only, 1-64 chars. Regex: `/^[a-z][a-z0-9-]{0,63}$/`. Must match directory name. | -- |
| `description` | string | Yes | 1-250 chars. Claude Code truncates beyond 250 -- skill becomes invisible/garbled in discovery. | -- |
| `user-invocable` | boolean | Recommended | If absent, Claude Code may not surface the skill on `/` invocation. | `true` |
| `argument-hint` | string | Recommended | Shown in Claude Code autocomplete. Empty string is valid. | `""` |
| `agent` | string | Recommended | Must be `none` or match a filename (without `.md`) in `.claude/agents/`. | `none` |

Include a note: `tracknames` uses a non-standard frontmatter format (`allowed_tools` instead of `user-invocable`/`argument-hint`). This is legacy. New skills must use the standard format above.

**2. Tier Definitions**

Define tiers by file structure, not complexity labels:

| Tier | Structure | Criteria | Examples |
|------|-----------|----------|----------|
| Tier 1 | `SKILL.md` only | Single-phase skill. Instructions fit in one file. No multi-step workflow. | morning, daily, weekly, save-raw, meeting, brain, minimax-art, minimax-video, penny |
| Tier 2 | `SKILL.md` + `workflow.md` | Multi-phase skill requiring separate node-by-node instructions. SKILL.md contains the overview, flow diagram, and phase table. workflow.md contains detailed per-node instructions. | tracknames, lyrics, create-track-variant, create-music-card, pr-review, security-audit, code-review, create-brief, create-prd, create-task, dev-start, dev-end |
| Tier 3 | `SKILL.md` + `workflow.md` + subdirectories | Tier 2 plus `assets/` and/or `scripts/` subdirectories for external resources (system prompts, helper scripts). | gemini, minimax-music, create-track |

**3. workflow.md Conventions**

- **TOC block:** HTML comment at top of file. Format:
  ```
  <!-- TOC
  | Node | Line | Title | Phase |
  |------|------|-------|-------|
  | P1-001 | 18  | Node title | 1 - Phase Name |
  ...
  TOC -->
  ```
- **Node ID pattern:** `[PX-NNN]` where X = phase number, NNN = zero-padded sequence. Regex for detection: `/\[([A-Z][0-9][-_][0-9]{3})\]/g`
- **Node heading format:** `## [PX-NNN] Node Title`
- **Each node contains:** Runs-in context, Input, Output, detailed instructions, Next pointer
- **Phase naming:** `Phase N - Descriptive Name` (e.g., `1 - Analysis & Slicing`)

**4. Mermaid Diagram Expectations**

- Tier 2 and Tier 3 skills should include a mermaid flow diagram in SKILL.md showing the phase flow
- The diagram nodes should correspond to the `[PX-NNN]` IDs in workflow.md
- Tier 1 skills do not need diagrams (they have no multi-phase flow)

**5. Subdirectory Conventions**

- `assets/` -- static files consumed by the skill (system prompts, templates, reference data)
- `scripts/` -- executable helper scripts called by the workflow
- Both are optional. Only create them if the skill needs external resources.
- If SKILL.md or workflow.md references these directories, they must exist.

**6. Description Length Limit**

- Hard limit: 250 characters
- Claude Code truncates descriptions beyond this limit, which can make the skill invisible or garbled in the `/` autocomplete
- Descriptions must be non-empty
- Write descriptions that explain what the skill does from the user's perspective, not implementation details

**7. Internal Link Conventions**

- `[PX-NNN]` IDs in SKILL.md (e.g., in phase tables or flow descriptions) must have matching `## [PX-NNN]` headers in workflow.md
- Dead links = the SKILL.md promises a workflow node that doesn't exist

**8. Anti-Patterns**

- Inline `bun -e "..."` in workflow files (violates command-execution rule -- create a CLI tool instead)
- Placeholder text in scaffolded skills ("TODO: fill in here") -- structure communicates intent, not placeholder prose
- Description over 250 chars -- causes silent Claude Code discovery failure
- `name` field that doesn't match the directory name -- skill becomes unreachable
- References to assets/ or scripts/ that don't exist -- dead globs cause silent failures
- Using `allowed_tools` instead of `user-invocable`/`argument-hint` in new skills (legacy format)

### Rule Content (skill-structure.md)

The rule file must have this frontmatter:

```yaml
---
globs: [".claude/skills/**"]
---
```

Body content (concise -- this loads into context on every skill edit):

1. **Frontmatter quick-reference:** Required fields (`name`, `description`), recommended fields (`user-invocable`, `argument-hint`, `agent`), constraints (name format, description 250 char limit)
2. **Tier selection guidance:** One line per tier explaining when to use it
3. **Common mistakes:** Top 3-4 mistakes (description length, dead workflow refs, name mismatch, inline bun -e)
4. **Link to full spec:** `See .claude/references/skill-spec.md for the complete specification.`

Total rule file body should be under 40 lines. It is a quick-reference card, not a duplicate of the spec.

## Files in Scope

| File | Action | Notes |
|------|--------|-------|
| `.claude/references/skill-spec.md` | create | Full spec document. Create the `references/` directory first. |
| `.claude/rules/skill-structure.md` | create | Auto-loading rule summary. Glob: `.claude/skills/**` |

## Interfaces to Implement

None. These are pure markdown reference documents with no TypeScript.

## Architectural Constraints

- **No TypeScript, no runtime behavior.** These files are read by humans and loaded into Claude Code context. They do not execute.
- **Rule file must be concise.** Under 40 lines of body content. It loads into context every time a skill file is edited -- bloated rules waste context.
- **Spec is the authority.** The rule file summarizes and links to the spec. If they conflict, the spec wins.
- **Create `.claude/references/` directory.** It does not exist yet (C-9). Just create it and put the spec file in it.
- **Tier definitions use file structure, not complexity labels.** Tier 1 / 2 / 3 based on what files exist, not subjective "simple/medium/complex."
- **`tracknames` legacy note.** The spec must document that `tracknames` uses a non-standard frontmatter format. New skills must NOT follow this pattern.
- **No `.md` extension in agent name.** When listing valid agent values, use the filename without extension (e.g., `suno`, not `suno.md`).

## Acceptance Criteria

- [ ] `.claude/references/` directory exists
- [ ] `.claude/references/skill-spec.md` exists and contains all 8 sections listed above
- [ ] Frontmatter fields table includes all 5 fields with correct types, required/recommended status, constraints, and defaults
- [ ] Tier definitions table has 3 tiers defined by file structure, with existing skill examples for each tier
- [ ] workflow.md conventions section documents TOC format, node ID pattern (`[PX-NNN]`), and node heading format
- [ ] Anti-patterns section lists at least 6 anti-patterns
- [ ] `.claude/rules/skill-structure.md` exists with `globs: [".claude/skills/**"]` frontmatter
- [ ] Rule file body is under 40 lines
- [ ] Rule file contains frontmatter quick-reference, tier guidance, common mistakes, and a link to the full spec
- [ ] Description length limit (250 chars) is documented with Claude Code truncation rationale in the spec
- [ ] Node ID regex is documented as `/\[([A-Z][0-9][-_][0-9]{3})\]/g`

## Out of Scope

- No TypeScript code. This task produces only markdown.
- No changes to existing skills. The spec documents what exists; it does not migrate or fix legacy skills.
- No changes to existing rules. Only the new `skill-structure.md` rule is created.
- No validator logic. That is Task 2.
- No scaffold logic. That is Task 3.

## Dependencies

| Depends On | Status |
|-----------|--------|
| None | This is the first task in the chain |

**Blocks:** Task 2 (validate-skill) -- the spec defines what the validator checks. Task 3 (create-skill) -- the spec defines what the scaffolder generates.
