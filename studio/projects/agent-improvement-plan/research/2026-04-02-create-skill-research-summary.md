---
source: notebooklm + web
notebook: bfbe27dc
topic: "Create-Skill Research Summary"
date: 2026-04-02
---

# Create-Skill Research Summary

## Sources Analyzed
- **Skill Forge** (AgriciDaniel/skill-forge) — full lifecycle: plan, build, audit, benchmark, publish
- **Build Skill CLI** (Flash-Brew-Digital/build-skill) — monorepo scaffolding with CI/CD
- **Agent Skill Creator** (FrancyJGLisboa/agent-skill-creator) — messy input → structured skill, 14+ platforms
- **Claude Code official docs** (code.claude.com/docs/en/skills) — spec, frontmatter, discovery
- **awesome-claude-code** (hesreallyhim) — community patterns
- Our existing skills (dev-start, create-track, architect) as reference

## Key Findings

### What create-skill should generate (minimum)
1. `SKILL.md` — lightweight entry point with frontmatter (name, description, user-invocable, argument-hint)
2. `workflow.md` — phase/node execution logic (our existing pattern)
3. Separate files, NOT inline — keeps SKILL.md under 500 lines, avoids context bloat

### Input design
- **Declarative intent**: user provides natural language description of what the skill should do
- **Auto-detect**: complexity tier, required agents, file structure from intent
- **Confirm before generating**: Freddie's "measure twice" — present plan, user approves

### Validation (before creating)
- Name: lowercase, hyphens, 1-64 chars
- Collision: check `.claude/skills/` and `.claude/commands/` for existing names
- Required fields: name and description minimum
- Directory existence: don't overwrite without --force

### Validation (after creating)
- Spec validation: frontmatter format, internal links resolve
- Registration check: skill appears in autocomplete

### Anti-patterns to avoid
- Description over 250 chars (gets truncated — Claude won't trigger)
- Inline everything in SKILL.md (context bloat)
- Dead rules: paths globs that don't match actual files
- Cross-platform adapters (we're Claude Code only)
- Offline benchmarking (tri-critic already handles quality)
- Team registries (single user)

### Supporting tools (minimum viable)
1. **Spec validator** — standalone, runs on all 20+ skills, checks frontmatter + dead rules
2. **Staleness checker** — review tracking (last_reviewed), dependency health, schema drift
3. **NOT needed**: benchmarking pipelines, cross-platform installers, team registries

### Decision: what we're building
- `create-skill` CLI tool — takes intent, scaffolds SKILL.md + workflow.md
- `validate-skill` CLI tool — validates any skill against spec
- Both as standalone tools in `src/tools/`
- The `/create-skill` skill itself uses these tools in its workflow
