# TASK: Build create-skill CLI Tool and Skill

> Owner: McCall | Assigned: Ryan | Source: [[ARCHITECTURE]]

## What to Build

Two artifacts:

1. **`src/tools/create-skill.ts`** -- A CLI tool that scaffolds new Claude Code skill directories with valid structure based on the tier system.
2. **`.claude/skills/create-skill/SKILL.md`** -- The `/create-skill` slash command that Freddie uses to orchestrate the interactive skill creation flow. This is a Tier 2 skill (SKILL.md only -- no workflow.md needed because the flow is simple enough for a single file).

### CLI Tool (create-skill.ts)

The tool generates skill directories under `.claude/skills/`. It writes valid frontmatter, structural headings, and (for Tier 2/3) a workflow.md skeleton. It calls `validate-skill.ts` on the output to verify correctness.

**CLI Interface:**

```
bun src/tools/create-skill.ts --name my-skill --description "Short description" --tier 2
bun src/tools/create-skill.ts --name my-skill --description "Short description" --tier 2 --agent suno
bun src/tools/create-skill.ts --name my-skill --description "Short description" --tier 3 --dry-run
```

**Arguments:**

| Arg | Required | Type | Default | Notes |
|-----|----------|------|---------|-------|
| `--name` | Yes | string | -- | Skill name. Validated against `/^[a-z][a-z0-9-]{0,63}$/` |
| `--description` | Yes | string | -- | Skill description. 1-250 chars. |
| `--tier` | Yes | `1 \| 2 \| 3` | -- | Determines which files are generated |
| `--agent` | No | string | `none` | Agent name. Must be `none` or match a file in `.claude/agents/` |
| `--dry-run` | No | flag | false | Print what would be created without writing files |
| `--force` | No | flag | false | Overwrite existing skill directory |

**Exit codes:** `0` = success, `1` = error (validation failure, collision, filesystem error).

### Scaffold Output by Tier

**All tiers** create `.claude/skills/{name}/SKILL.md` with this frontmatter:

```yaml
---
name: {name}
description: {description}
user-invocable: true
argument-hint: ""
agent: {agent}
---
```

**Tier 1** -- SKILL.md only:

```
.claude/skills/{name}/
└── SKILL.md
```

SKILL.md body (below frontmatter):

```markdown
# /{name}

$ARGUMENTS

## Instructions

<!-- Skill instructions go here -->
```

**Tier 2** -- SKILL.md + workflow.md:

```
.claude/skills/{name}/
├── SKILL.md
└── workflow.md
```

SKILL.md body:

```markdown
# /{name}

$ARGUMENTS

## Input Collection

<!-- How to collect and validate user input -->

## Execution Model

| Phase | Runs in | Purpose |
|-------|---------|---------|
| 1 | | |
| 2 | | |

## Workflow

Follow the mermaid diagram below. Each node `[PX-NNN]` maps to a section in `workflow.md` with full instructions.

` ` `mermaid
graph TD
    INPUT[User input] --> P1

    subgraph P1["Phase 1"]
        P1_001[P1-001: Step 1]
    end

    subgraph P2["Phase 2"]
        P2_001[P2-001: Step 1]
    end

    P1 --> P2 --> DONE[Done]
` ` `

## Output

<!-- What the skill produces -->
```

(Note: the triple backtick fences in the actual output must be real markdown fences, not escaped. The escaped form above is only to avoid breaking this task doc.)

workflow.md body:

```markdown
<!-- TOC
| Node | Line | Title | Phase |
|------|------|-------|-------|
| P1-001 | 14  | Step 1 | 1 |
| P2-001 | 28  | Step 1 | 2 |
TOC -->

# {name} Workflow -- Node Instructions

---

## [P1-001] Step 1

> **Runs in:** TBD

**Input:**
**Output:**

**Next:** --> P2-001

---

## [P2-001] Step 1

> **Runs in:** TBD

**Input:**
**Output:**
```

**Tier 3** -- SKILL.md + workflow.md + subdirectories:

