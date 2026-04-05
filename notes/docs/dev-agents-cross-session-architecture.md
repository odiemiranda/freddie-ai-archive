---
title: "Tech Approach: Dev Agents — Cross-Session Architecture"
date: 2026-03-29
status: planning
tags: [agents, dev, pm, architect, mcp, cross-session, brain-db]
---

# Tech Approach: Dev Agents — Cross-Session Architecture

## Vision

Expand freddie-ai with PM, Architect, and Dev agents that work from external Claude Code sessions via MCP. Agents accumulate knowledge back into our system — brain.db and vault memory — tagged by project, shared across agents, evolving over time.

External sessions own the code context. Our system owns the accumulated knowledge.

## Universal Save-on-Execution

**Every agent, every skill, every execution saves to memory.** This is not a dev-agent-only pattern — it's a system-wide architecture that applies to all agents (Tala, Rune, Sol, Echo, Ryan, McCall, Wick, Penny).

### Current State

Today, only some workflows call `reflect.ts` at the end — and only when the skill explicitly includes it (e.g., `/create-track` P5-003). If a skill doesn't have a reflect step, learnings are lost.

### Target State

Every skill execution — local or remote — produces a save event. The MCP server's background timers handle reflection and consolidation automatically.

| Agent | Skills | Saves on |
|-------|--------|----------|
| **Tala** | `/create-track`, `/create-track-variant` | Style decisions, critic feedback, user adjustments, BPM/key choices |
| **Rune** | `/lyrics` | Lyrical approach, phonetic fixes, volta placement, genre density |
| **Sol** | `/minimax-music` | Vocal choices, style block decisions, generation outcomes |
| **Echo** | `/gemini` | Critique patterns, disagreements between models |
| **Ryan** | `/create-brief`, `/create-prd` | Requirements patterns, user priority signals |
| **McCall** | `/architect`, `/code-review`, `/security-audit` | Architecture decisions, tech trade-offs, security findings |
| **Wick** | `/dev-start`→`/dev-end`, `/create-task` | Implementation patterns, error resolutions, commit context |
| **Penny** | `/daily`, `/meeting`, `/weekly` | Workflow patterns (lightweight — mostly operational) |

### How It Works

1. **Every skill ends with a save** — either explicit `agent_save` (remote) or `reflect.ts` call (local)
2. **MCP server timers** batch-process saves into reflections (5min) and consolidations (30min)
3. **brain.db** accumulates decisions across all agents and projects
4. **Knowledge evolves** — consolidation merges, deduplicates, resolves contradictions

### Migration Path

Existing skills that already call `reflect.ts` explicitly (create-track P5-003, lyrics workflow) continue as-is — those are local calls that run immediately.

New pattern: skills that run via MCP (remote) use `agent_save` for lightweight writes, and the server timers handle the rest.

Eventually, even local skills can switch to the save → timer pattern for consistency, but that's not required. Both paths feed the same inbox → knowledge → brain.db pipeline.

## Agents

| Agent | Persona | Model | Role | Soul? |
|-------|---------|-------|------|-------|
| **Ryan** (PM) | Product strategist | Claude Opus | Requirements, user stories, priorities | Yes |
| **McCall** (Architect) | System designer | Claude Opus | Plans, slices tasks, reviews code, merges | Yes |
| **Wick** (Dev) | Builder | Claude Sonnet | Implements from task docs, runs in worktree | Yes |
| **Council** | Multi-model panel | Gemini + Grok + MiniMax | Parallel analysis for architecture decisions | No (stateless) |

Freddie remains the orchestrator. No Scrum Master (Freddie handles orchestration). No QA (code review + security audit skills cover this).

**Model rationale:**
- **Opus for McCall/Ryan** — architecture decisions, planning, and code review need the strongest reasoning
- **Sonnet for Wick** — fast, capable, cheaper per-token for execution work. Code generation doesn't need Opus-level reasoning when the task doc is well-written.
- **Council models** — each model brings different strengths: Gemini (thorough analysis), Grok (contrarian angles), MiniMax (third perspective)

## Council Pattern

Multi-model analysis for architecture and review phases. Same principle as tri-critic for music — consensus across models catches blind spots.

### When Council Runs

