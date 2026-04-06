---
title: "AAR-0002: Autonomous Consolidation"
type: task
project: autonomous-agent-research
project-code: AAR
task-id: AAR-0002
priority: P1
status: pending
effort: low
created: 2026-04-07
author: mccall
depends-on: [AAR-0001]
blocks: [AAR-0003, AAR-0004, AAR-0005]
files-modified: [src/libs/reflect.ts, src/tools/consolidate-memory.ts]
files-created: []
---

# AAR-0002: Autonomous Consolidation

## Objective

Remove the 3-item inbox threshold so consolidation runs whenever ANY inbox items exist. Wire consolidation into the session-end flow (the `/bye` skill) so it runs automatically after reflection completes. Add an `--autonomous` flag to the consolidation tool that bypasses the threshold check.

## Context

**Current behavior:** `shouldConsolidate(agent, threshold=3)` in `src/libs/reflect.ts` only returns `true` when an agent has 3+ inbox items. The `/bye` skill calls `consolidate-memory --auto` which also uses this 3-item threshold. This means learnings can sit in inbox for multiple sessions before consolidation.

**Target behavior:** After AAR-0001, only importance 7+ learnings reach the inbox — the noise is already filtered. The threshold is now counterproductive because it delays consolidation of high-value learnings. Consolidation should run automatically after every session that produces inbox items.

**Why this is safe:** With importance scoring (AAR-0001), the inbox only contains learnings the system has already judged as significant. Running consolidation on 1-2 items is fine — Gemini handles small inputs gracefully, and the cost is one API call per session.

## File-by-File Spec

### 1. `src/libs/reflect.ts`

#### `shouldConsolidate()` changes

Add an `autonomous` parameter:

```typescript
/**
 * Check if an agent's inbox has enough items to trigger consolidation.
 *
 * @param agent - Agent name
 * @param options.threshold - Item count threshold (default: 3, ignored if autonomous)
 * @param options.autonomous - If true, any inbox items trigger consolidation (default: false)
 */
export function shouldConsolidate(
  agent: string,
  options?: { threshold?: number; autonomous?: boolean }
): boolean {
  const { threshold = 3, autonomous = false } = options ?? {};
  const inboxDir = join(MEMORY_ROOT, agent, "inbox");
  if (!existsSync(inboxDir)) return false;
  const count = readdirSync(inboxDir).filter(f => f.endsWith(".md")).length;
  return autonomous ? count > 0 : count >= threshold;
}
```

**Breaking change note:** The current signature is `shouldConsolidate(agent: string, threshold = 3): boolean`. The reflect tool (`src/tools/reflect.ts` line 59) calls it as `shouldConsolidate(args.agent)` with no second argument. The consolidate-memory tool does NOT call this function — it checks inbox count directly. So the signature change is backwards-compatible because:
- Callers passing no second arg get `{ threshold: 3, autonomous: false }` — same as before.
- Callers passing a number will break — but grep confirms no callers pass a number. Only `args.agent` is passed.

Actually, to be safe and unambiguous, keep the old signature working too:

```typescript
export function shouldConsolidate(
  agent: string,
  thresholdOrOpts?: number | { threshold?: number; autonomous?: boolean }
): boolean {
  let threshold = 3;
  let autonomous = false;

  if (typeof thresholdOrOpts === "number") {
    threshold = thresholdOrOpts;
  } else if (thresholdOrOpts) {
    threshold = thresholdOrOpts.threshold ?? 3;
    autonomous = thresholdOrOpts.autonomous ?? false;
  }

  const inboxDir = join(MEMORY_ROOT, agent, "inbox");
  if (!existsSync(inboxDir)) return false;
  const count = readdirSync(inboxDir).filter(f => f.endsWith(".md")).length;
  return autonomous ? count > 0 : count >= threshold;
}
```

This keeps backward compatibility with any caller that might pass a raw number.

### 2. `src/tools/consolidate-memory.ts`

#### Add `--autonomous` flag to CLI

Add the flag to the CLI arg parser (around line 491):

```typescript
let autonomous = false;

// In the arg parsing loop:
else if (args[i] === "--autonomous") autonomous = true;
```

