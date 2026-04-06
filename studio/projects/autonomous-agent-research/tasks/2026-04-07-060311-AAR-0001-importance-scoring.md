---
title: "AAR-0001: Importance Scoring in reflect.ts"
type: task
project: autonomous-agent-research
project-code: AAR
task-id: AAR-0001
priority: P1
status: pending
effort: low
created: 2026-04-07
author: mccall
depends-on: []
blocks: [AAR-0002]
files-modified: [src/libs/reflect.ts, src/tools/reflect.ts]
files-created: []
---

# AAR-0001: Importance Scoring in reflect.ts

## Objective

Add an importance score (1-10) to each learning produced by the reflection engine. Only learnings scoring 7+ are written to the agent's inbox. Lower-scoring learnings are still logged to brain.db decisions table for recall but do not trigger the downstream pipeline.

This is the noise filter at the top of the autonomous pipeline. Without it, every minor observation floods the consolidation stage.

## Context

**Current behavior:** `reflect()` in `src/libs/reflect.ts` sends all learnings from Gemini to the agent's inbox. There is no filtering — every observation, regardless of significance, creates an inbox file.

**Target behavior:** Gemini assigns an importance score per learning. Only score 7+ learnings become inbox files. All learnings (regardless of score) are logged to brain.db decisions table with the score as metadata, so they remain searchable via `--recall-all`.

## File-by-File Spec

### 1. `src/libs/reflect.ts`

#### Type changes

```typescript
// ADD importance field to Learning interface (line ~38)
export interface Learning {
  observation: string;
  learning: string;
  evidence: string;
  scope: LearningScope;
  confidence: Confidence;
  memoryType: MemoryType;
  tags: string[];
  importance: number;  // NEW: 1-10, assigned by Gemini
}
```

#### Prompt changes — `buildReflectionPrompt()`

Add importance scoring instructions to the JSON output section of the prompt. Insert after the existing `"tags"` field in the output format:

```
"importance": 1-10 (how significant is this learning for future work?)

Importance scale:
- 1-3: Trivial — confirms what's already known, or too specific to generalize
- 4-6: Useful — valid observation but not urgent to remember
- 7-8: Important — changes how the agent should approach similar future work
- 9-10: Critical — contradicts existing knowledge, reveals a platform truth, or prevents a recurring failure
```

Also update the "Rules" section at the end of the prompt:

```
- Learnings: 1-5 with honest importance scores. Don't inflate — a session with all 3s is fine.
```

Note: The current prompt says "1-3 high-quality." Change this to "1-5 with honest importance scores" because importance filtering replaces the implicit quality filter. Gemini should report more observations and let the score decide what enters the pipeline.

#### `parseReflectionOutput()` changes

Update the filter in the `learnings` mapping to include `importance`:

```typescript
// In the parsed.learnings mapping (line ~279-282):
const learnings = (parsed.learnings || []).map((l: any) => ({
  ...l,
  memoryType: l.memoryType || "semantic",
  importance: typeof l.importance === "number" ? l.importance : 5, // default to 5 if missing
})).filter(
  (l: any) => l.observation && l.learning && l.scope && l.confidence,
) as Learning[];
```

Do NOT filter by importance here. Filtering happens in the `reflect()` function so that low-importance learnings can still be logged to decisions.

Apply the same default to the backwards-compat array path (line ~293-296).

#### `reflect()` function changes

After parsing, split learnings into two groups:

```typescript
// After parseReflectionOutput() returns (around line 209):
const { learnings: allLearnings, decisions } = parseReflectionOutput(text);

// Split by importance threshold
const IMPORTANCE_THRESHOLD = 7;
const highImportance = allLearnings.filter(l => l.importance >= IMPORTANCE_THRESHOLD);
const lowImportance = allLearnings.filter(l => l.importance < IMPORTANCE_THRESHOLD);

// Write only high-importance learnings to inbox
const filesWritten = highImportance.length > 0
  ? writeLearnings(input, highImportance, project, projectPath)
  : [];

// Log ALL learnings (including low-importance) to brain.db decisions
// Low-importance learnings still have value for recall — they just don't trigger consolidation
```