| Phase | Council does | Output |
|-------|-------------|--------|
| `/dev-start` analysis | 3 parallel analyses of the task | Approach, risks, alternatives per model |
| Post-implementation review | 3 parallel code reviews | Architecture alignment, security, edge cases |

### Council Flow

```
McCall (Opus) dispatches council:
  ├── Gemini: architecture analysis, dependency check, risk assessment
  ├── Grok: alternative approaches, edge cases, "what could go wrong"
  └── MiniMax: third perspective, creative solutions

All 3 return structured analysis → McCall consolidates:
  ├── Reconciliation table (agree / disagree / unique per model)
  ├── Distill into approach doc
  └── Present to user
```

### Council API Calls

```bash
# All three run in parallel
bun src/tools/gemini.ts --system-file .claude/critics/architecture-analysis.md --input-file out/task-context.md --caller mccall
bun src/tools/minimax/chat.ts --system-file .claude/critics/architecture-analysis.md --input-file out/task-context.md --caller mccall
bun src/tools/grok.ts --system-file .claude/critics/architecture-analysis.md --input-file out/task-context.md --caller mccall
```

Cost per council: ~$0.02-0.05 (short analysis prompts, not full generation).

## Worktree Execution

### Why Worktrees

When McCall slices a task into parallel sub-tasks, each Wick agent needs its own copy of the repo to avoid file conflicts. Git worktrees provide this — isolated working directories sharing the same git history.

### How It Works

```
McCall slices task into 2 sub-tasks:
  ├── Slice A: auth middleware + JWT utils
  └── Slice B: login routes + tests

Spawn Wick agents:
  ├── Wick-1 (Sonnet, worktree A) → implements Slice A
  └── Wick-2 (Sonnet, worktree B) → implements Slice B

Both return completed work:
  ├── McCall reviews each slice (Opus)
  ├── Council reviews if complex (Gemini + Grok + MiniMax)
  ├── Merge worktrees back to main
  └── Run full test suite
```

### Worktree Rules

- **Each Wick agent gets `isolation: "worktree"`** when spawned via the Agent tool
- **McCall never codes** — it plans, reviews, and merges. Coding is Wick's job.
- **Task doc is the contract** — Wick cannot start without a task doc. The doc contains: what to build, acceptance criteria, files to touch, architectural constraints.
- **Single tasks run without worktree** — only parallel slices need isolation
- **Merge conflicts = McCall's problem** — if worktrees conflict, McCall resolves (Opus-level reasoning)

### When to Parallelize

| Condition | Action |
|-----------|--------|
| Task touches 1 module/feature | Single Wick, no worktree |
| Task touches 2+ independent modules | Parallel Wicks, worktrees |
| Task touches shared code (e.g., utils) | Sequential — Wick-1 first, Wick-2 after merge |
| User says "just do it" | McCall decides based on file overlap analysis |

## Future: MiniMax M2.7 for Coding via Bifrost

MiniMax's M2.7 coding model is reportedly Sonnet-class. MiniMax provides an **Anthropic-compatible API endpoint** (`https://api.minimax.io/anthropic`), which means native integration is possible without custom tool plumbing.

### Discovery: Bifrost LLM Gateway

**Source:** NotebookLM research on MiniMax + Claude Code integration.

Bifrost is an open-source LLM gateway that runs locally and acts as a transparent router. It allows different subagents to use different models while keeping the main session on Claude.

### How It Would Work

```
Claude Code (main session)
  │
  ANTHROPIC_BASE_URL → localhost:8080/anthropic (Bifrost)
  │
  ├── Freddie/McCall/Ryan → Bifrost routes to Claude API (default)
  └── Wick (model: minimax/MiniMax-M2.7) → Bifrost routes to MiniMax API
```

**Subagent config:**
```yaml
---
name: wick
model: minimax/MiniMax-M2.7
tools: [Bash, Read, Write, Edit, Grep, Glob]
---
```

Bifrost detects the `minimax/` prefix → routes to MiniMax API. Wick gets full Claude Code tool access (Read, Write, Edit, Bash, Grep) natively — no CLI wrapper needed.

### Two Integration Options

**Option A: Direct env var (simple, all-or-nothing)**
- Set `ANTHROPIC_BASE_URL` to MiniMax endpoint
- Set `ANTHROPIC_AUTH_TOKEN` to MiniMax API key
- All subagents use MiniMax — not ideal (we want Opus for McCall/Ryan)