#### Auto mode changes

In the auto-consolidation section (around line 515-547), respect the `--autonomous` flag:

```typescript
if (autoMode) {
  const needsWork: string[] = [];
  for (const a of AGENTS) {
    const inboxDir = join(memoryRoot, a, "inbox");
    const count = countMd(inboxDir);
    // When autonomous, any inbox items qualify; otherwise use threshold
    if (autonomous ? count > 0 : count >= threshold) needsWork.push(a);
  }

  if (needsWork.length === 0) {
    const msg = autonomous
      ? "No agents have inbox items. Nothing to consolidate."
      : `No agents have ${threshold}+ inbox items. Nothing to consolidate.`;
    console.log(msg);
    process.exit(0);
  }

  console.log(`=== AUTO-CONSOLIDATION (${autonomous ? "autonomous" : `threshold: ${threshold}`}) ===`);
  // ... rest unchanged
}
```

#### Tool definition changes

Add `autonomous` to the inputSchema:

```typescript
export const toolDef: ToolDefinition = {
  name: "consolidate_memory",
  description: "Reconcile agent memory inbox into clean knowledge files. ...",
  inputSchema: z.object({
    agent: z.string().optional().describe("Agent name. Required unless list=true."),
    reason: z.string().optional().describe("Short label for archive folder (default: 'consolidation')"),
    dry_run: z.boolean().optional().describe("Show plan without writing files"),
    list: z.boolean().optional().describe("Show inbox/knowledge counts for all agents"),
    autonomous: z.boolean().optional().describe("Run with threshold=0 (any inbox items qualify)"),
  }),
  // ...
  execute: async (args) => {
    // When autonomous, pass threshold=0 or just check count > 0
    // (auto mode is not directly exposed via toolDef — only single-agent mode)
    // For the tool API, autonomous just means "run even with <3 items"
  },
};
```

#### Usage string update

```
Usage: bun run tool consolidate-memory --agent {agent} [--reason label] [--dry-run]
       bun run tool consolidate-memory --auto [--threshold N] [--autonomous] [--dry-run]
       bun run tool consolidate-memory --list
```

### 3. Workflow Integration (documentation only)

The `/bye` skill (`.claude/skills/bye/SKILL.md`) currently runs:

```bash
bun run tool consolidate-memory --auto
```

After this task, the autonomous pipeline should run:

```bash
bun run tool consolidate-memory --auto --autonomous
```

**Do NOT modify the skill file in this task.** The skill file update is a follow-up wiring step after all AAR Phase 1 tasks are complete. Document this in a `## Wiring Notes` section at the bottom of this task doc.

## Acceptance Criteria

1. **`shouldConsolidate()` supports autonomous mode.** `shouldConsolidate("tala", { autonomous: true })` returns `true` when tala has 1 inbox item.
2. **Backward compatibility.** `shouldConsolidate("tala")` with no second arg still uses threshold=3.
3. **`--autonomous` flag works.** `bun run tool consolidate-memory --auto --autonomous` processes agents with any inbox items (not just 3+).
4. **Tool definition updated.** The `autonomous` field exists in the tool's inputSchema.
5. **No behavioral change without the flag.** Default `--auto` still uses the 3-item threshold.

## Constraints

- Do NOT modify `.claude/skills/bye/SKILL.md` — that is a wiring step after all Phase 1 tasks complete.
- Do NOT change the Gemini prompt or consolidation logic in `consolidateAgent()` — only the threshold/trigger logic.
- The `--autonomous` flag is additive — it does not remove or replace `--threshold`.

## Wiring Notes (for post-task integration)

After all AAR Phase 1 tasks are complete, the `/bye` skill workflow should be updated:

```
Phase 2 — Sync & Consolidate:
  bun run tool brain --check
  bun run tool consolidate-memory --auto --autonomous
```

This means every session that produces importance 7+ learnings will automatically consolidate them into knowledge files.

## Out of Scope

- Modifying the consolidation Gemini prompt
- Changing the archive/knowledge file format
- Wiring into the `/bye` or `/dev-end` skills (post-task integration)