For the low-importance learnings, log them to brain.db decisions table:

```typescript
// Inside the try block that logs decisions (around line 224):
try {
  initBrain();

  // Log explicit decisions
  if (decisions.length > 0) {
    for (const d of decisions) {
      logDecision({ /* ...existing code... */ });
    }
  }

  // NEW: Log low-importance learnings as decisions (for recall, not pipeline)
  for (const l of lowImportance) {
    logDecision({
      agent: input.agent,
      decision: `[low-importance observation] ${l.learning}`,
      context: `Observation: ${l.observation}. Evidence: ${l.evidence}. Importance: ${l.importance}/10`,
      alternatives: undefined,
      tags: l.tags.join(", "),
      project,
      projectPath,
      rawRef: input.rawRef,
    });
  }

  await syncMemories();
} catch {
  // brain.db failure shouldn't break reflection
}
```

#### `renderLearning()` changes

Add importance to the frontmatter output:

```typescript
// In the frontmatter section of renderLearning() (around line 359-372):
// Add after the confidence line:
importance: ${learning.importance}
```

#### Return value changes

Update the `ReflectionResult` to include importance breakdown:

```typescript
// Update ReflectionResult interface (line ~55):
export interface ReflectionResult {
  learnings: Learning[];         // only high-importance (7+)
  allLearnings: Learning[];      // all learnings including low-importance
  decisions: Decision[];
  filesWritten: string[];
  skipped: boolean;
  reason?: string;
  usageReport?: string;
}
```

Update the return statements in `reflect()` to include `allLearnings`:

- The `NO_LEARNINGS` early return: `allLearnings: []`
- The parse failure early return: `allLearnings: []`
- The main return: `learnings: highImportance, allLearnings: allLearnings`

### 2. `src/tools/reflect.ts`

#### Output format changes

Update `runReflect()` to include importance info in the JSON output:

```typescript
const output: any = {
  learnings: result.learnings.length,
  allLearnings: result.allLearnings.length,
  lowImportanceLogged: result.allLearnings.length - result.learnings.length,
  decisions: result.decisions.length,
  filesWritten: result.filesWritten,
  skipped: result.skipped,
  consolidationRecommended: consolidationNeeded,
};
```

No other changes needed in the tool — the tool is a thin CLI wrapper.

## Acceptance Criteria

1. **Gemini returns importance scores.** Every learning in the parsed JSON has an `importance` field with a value 1-10.
2. **Only 7+ learnings reach inbox.** Learnings with importance < 7 do NOT create inbox files.
3. **All learnings logged to brain.db.** Low-importance learnings are logged as decisions with `[low-importance observation]` prefix, searchable via FTS5.
4. **Importance in frontmatter.** Inbox files include `importance: N` in their YAML frontmatter.
5. **Backwards compatibility.** If Gemini returns a learning without an importance field, it defaults to 5 (below threshold — conservative).
6. **Tool output includes breakdown.** `bun run tool reflect` output shows `allLearnings`, `learnings` (high-importance), and `lowImportanceLogged` counts.
7. **Existing tests/behavior unchanged.** The `shouldConsolidate()` function is not modified in this task.

## Constraints

- Do NOT change the `shouldConsolidate()` threshold — that is AAR-0002.
- Do NOT modify `consolidate-memory.ts` — that is AAR-0002.
- The importance threshold (7) should be a named constant (`IMPORTANCE_THRESHOLD`) at module scope, not a magic number.
- Keep the backwards-compatible array parsing path working.

## Out of Scope

- Configurable importance thresholds (hardcoded to 7 is fine for now)
- UI for reviewing low-importance learnings
- Any changes to consolidation behavior