**Option B: Bifrost gateway (recommended)**
- Bifrost runs locally on port 8080
- Main session + McCall/Ryan → Claude API (default)
- Wick → MiniMax API (via model prefix routing)
- Each provider configured in Bifrost with its own API key
- Transparent — subagents don't know about the routing

### What This Changes

| Without Bifrost | With Bifrost |
|----------------|-------------|
| Wick = Sonnet (Claude subscription) | Wick = MiniMax M2.7 (pay-per-token) |
| MiniMax coding = chat API wrapper | MiniMax coding = full tool access natively |
| One model provider | Multi-provider, per-agent routing |

### Evaluation Checklist

Before adopting:
1. **Bifrost setup on Windows** — `bunx` install, background service, start/stop scripts
2. **Bifrost config** — configure Claude API (default) + MiniMax API (minimax/ prefix) providers
3. **Model prefix fallback** — does Claude API error on unknown model prefix `minimax/MiniMax-M2.7`? If yes, skill needs detection logic to set model field dynamically
4. **MiniMax M2.7 tool use reliability** — does it handle Claude Code's tool call format correctly? (Read, Write, Edit, Bash, Grep)
5. **Context window** — M2.7's limit for coding (need 100K+ for large file context)
6. **Cost comparison** — M2.7 per-token vs Sonnet subscription
7. **Latency** — Bifrost routing overhead + MiniMax API response time
8. **Fallback** — if MiniMax is down or Bifrost not running, graceful degradation to Sonnet
9. **External session behavior** — verify that sessions without Bifrost ignore the model prefix or fall back cleanly
10. **npm run scripts** — add `bifrost:start`, `bifrost:stop`, `bifrost:status` to package.json (same pattern as MCP server)

### Current Decision

**Wick starts on Sonnet.** MiniMax M2.7 via Bifrost is the upgrade path once evaluated. The subagent architecture is the same either way — only the `model` field in the frontmatter changes.

## Document Storage

### Dual-Location Pattern

| Context | Path | Why |
|---------|------|-----|
| **Internal** (freddie-ai) | `vault/studio/projects/{project-name}/` | Vault is our knowledge hub, Obsidian-browsable |
| **External** (other repos) | `{project-root}/docs/` | Stays with the codebase, portable |

### Folder Structure

```
{root}/
├── tech/           ← architecture docs, tech approaches
├── tasks/          ← task docs for Wick
├── ARCHITECT.md    ← McCall's architecture decision record
├── PRD.md          ← Ryan's product requirements
└── BRIEF.md        ← Ryan's problem brief
```

For internal projects, `{root}` = `vault/studio/projects/{project-name}/`.
For external projects, `{root}` = `{project-root}/docs/`.

### File Naming

Top-level docs use fixed names: `BRIEF.md`, `PRD.md`, `ARCHITECT.md` — these are "current state" documents that get updated in place.

All other documents (tech approaches, task docs, etc.) use timestamped names:

```
{unix-timestamp-ms}-{document-slug}.md
```

Examples:
```
tech/1743321600000-auth-middleware-approach.md
tech/1743408000000-database-migration-strategy.md
tasks/1743321600000-implement-jwt-utils.md
tasks/1743321700000-build-login-routes.md
tasks/1743408000000-add-rate-limiting.md
```

This gives chronological ordering at a glance — sort by filename = sort by creation time. No ambiguity about which doc came first.

### Vault Snapshot (Option C)

External project's `docs/` is the **source of truth** during active work. When `/dev-end` runs, it snapshots the final state to `vault/studio/projects/{project-name}/` — clearly a point-in-time copy, not a living doc.

For internal (freddie-ai) projects, there's only one location — `vault/studio/projects/` — so no sync issue.

brain.db already captures decisions with `project` + `projectPath`, and reflect.ts captures learnings. The vault copy is for Obsidian browsability, not the authoritative source.

## Skills

### Phase 1 — Foundation
| Skill | Agent | Output |
|-------|-------|--------|
| `/create-brief` | Ryan | Problem statement, goals, constraints, success criteria |
| `/create-prd` | Ryan | Full PRD with user stories, acceptance criteria, priorities |
| `/architect` | McCall | Tech approach doc with decisions, trade-offs, component design |