```
.claude/skills/{name}/
├── SKILL.md
├── workflow.md
├── assets/
└── scripts/
```

Same as Tier 2, plus empty `assets/` and `scripts/` directories. Both directories are created with a `.gitkeep` file so git tracks them.

### Scaffold Content Policy

- **Skeleton only.** Valid frontmatter, structural headings, empty section bodies.
- **No placeholder prose.** No "TODO: fill in your workflow here." The heading structure communicates what goes where.
- **Mermaid skeleton for Tier 2/3.** A minimal graph with placeholder node names (P1-001, P2-001). Not a real diagram -- just the structure the author fills in.
- **workflow.md skeleton for Tier 2/3.** TOC comment block, node headings matching the mermaid skeleton, empty Input/Output/Next fields.

### Tool Flow

```
1. Parse CLI arguments
2. Validate name format (regex)
3. Check for name collision (directory already exists in .claude/skills/)
   - If collision and no --force: exit 1 with message
   - If collision and --force: proceed (overwrite)
4. If --agent is not "none", verify it exists in .claude/agents/
5. Validate description length (1-250 chars)
6. If --dry-run: print plan (files that would be created) and exit 0
7. Create directory: .claude/skills/{name}/
8. Write SKILL.md
9. If tier 2 or 3: write workflow.md
10. If tier 3: create assets/ and scripts/ with .gitkeep files
11. Call validator: bun src/tools/validate-skill.ts {name}
12. Print results (files created + validation output)
13. Exit 0 if validator passed, exit 1 if validator found errors
```

### The /create-skill Skill

This SKILL.md follows the meta-skill pattern established by `create-brief`, `create-prd`, and `create-task`. Freddie orchestrates the interactive flow. No sub-agent dispatch needed -- Freddie collects inputs and calls the CLI tool.

**Flow:**

1. Freddie asks the user to describe what skill they want to create (or reads `$ARGUMENTS`)
2. Freddie asks tier selection, presenting the three options with one-line descriptions:
   - **Tier 1:** Single-file skill -- instructions fit in one file, no multi-step workflow
   - **Tier 2:** Multi-phase skill -- separate workflow.md with step-by-step node instructions
   - **Tier 3:** Multi-phase with resources -- Tier 2 plus assets/ and/or scripts/ directories
3. Freddie derives the name (lowercase, hyphens) and description (under 250 chars) from the conversation
4. Freddie presents the plan for confirmation:
   ```
   Name: {name}
   Description: {description}
   Tier: {tier}
   Agent: {agent}
   Files: [list of files that will be created]
   ```
5. User confirms or adjusts
6. Freddie runs: `bun src/tools/create-skill.ts --name {name} --description "{desc}" --tier {tier}`
7. Freddie reports the result and reminds the user to fill in the skill content

## Files in Scope

| File | Action | Notes |
|------|--------|-------|
| `src/tools/create-skill.ts` | create | CLI scaffolding tool |
| `.claude/skills/create-skill/SKILL.md` | create | The `/create-skill` slash command |

## Interfaces to Implement

```typescript
interface SkillConfig {
  name: string;
  description: string;
  tier: 1 | 2 | 3;
  agent: string;           // default: "none"
  userInvocable: boolean;  // default: true
  argumentHint: string;    // default: ""
}

interface ScaffoldResult {
  created: string[];       // list of files/dirs created (relative paths)
  skipped: string[];       // list of files that already existed (only relevant with --force)
}
```

Internal functions (not exported, but this is the expected structure):

```typescript
function validateName(name: string, skillsDir: string): { valid: boolean; error?: string }
function generateSkillMd(config: SkillConfig): string
function generateWorkflowMd(config: SkillConfig): string
function scaffoldSkill(config: SkillConfig): ScaffoldResult
```

## Architectural Constraints

