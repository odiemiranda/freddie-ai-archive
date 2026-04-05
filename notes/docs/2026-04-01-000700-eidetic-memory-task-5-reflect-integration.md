---
title: "Eidetic Memory Task 5 — reflect.ts Integration"
type: task
status: pending
parent: 2026-04-01-000700-eidetic-memory-raw-layer
task: 5
depends: [1]
created: 2026-04-01
author: mccall
---

# Task 5 — reflect.ts Integration (raw_ref threading)

## Objective

Thread the `rawRef` timestamp through `reflect.ts` so every reflected observation and logged decision traces back to the exact raw context file. Four touch points as specified in the tech approach doc (R4).

## What to Build

### 1. Add `rawRef` to `ReflectionInput` (`src/libs/reflect.ts`)

```typescript
export interface ReflectionInput {
  agent: string;
  task: string;
  workflow: string;
  outcome: "success" | "partial" | "failed" | "rejected";
  trace: string;
  userFeedback?: string;
  startedAt?: string;
  project?: string;
  projectPath?: string;
  rawRef?: number;           // <-- NEW: unix timestamp of raw context file
}
```

### 2. Thread through `writeLearnings()` (`src/libs/reflect.ts`)

In the `renderLearning()` function (line ~350), add `raw_ref` to the frontmatter output when present:

```typescript
function renderLearning(input: ReflectionInput, learning: Learning, now: ..., contradicts: string[] = [], project = "unknown", projectPath = ""): string {
  const rawRefLine = input.rawRef ? `\nraw_ref: ${input.rawRef}` : "";
  // ... existing code ...

  return `---
source: reflection
agent: ${input.agent}
project: ${project}
...existing fields...
session: ${now.format("YYYY-MM-DDTHH:mm:ss")}${contradictsLine}${rawRefLine}
---
...
`;
}
```

### 3. Thread through `logDecision()` calls (`src/libs/reflect.ts`)

In the `reflect()` function, where decisions are logged (line ~222-233), pass `rawRef`:

```typescript
// Current:
logDecision({
  agent: input.agent,
  decision: d.decision,
  context: d.context,
  alternatives: d.alternatives,
  tags: d.tags.join(", "),
  project,
  projectPath,
});

// Updated:
logDecision({
  agent: input.agent,
  decision: d.decision,
  context: d.context,
  alternatives: d.alternatives,
  tags: d.tags.join(", "),
  project,
  projectPath,
  rawRef: input.rawRef,        // <-- NEW
});
```

This requires Task 1 to have already added `rawRef` to `DecisionInput` and `logDecision()`.

### 4. Optional: pass raw context to reflection prompt

In `buildReflectionPrompt()`, if `rawRef` is provided, add a note to the prompt:

```typescript
${input.rawRef ? `- **Raw context:** memory/raw/${input.rawRef}.md (full session transcript available)` : ""}
```

This is informational only — we are NOT loading the full raw file into the reflection prompt (that would blow up token limits). The reflection engine uses the `trace` field for context. The raw file is for human and search recall, not for the reflection LLM.

## Files to Modify

| File | Change |
|------|--------|
| `src/libs/reflect.ts` | Add `rawRef` to `ReflectionInput`, thread through `renderLearning()`, `logDecision()`, and `buildReflectionPrompt()` |

## Dependencies

- Task 1 must be complete (DecisionInput.rawRef + logDecision() persistence)

## Acceptance Criteria

1. `ReflectionInput` accepts optional `rawRef` field
2. When `rawRef` is provided, inbox observation files include `raw_ref: {timestamp}` in frontmatter
3. When `rawRef` is provided, `logDecision()` is called with `rawRef` value
4. When `rawRef` is NOT provided, behavior is identical to current (no raw_ref in output)
5. Reflection prompt mentions raw context file when `rawRef` is present
6. No token budget increase — raw file content is NOT loaded into reflection prompt

## Test Criteria

Call `reflect()` with a `rawRef` value and verify:

```bash
bun -e "
  import { reflect } from './src/libs/reflect.js';
  const result = await reflect({
    agent: 'tala',
    task: 'test raw ref threading',
    workflow: 'test',
    outcome: 'success',
    trace: 'Built a jazz track. Used brush drums. Echo approved.',
    rawRef: 1743408000,
  });
  console.log('Files:', result.filesWritten);
  console.log('Skipped:', result.skipped);
"
```

Then inspect the written inbox file and verify `raw_ref: 1743408000` appears in frontmatter. Check brain.db decisions table for the `raw_ref` column value.