### Phase 2 — Execution
| Skill | Agent | Output |
|-------|-------|--------|
| `/create-task` | Ryan → McCall | Scoped task with acceptance criteria, tech approach, ready for dev |
| `/dev-start` / `/dev-end` | Wick | Session-mode development from a task/spec |
| `/code-review` | McCall + Echo | Architecture alignment + quality review |
| `/security-audit` | McCall | OWASP top 10, dependency check, secrets scan |

## Cross-Session Architecture

### How It Works

```
External Session (project-foo)            freddie-ai System
┌──────────────────────────┐             ┌───────────────────────────┐
│ User runs /dev            │             │ vault/studio/memory/      │
│                           │             │   wick/inbox/             │
│ Full code context HERE    │   MCP       │   wick/knowledge/         │
│ File edits HERE           │ ─────────>  │   mccall/inbox/           │
│ Git operations HERE       │  (tools)    │   ryan/inbox/             │
│ Tests run HERE            │             │                           │
│                           │             │ brain.db                  │
│ Calls our MCP for:        │             │   decisions (project:foo) │
│   agent_save, brain       │             │   memories (project:foo)  │
│                           │             │                           │
└──────────────────────────┘             └───────────────────────────┘
```

### Context Isolation

| What | Where it lives | Crosses MCP? |
|------|---------------|--------------|
| Conversation context | External session | No |
| File reads/writes | External session | No |
| Git operations | External session | No |
| Agent memory (learnings) | freddie-ai vault | Yes — via remember/reflect |
| Architecture decisions | freddie-ai brain.db | Yes — via brain tools |
| Consolidation | freddie-ai vault | Yes — via consolidate tool |

### What External Sessions CAN Do (via MCP)

- Save learnings to agent inbox (`remember --save`)
- Query agent knowledge (`remember --query`)
- Log decisions to brain.db (`brain --decide`)
- Search past decisions (`brain --recall`)
- Run reflection (`reflect`)
- Trigger consolidation (`consolidate-memory`)
- Query brain.db (`brain --query`)

### What External Sessions CANNOT Do

- Read our conversation context
- Access our task list
- Modify our working files
- See other sessions' conversations
- Access tools not exposed via MCP

## Project Tagging

Every memory and decision gets tagged with its source project.

### How Project Is Identified

**Auto-detect from working directory (decided).** Every Claude Code session has a working directory. Extract project slug from the last path segment, store full path for disambiguation.

```
Working dir: D:\PROJECTS\test-suno
  → project: "test-suno"        (slug — for queries and grouping)
  → projectPath: "D:\PROJECTS\test-suno"  (full path — for disambiguation)
```

**Why this over env var or git remote:**
- Zero configuration — always available, no setup needed
- Works for non-git projects
- Full path handles duplicate folder names (`D:\PROJECTS\test-suno` vs `D:\PROJECTS-BAK\test-suno`)

**Resolution order:**
1. `process.cwd()` → extract last segment as slug, store full path
2. If `FREDDIE_PROJECT` env var is set → override slug (manual override for edge cases)

### Schema Changes

**brain.db decisions table** — add `project` and `project_path` columns:
```sql
ALTER TABLE decisions ADD COLUMN project TEXT DEFAULT 'unknown';
ALTER TABLE decisions ADD COLUMN project_path TEXT;
```

**Memory inbox files** — add both fields to frontmatter:
```yaml
---
source: reflection
agent: wick
project: test-suno
projectPath: "D:\\PROJECTS\\test-suno"
scope: self
confidence: high
tags: [drizzle, migrations, timestamps]
---
```

### Query Patterns

```bash
# What did we decide about auth in test-suno?
bun src/tools/brain.ts --recall --agent mccall --pattern "auth" --project test-suno

# Disambiguate if two projects share slug:
bun src/tools/brain.ts --recall --agent mccall --pattern "auth" --project-path "D:\PROJECTS\test-suno"

# How have we handled auth across ALL projects?
bun src/tools/brain.ts --recall --agent mccall --pattern "auth"

# What has Wick learned across all projects?
bun src/tools/brain.ts --query "patterns" --type decisions --agent wick

# What projects has Ryan worked on?
bun src/tools/brain.ts --query "" --type decisions --agent ryan
```