- **Single file for the CLI tool (C-1).** All logic in `src/tools/create-skill.ts`. No library directory.
- **CLI-first (C-1).** This is a `bun src/tools/...` CLI tool. No MCP.
- **Confirm before generating (C-3).** The SKILL.md skill file must instruct Freddie to present the plan and wait for user approval before calling the CLI tool. The CLI tool itself does NOT prompt -- it assumes the caller has confirmed.
- **No inline bun -e (C-4).** The SKILL.md calls the CLI tool via `bun src/tools/create-skill.ts`, never inline code.
- **Writes to .claude/skills/ (C-5).** Not to `out/`. The skill directory is the final destination -- there is no draft step for scaffolded files.
- **Idempotency.** Refuses to overwrite existing directory unless `--force` is passed. Reports "skill '{name}' already exists at .claude/skills/{name}/" and exits 1.
- **Calls validator via CLI subprocess.** `create-skill.ts` calls `bun src/tools/validate-skill.ts {name}` after writing files. It does NOT import `validateSkill`. This keeps both tools independently deployable.
- **Skeleton only (ADR-5).** No placeholder prose. Structural headings with empty bodies. The mermaid diagram and workflow.md get minimal node skeletons, not real content.
- **No sub-agent dispatch in the SKILL.md.** This is a Freddie-only skill. Freddie collects inputs and runs the CLI tool. No Ryan/McCall dispatch.

## Acceptance Criteria

- [ ] `bun src/tools/create-skill.ts --name test-skill --description "A test skill" --tier 1` creates `.claude/skills/test-skill/SKILL.md` with correct frontmatter
- [ ] `bun src/tools/create-skill.ts --name test-skill --description "A test skill" --tier 2` creates both `SKILL.md` and `workflow.md`
- [ ] `bun src/tools/create-skill.ts --name test-skill --description "A test skill" --tier 3` creates `SKILL.md`, `workflow.md`, `assets/.gitkeep`, and `scripts/.gitkeep`
- [ ] Scaffolded SKILL.md has valid frontmatter with all 5 fields (`name`, `description`, `user-invocable`, `argument-hint`, `agent`)
- [ ] Scaffolded SKILL.md body has structural headings with no placeholder prose
- [ ] Tier 2/3 scaffolded SKILL.md contains a mermaid code block with placeholder nodes
- [ ] Tier 2/3 scaffolded workflow.md has a TOC comment block and at least two `## [PX-NNN]` node headings
- [ ] `--dry-run` prints the plan without creating any files
- [ ] Running against an existing skill name without `--force` exits 1 with collision message
- [ ] Running against an existing skill name with `--force` overwrites the directory
- [ ] `--agent bogus-name` exits 1 with "unknown agent" error (if `bogus-name.md` doesn't exist in `.claude/agents/`)
- [ ] `--description` longer than 250 chars exits 1 with length error
- [ ] `--name "Invalid Name"` exits 1 with format error
- [ ] After scaffold, the tool calls `bun src/tools/validate-skill.ts {name}` and reports the results
- [ ] The scaffolded skill passes validation with 0 errors (warnings are acceptable)
- [ ] `.claude/skills/create-skill/SKILL.md` exists with the `/create-skill` slash command following the meta-skill pattern
- [ ] The SKILL.md for create-skill includes tier selection guidance and the confirmation-before-execution flow
- [ ] **Cleanup:** After running acceptance tests, delete any test skill directories created (e.g., `.claude/skills/test-skill/`)

## Out of Scope

- No `workflow.md` for the create-skill skill itself. The interactive flow is simple enough for a single SKILL.md.
- No auto-detection of tier from user intent. Explicit user selection only (ADR-4).
- No `--annotated` flag for comment-style hints. Deferred to v2.
- No tool registry integration. No `toolDef` export.
- No sub-agent dispatch from the skill. Freddie handles it directly.
- No changes to existing skills or the validator.
- No `--template` flag for custom templates. Scaffold uses hardcoded tier structures.

## Dependencies

| Depends On | Status |
|-----------|--------|
| Task 1 (Skill Spec + Rule File) | Must complete first -- spec defines the structural conventions the scaffold must produce |
| Task 2 (validate-skill CLI) | Must complete first -- scaffold calls the validator as a post-creation verification step |

**Blocks:** Nothing. This is the final task in the chain.
