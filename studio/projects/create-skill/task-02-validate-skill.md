# TASK: Build validate-skill CLI Tool

> Owner: McCall | Assigned: Ryan | Source: [[ARCHITECTURE]]

## What to Build

A single-file CLI tool at `src/tools/validate-skill.ts` that validates Claude Code skill directories against the structural conventions defined in the skill spec (Task 1). The tool reads files from `.claude/skills/`, runs a suite of 14 checks, and reports errors, warnings, and info findings.

The validator does NOT read the spec file at runtime. All rules are encoded as check functions inside the tool. The spec is the human reference; the validator is the machine enforcement.

### CLI Interface

```
bun src/tools/validate-skill.ts {skill-name}     # validate one skill
bun src/tools/validate-skill.ts --all             # validate all skills
bun src/tools/validate-skill.ts {name} --json     # JSON output for one skill
bun src/tools/validate-skill.ts --all --json      # JSON output for all skills
```

### Exit Codes

- `0` -- no errors (warnings and info are allowed)
- `1` -- one or more errors found, OR filesystem/argument error

### Output Formats

**Human (default):** Grouped by skill, with ANSI color-coded severity. Example shape:

```
morning
  PASS  No issues found

tracknames
  WARN  frontmatter.user-invocable-missing — SKILL.md — user-invocable field absent. Skill may not appear in / autocomplete. Add user-invocable: true to frontmatter.
  WARN  frontmatter.argument-hint-missing — SKILL.md — argument-hint field absent. Add argument-hint: "" for discoverability.
  INFO  frontmatter.agent-missing — SKILL.md — agent field absent. Recommended: add agent: none.

Summary: 26 skills scanned, 0 errors, 4 warnings, 2 info
```

**JSON (`--json`):** Array of `ValidationResult` objects (see Interfaces section).

**Summary table for `--all`:** After per-skill details, print a summary line with total skills scanned, total errors, total warnings, total info.

## Files in Scope

| File | Action | Notes |
|------|--------|-------|
| `src/tools/validate-skill.ts` | create | Single-file CLI tool. All logic in this file. |

## Interfaces to Implement

```typescript
interface ValidationResult {
  skill: string;
  errors: ValidationFinding[];
  warnings: ValidationFinding[];
  info: ValidationFinding[];
}

interface ValidationFinding {
  check: string;           // e.g., "frontmatter.description-length"
  message: string;         // human-readable: what's wrong + how to fix
  file: string;            // "SKILL.md" or "workflow.md"
  line?: number;           // line number if applicable
}

// Internal: return type for frontmatter parsing
interface FrontmatterResult {
  valid: boolean;
  fields: Record<string, unknown>;
  raw: string;
  errors: string[];        // parse-level errors (malformed YAML)
}
```

Additionally, export a function for programmatic use by `create-skill.ts`:

```typescript
export function validateSkill(skillDir: string): ValidationResult
```

This function takes an absolute path to a skill directory and returns a `ValidationResult`. The CLI entry point calls this function. `create-skill.ts` may also call it (via CLI subprocess or direct import).

### Internal Structure

```
validate-skill.ts
├── CLI argument parsing (positional name, --all, --json)
├── validateSkill(skillDir: string): ValidationResult
│   ├── parseFrontmatter(content: string): FrontmatterResult
│   ├── check functions (see Check Inventory below)
│   └── aggregate results into ValidationResult
├── formatHuman(results: ValidationResult[]): string
├── formatJson(results: ValidationResult[]): string
└── main() — CLI entry point
```

## Check Inventory

These are the exact checks to implement, with their IDs, severities, and behavior.