## Project as Entity — Similarity & Cross-Project Linking

Projects are registered as entities in brain.db (`type: "project"`). This reuses the entity + relations system we already built — no new tables needed.

### Auto-Registration

On first `agent_save` from a new working directory:

1. Extract project slug from `process.cwd()`
2. Upsert entity: `name: "test-suno", type: "project", summary: "D:\PROJECTS\test-suno"`
3. Fuzzy match existing project entities by name prefix/substring
4. If similar project found: auto-link with `similar_to` relation at weight 0.7

```
First save from D:\PROJECTS\test-suno-v2:
  → Entity "test-suno-v2" registered (type: project)
  → Similar project found: "test-suno" (prefix match)
  → Auto-linked: test-suno-v2 → similar_to → test-suno (weight: 0.7)
```

### Relation Types for Projects

| Relation | Meaning | Auto-detected? |
|----------|---------|---------------|
| `similar_to` | Related projects, shared context | Yes — name prefix match |
| `evolved_from` | New version of an older project | No — user confirms |
| `backup_of` | Backup/archive copy | No — user confirms |
| `depends_on` | Uses the other project as dependency/tool | No — user confirms |

Auto-detected relations start at weight 0.7 (uncertain). User-confirmed relations start at 1.0. Repeated cross-project learnings reinforce the weight.

### Cross-Project Knowledge Queries

When querying brain.db for a project, also surface knowledge from linked projects:

```bash
# Direct query — only test-suno-v2
bun src/tools/brain.ts --recall --agent wick --project test-suno-v2

# Expanded query — test-suno-v2 + all similar/related projects
bun src/tools/brain.ts --recall --agent wick --project test-suno-v2 --include-related

# Entity lookup — see all project relations
bun src/tools/brain.ts --entity test-suno-v2
# Returns:
#   test-suno-v2 → similar_to → test-suno (weight: 0.7)
#   test-suno-v2 → depends_on → freddie-ai (weight: 1.0)
```

### How This Helps Agents

- **Wick** starts `/dev-start` on `test-suno-v2` → queries brain.db → finds decisions from `test-suno` (similar project) → "last time on the original test-suno, we used Drizzle for migrations — same pattern applies here"
- **McCall** architects for `freddie-ai-plugin` → finds architecture decisions from `freddie-ai` (depends_on) → carries over patterns
- **Ryan** scopes a brief for a new project → finds similar past projects → reuses acceptance criteria patterns

### Similarity Detection Logic

```typescript
function findSimilarProjects(slug: string): EntityResult[] {
  // 1. Exact prefix match: "test-suno" matches "test-suno-v2", "test-suno-bak"
  // 2. Substring match: "suno" matches "test-suno", "suno-ai-tools"
  // 3. Only match type: "project" entities
  return searchEntities(slug, { type: "project" });
}
```

Lightweight — runs during first `agent_save`, not on every call.

## Shared Memory

Same pattern as music agents: `scope` field in frontmatter controls propagation.

```yaml
scope: self              # Only this agent sees it
scope: shared            # All agents see it
scope: mccall            # Specifically for the architect
sharedWith: [wick, mccall] # Multiple specific agents
```

Consolidation propagates shared-scope learnings to target agent inboxes, same as today.

### Cross-Domain Learning

Dev agents can learn from music agents and vice versa:
- Architect discovers a good API pattern → tagged `shared` → all agents benefit
- Music agent discovers a platform behavior → stays in music scope unless explicitly shared

Brain.db FTS5 doesn't care about domain — queries return relevant results across all agents and projects.

## Dev Session Mode (`/dev-start` → `/dev-end`)

### Concept

`/dev` is not a one-shot skill — it's a **session mode**. McCall (Architect, Opus) is the main agent orchestrating the session. Wick (Dev, Sonnet) sub-agents are spawned for coding tasks.

### Session Lifecycle

```
/dev-start
  ├── Query brain.db for recent decisions on this project
  ├── Load project index from vault/studio/projects/{project-name}/ (if exists)
  ├── List current to-do and in-progress tasks (via Penny)
  ├── Ask user: continue an existing task, or start a new one?
  ├── Mark chosen task as in-progress in Penny
  ├── Look for planning docs (docs/, PRD, tech approach, task doc)
  ├── If found: summarize context
  ├── If not found: ask user to describe the task
  ├── Flag stale tasks (in-progress 3+ days with no recent checkpoint)
  ├── Run council analysis (Gemini + Grok + MiniMax in parallel)
  ├── Consolidate into approach + reconciliation table
  ├── Present to user → approves/adjusts
  ├── Slice into task docs (if multiple sub-tasks)
  ├── SAVE: task intent, approach chosen, docs found
  └── Spawn Wick agent(s) to execute

During development (McCall orchestrates, Wick executes):
  ├── Wick works in worktree (if parallel) or main branch (if single)
  ├── On question/validation needed → McCall asks user → SAVE
  ├── On significant decision made → SAVE
  ├── On Wick returns work → McCall reviews (Opus) → SAVE
  ├── On git commit/push → SAVE
  ├── On error/blocker → SAVE
  └── User works normally — saves don't block

/dev-end (explicit — user types /dev-end)
  ├── Confirm task status with user: complete or pausing?
  ├── If complete → mark task "done" in Penny
  ├── If incomplete → keep task "in-progress", add Penny note: "Paused — {reason}"
  ├── McCall runs council review on completed work (if significant)
  ├── Merge worktrees if parallel
  ├── Update tech doc / planning doc in session
  ├── SAVE: final session summary
  ├── Flush: immediate reflect + consolidate
  ├── Snapshot docs/ to vault/studio/projects/{project-name}/ (if external project)
  ├── Log to daily note: "Worked on {project}: {summary}"
  └── Print summary

/dev-end (auto — session ends without explicit /dev-end)
  ├── Hook detects session ending
  ├── Lightweight flow only:
  │   ├── Save checkpoint via agent_save
  │   ├── Add Penny note: "Session ended — auto-paused"
  │   ├── Log to daily note: "{project}: session ended, task still in-progress"
  │   └── Skip: reflect, vault snapshot, council review (too heavy for teardown)
  └── Task stays "in-progress" — /dev-start will flag it next session
```

### Saves Are Cheap, Reflects Are Batched

**Key insight:** External sessions only SAVE (write raw events to inbox). Reflection and consolidation happen on our MCP server via background timers — not in the external session.

```
External Session                      freddie-ai MCP Server
                                      (always running)
/dev-start ──agent_save──>              inbox/wick/raw-log-001.md
  coding...
  decision ──agent_save──>              inbox/wick/raw-log-002.md
  coding...
  commit ──agent_save──>                inbox/wick/raw-log-003.md
  coding...                                      │
                                      ┌─── 5min timer fires ────┐
                                      │ 3 raw logs found        │
                                      │ → 1 Gemini reflect call │
                                      │ → learnings to inbox/   │
                                      │ → decisions to brain.db │
                                      └─────────────────────────┘
  more coding...
  question ──agent_save──>              inbox/wick/raw-log-004.md
  approval ──agent_save──>              inbox/wick/raw-log-005.md
                                                 │
                                      ┌─── 30min timer fires ──┐
                                      │ inbox has 5+ items      │
                                      │ → consolidate-memory    │
                                      │ → inbox → knowledge     │
                                      └─────────────────────────┘

/dev-end ──agent_save (end)──>          Detected "end" event
                                      → immediate flush: reflect + consolidate
                                      → penny task update
                                      → brain.db sync
```

### Why This Architecture

| Approach | API calls/session | Blocks dev? | Learning quality |
|----------|-------------------|-------------|------------------|
| Reflect on every checkpoint | 10-15 Gemini calls | Yes | Fragmented |
| Reflect only at /dev-end | 1 call | No | Misses mid-session context |
| **Debounced timers (this)** | **2-4 calls** | **No** | **Batched, contextual** |

### The `agent_save` MCP Tool

Lightweight write-only tool. No API calls. Instant return.

```typescript
{
  name: "agent_save",
  params: {
    agent: "wick",           // which agent
    project: "project-foo",  // project tag
    event: "decision",       // decision | question | commit | error | start | end
    content: "Chose Drizzle over Prisma because..."
  }
}
```

Writes a timestamped markdown file to `memory/{agent}/inbox/raw-log-{timestamp}.md`:

```yaml
---
source: dev-session
agent: wick
project: project-foo
event: decision
session: 2026-03-29T19:30:00
---

Chose Drizzle over Prisma because the project already uses bun:sqlite
and Drizzle's query builder aligns with the existing pattern.
```

### MCP Server Background Timers

The http MCP server (`src/servers/freddie-ai/`) already runs as a persistent background process. Add two intervals:

```typescript
// Every 5 minutes: batch-reflect pending raw logs
setInterval(() => reflectIfNeeded(), 5 * 60 * 1000);

// Every 30 minutes: consolidate if inbox threshold met
setInterval(() => consolidateIfNeeded(), 30 * 60 * 1000);
```

**`reflectIfNeeded()`:**
1. Scan `memory/*/inbox/` for `raw-log-*.md` files
2. If none → skip
3. Group by agent
4. Per agent: concatenate raw logs into one trace, call `reflect()` once
5. Archive processed raw logs to `memory/{agent}/inbox/processed/`

**`consolidateIfNeeded()`:**
1. Check inbox counts per agent (excluding raw-log files already processed)
2. If any agent has 3+ reflection-generated items → run consolidation
3. Sync brain.db

**`/dev-end` flush:**
When a `agent_save` arrives with `event: "end"`:
1. Skip the timer — run reflect immediately with all pending raw logs
2. Run consolidation if threshold met
3. Update penny task via MCP
4. Sync brain.db

### Timer vs Hook Placement

Timers live in the **MCP server**, not in Claude Code hooks. Hooks fire on Claude Code events (tool use, session start) — these timers need to fire independently, even when no one is talking to our local session.

The MCP http server is always running → correct place for background processing.

## Silent Memory Operations

### Problem
When `/dev-start` and checkpoints run from an external session, the `agent_save` MCP call would show as a visible tool call.

### Solution: Subagent Containment

Save operations dispatch inside a subagent. The external session sees:

```
[Agent: dev-checkpoint] Saving checkpoint...
→ "Checkpoint saved."
```

The `agent_save` MCP call is **hidden inside the subagent context**. User sees intent + result, not plumbing.

For `/dev-end`, the subagent handles the final save:

```
[Agent: dev-session-close] Closing dev session...
→ "Session closed. 2 learnings saved, 1 decision logged. Penny task updated."
```

### Permissions

External sessions need MCP tool access. Recommended setup:
```json
{
  "permissions": {
    "allow": ["mcp__freddie-ai_*"]
  }
}
```

With `dontAsk` or allowed permissions, MCP calls proceed without prompts.

## MCP Transport Exposure

### Current (music agents)
| Transport | Exposes |
|-----------|---------|
| stdio | All tools (penny + registry) |
| http (3456) | Penny + time only |

### Proposed (with dev agents)
| Transport | Exposes |
|-----------|---------|
| stdio | All tools (penny + registry + dev memory) |
| http (3456) | Penny + time + dev agent memory tools |

Dev agent tools exposed on http:
- `remember` (with project tag support)
- `reflect` (with project tag support)
- `brain` (query, decide, recall — with project filter)
- `consolidate-memory` (with agent filter)

## Design Principles (from BMAD + GSD, adapted)

### From BMAD
- **Role separation** — PM owns requirements, Architect owns design, Dev owns implementation
- **Context in handoff documents** — each agent writes structured output, next agent reads it
- **Human-in-the-loop** — user confirms at phase boundaries (our phase-gate pattern)

### From GSD
- **Fresh context per task** — subagent dispatch with only relevant context (our existing pattern)
- **Spec-driven execution** — dev agent works from specs, not vibes
- **Must-haves as verification** — mechanically checkable outcomes per task
- **State on disk** — decisions in brain.db, learnings in vault (our existing pattern)

### Our additions
- **Soul/persona system** — agents have identity and philosophy
- **brain.db with confidence decay** — decisions age, contradictions detected
- **Cross-project learning** — agents evolve across projects via tagged memory
- **Tri-critic pattern** — multi-model quality validation (for code review)

## Implementation Plan