| # | Check ID | Severity | What It Catches | Implementation |
|---|----------|----------|----------------|----------------|
| 1 | `frontmatter.required-fields` | Error | Missing `name` or `description` in frontmatter | Check parsed frontmatter has `name` (string) and `description` (string, non-empty). If frontmatter itself is malformed YAML, report that as error and skip remaining frontmatter checks. |
| 2 | `frontmatter.name-format` | Error | Name not lowercase, has invalid chars, or > 64 chars | Regex: `/^[a-z][a-z0-9-]{0,63}$/`. Must start with lowercase letter. Only lowercase letters, digits, hyphens. 1-64 chars total. |
| 3 | `frontmatter.name-matches-dir` | Error | `name` field value does not match the skill directory name | Compare `frontmatter.name` to the basename of the skill directory (e.g., `path.basename(skillDir)`). |
| 4 | `frontmatter.description-length` | Error | Description > 250 chars | `description.length > 250`. Report actual length in the message. |
| 5 | `frontmatter.description-empty` | Error | Description is empty string or whitespace-only | `description.trim().length === 0`. |
| 6 | `frontmatter.user-invocable-type` | Error | `user-invocable` is present but not a boolean | Check `typeof fields['user-invocable'] !== 'boolean'`. Only fire if the field exists. |
| 7 | `naming.collision` | Error | Another skill directory uses the same name | Scan all directories in `.claude/skills/`. If more than one directory resolves to the same `name` frontmatter value, flag collision. For single-skill validation, check if the name matches any OTHER directory's basename. |
| 8 | `structure.workflow-ref` | Error | SKILL.md text references workflow.md but the file doesn't exist | Case-insensitive search for "workflow.md" in SKILL.md content. If found and no `workflow.md` file exists in the skill directory, error. |
| 9 | `structure.dead-subdirs` | Warning | SKILL.md references assets/ or scripts/ but directory doesn't exist | Search SKILL.md + workflow.md (if exists) for text references to `assets/` or `scripts/`. If referenced but the subdirectory doesn't exist, warn. |
| 10 | `structure.internal-links` | Warning | `[PX-NNN]` IDs in SKILL.md without matching headers in workflow.md | Extract IDs from SKILL.md using regex `/\[([A-Z][0-9][-_][0-9]{3})\]/g`. For each ID, check if workflow.md contains a heading with that ID (e.g., `## [P1-001]`). If no workflow.md exists, skip this check. If SKILL.md has no IDs, skip. |
| 11 | `structure.mermaid-exists` | Warning | Skill has workflow.md but no mermaid block in SKILL.md | If workflow.md exists, check SKILL.md for a fenced code block with language `mermaid`. Absence = warning. |
| 12 | `frontmatter.agent-known` | Warning | `agent` field value is not `none` and not a known agent | List files in `.claude/agents/`, strip `.md` extension to get valid agent names. If `agent` field exists, is not `none`, and does not match any known agent filename, warn. |
| 13 | `frontmatter.user-invocable-missing` | Warning | `user-invocable` field is absent from frontmatter | Fire only if the field is completely absent (not if it's present with a value). |
| 14 | `frontmatter.argument-hint-missing` | Info | `argument-hint` field is absent from frontmatter | Informational only. Skill works but loses autocomplete hint. |

**Check execution order:** Parse frontmatter first. If frontmatter is malformed YAML, report the parse error and skip all frontmatter.* checks (checks 1-6, 12-14). Structure checks (8-11) and naming checks (7) can still run.

**Important edge cases:**
- `tracknames` uses `allowed_tools` instead of `user-invocable`/`argument-hint`. The validator will correctly warn about missing `user-invocable` and report info about missing `argument-hint`. This is expected behavior, not a bug.
- Check #10 (internal links) must use a flexible regex that handles both `P1-001` and `S1-001` formats, as well as both hyphen and underscore separators.
- Check #7 (collision) in single-skill mode: compare the skill's `name` against the basenames of all OTHER skill directories. In `--all` mode: collect all names and flag duplicates.

## Architectural Constraints

- **Single file.** All logic in `src/tools/validate-skill.ts`. No library directory, no shared modules.
- **CLI-first (C-1).** This is a `bun src/tools/...` CLI tool. No MCP registration.
- **No external YAML parser dependency.** Parse frontmatter manually -- extract content between `---` delimiters and parse the simple key-value YAML. Frontmatter in skills is flat key-value pairs (no nested objects, no arrays except `allowed_tools` in the legacy `tracknames` case). Use a simple line-by-line parser or `yaml` from bun's built-in capabilities. If `gray-matter` is already in the project's dependencies, it can be used. Otherwise, manual parsing.
- **Exit codes are contracts.** `0` = no errors. `1` = errors or runtime failure. Scripts and `create-skill.ts` depend on these exit codes.
- **All paths relative to project root.** The tool resolves `.claude/skills/` and `.claude/agents/` relative to the project root. Use `import.meta.dir` or equivalent to locate the project root.
- **Error messages must include remediation.** Every finding message tells the user what's wrong AND how to fix it (e.g., "Description is 267 chars, exceeds 250 char limit. Shorten the description.").
- **Do not read the spec file at runtime.** Rules are hardcoded in the check functions. The spec is for humans.

## Acceptance Criteria

- [ ] `bun src/tools/validate-skill.ts morning` runs without error and reports results for the `morning` skill
- [ ] `bun src/tools/validate-skill.ts --all` scans all 26 skills in `.claude/skills/` and prints a summary line
- [ ] `bun src/tools/validate-skill.ts --all` exits 0 if no errors found across all skills (warnings are fine)
- [ ] `bun src/tools/validate-skill.ts nonexistent-skill` exits 1 with a clear error message
- [ ] `bun src/tools/validate-skill.ts morning --json` outputs valid JSON matching the `ValidationResult` interface
- [ ] `bun src/tools/validate-skill.ts --all --json` outputs a JSON array of `ValidationResult` objects
- [ ] All 14 checks from the Check Inventory are implemented
- [ ] `tracknames` correctly receives warnings for `user-invocable-missing` and info for `argument-hint-missing` (not errors)
- [ ] A skill with `description` > 250 chars would produce an error (verify by testing with a synthetic case or by confirming the check logic)
- [ ] Check #8 (workflow-ref) correctly identifies skills that reference workflow.md in their text but don't have the file (if any exist)
- [ ] Check #10 (internal-links) uses flexible regex `/\[([A-Z][0-9][-_][0-9]{3})\]/g` to extract IDs
- [ ] Check #12 (agent-known) correctly reads `.claude/agents/` directory to build the valid agent list
- [ ] Human output uses ANSI colors for severity (red=error, yellow=warning, blue/cyan=info, green=pass)
- [ ] Summary line format: `Summary: N skills scanned, X errors, Y warnings, Z info`
- [ ] The `validateSkill` function is exported for programmatic use

## Out of Scope

- No auto-fix mode. The validator reports only; it does not modify files.
- No unit test framework. The `--all` flag running against 26 existing skills IS the integration test.
- No staleness detection (`last_reviewed` tracking). Deferred to v2.
- No prose scanning for path-like strings. Only check explicit structural references (workflow.md, assets/, scripts/).
- No changes to existing skills. If the validator finds issues in existing skills, that's expected output -- do not fix the skills.
- No `--fix` flag. Report-only for v1.
- No MCP registration or tool registry integration.

## Dependencies

| Depends On | Status |
|-----------|--------|
| Task 1 (Skill Spec + Rule File) | Must complete first -- spec defines the rules this tool enforces |

**Blocks:** Task 3 (create-skill) -- the scaffolder calls this validator after creating files.

**Note on the dependency:** Task 1 produces the spec document, but the validator does NOT read the spec at runtime. The dependency is conceptual -- Ryan needs to reference the spec to know what rules to encode. If the spec content is clear from THIS task doc (which it is -- the Check Inventory above is complete), Ryan can technically start this task in parallel with Task 1. However, sequential execution is recommended to ensure the spec is reviewed and finalized before encoding its rules.