### Phase 0 — Infrastructure (Freddie builds, user reviews)
1. Add `project` + `project_path` fields to `reflect.ts`, `remember.ts`, `logDecision()`
2. Add `project_path` column to brain.db decisions table (project column already added)
3. Build `agent_save` MCP tool (lightweight write-only, project-tagged)
4. Add background timers to MCP http server (`reflectIfNeeded`, `consolidateIfNeeded`)
5. Add flush handler (immediate reflect on `event: "end"`)
6. Expose `agent_save`, `remember`, `reflect`, `brain`, `consolidate-memory` on http transport
7. Create memory directories: `memory/ryan/`, `memory/mccall/`, `memory/wick/`
8. Test save → timer → reflect → consolidate cycle

### Phase 1 — Agent Identity (Co-created with user)

**Soul files — user shapes personality, voice, philosophy:**
1. Draft Ryan soul (`souls/ryan.md`) — the analyst. How he thinks, communicates, assesses risk.
2. Draft McCall soul (`souls/mccall.md`) — the professional. His precision, contingency thinking, calm.
3. Draft Wick soul (`souls/wick.md`) — the executor. His terseness, autonomy, problem-solving style.

**Agent files — user designs workflow, tools, rules:**
4. Design Ryan agent (`agents/pm.md`) — tools, hard rules, how he interacts with McCall
5. Design McCall agent (`agents/architect.md`) — tools, council dispatch, review approach, how he directs Wick
6. Design Wick agent (`agents/dev.md`) — tools, worktree rules, task doc contract, how he reports back

### Phase 2 — Skills & Workflows (Co-designed with user)

**Planning skills:**
1. Design `/create-brief` skill + workflow — phases, gates, output format
2. Design `/create-prd` skill + workflow — phases, gates, output format
3. Design `/architect` skill + workflow — council pattern, reconciliation, output format
4. Design `/create-task` skill + workflow — Ryan scopes, McCall adds tech approach

**Execution skills:**
5. Design `/dev-start` → `/dev-end` skill + workflow — session lifecycle, checkpoints, Wick dispatch
6. Build architecture-analysis critic: `.claude/critics/architecture-analysis.md`
7. Build council dispatch pattern (parallel Gemini + Grok + MiniMax analysis)
8. Build council reconciliation (merge 3 analyses into approach doc)
9. Build Wick worktree execution (Agent tool with `isolation: "worktree"`, `model: "sonnet"`)
10. Build task slicing logic in McCall (parallel vs sequential decision)
11. Build dev-checkpoint subagent pattern (silent saves)

### Phase 3 — Quality (Co-designed with user)
1. Design `/code-review` skill + workflow — council-powered, architecture + security + alternative
2. Design `/security-audit` skill + workflow — OWASP checklist, dependency scanning
3. Build and test from external session via MCP

### Phase 4 — Bifrost Gateway (Deferred)
1. Research Bifrost: package name, bunx compatibility, Windows support
2. Install and configure: Claude API (default) + MiniMax API (minimax/ prefix)
3. Test routing, fallback, external session behavior
4. Add npm run scripts: `bifrost:start`, `bifrost:stop`, `bifrost:status`
5. Evaluate MiniMax M2.7 as Wick replacement for Sonnet
6. Document setup

## Open Questions

- Should `/create-task` be one monolithic skill or a pipeline of `/create-brief` → `/create-prd` → `/architect` → `/dev-start`?
- How granular should project tagging be? Project-level? Feature-level? Both?
- Should dev agents have their own critics (code-quality critic, security critic)?
- Rate limiting on http MCP — should we throttle external session memory writes?
- Do we need a `/dev:setup` skill that configures the external session's MCP connection to us?
- Timer intervals: 5min reflect / 30min consolidate — too frequent? Too slow? Should they be configurable per project?
- Should raw logs be pruned after processing, or kept in `processed/` for audit trail?
- If the MCP server restarts mid-session, how do we recover pending raw logs? (They're on disk — timers just restart and pick them up.)
- Should `/dev-start` query brain.db for past decisions on the same project before presenting approach?
- Council: should all 3 models always run, or should McCall decide based on task complexity? Simple tasks may not need a council.
- Worktree merge: automatic (if no conflicts) or always user-reviewed?
- Should Wick agents run tests in their worktree before returning to McCall?
- MiniMax M2.7 evaluation: when to test as Wick replacement for Sonnet?
