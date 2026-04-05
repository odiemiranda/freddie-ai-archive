---
title: "Report Aggregator, Index, and CLI Entry Point"
type: task
project: songwriting-lyrics-improvement
status: done
created: 2026-04-03
author: mccall
architect: "vault/studio/projects/songwriting-lyrics-improvement/ARCHITECT.md"
group: 3
parallel: false
depends_on: ["OT-0017", "OT-0018", "OT-0019", "OT-0020"]
---

# OT-0021: Report Aggregator, Index, and CLI Entry Point

## What to Build

Three tightly coupled files that wire together the T1 tools into a usable system:

### 1. `src/libs/lyrics-analyzer/report.ts` -- Report Aggregator

Collects individual `ToolReport` results and produces a `UnifiedReport`:

```typescript
interface UnifiedReport {
  tools: ToolReport[];
  overall: "pass" | "warn" | "fail";
  language: LanguageProfile;
  sections: ParsedSection[];
  metadata: {
    linesAnalyzed: number;
    wordsAnalyzed: number;
    wordsNotFound: number;
    wordsApproximate: number;
    executionMs: number;
    toolsRun: string[];
  };
}
```

**Status rollup:**
- Any tool "fail" (high-severity flags) -> overall "fail"
- Any tool "warn" (medium, no high) -> overall "warn"
- All "pass" -> overall "pass"
- Individual tool errors (try/catch) -> that tool "fail" with error flag, others still run

**Formatted output:** Human-readable report with per-tool sections showing status,
key findings, and flag details. Similar to current `formatReport()` in
`src/tools/phonetics.ts` but structured for multiple tools.

### 2. `src/libs/lyrics-analyzer/index.ts` -- Barrel Export + Orchestrator

The `analyze()` function is the main entry point:

```typescript
interface AnalyzeOptions {
  tools?: string[];
  json?: boolean;
  genreTargets?: Record<string, unknown>;
  targetMeter?: "iambic" | "trochaic";
  prosodyStrictness?: number;
}

export async function analyze(
  lyrics: string,
  options?: AnalyzeOptions
): Promise<UnifiedReport>;
```

**Orchestration flow:**
1. `validateInput(lyrics)` -- reject invalid
2. `parseSections(sanitized)` -- split into sections
3. Tokenize all words: for each word, run contraction handler, language detection, phoneme lookup, syllable count, stress extraction -> build `WordToken[]`
4. Build `ToolInput` with sections (now containing `ParsedLine[]` with `WordToken[]`)
5. Run requested tools (default: all T1). Each tool wrapped in try/catch
6. Aggregate via `report.ts`
7. Return `UnifiedReport`

**Re-exports:** All individual tool functions for granular programmatic use:
```typescript
export { countSyllables } from "./tools/syllable-counter.js";
export { mapStress } from "./tools/stress-mapper.js";
export { detectRhymes } from "./tools/rhyme-detector.js";
export { analyzeBalance } from "./tools/section-balance.js";
```

### 3. `src/tools/lyrics-analyze.ts` -- CLI Entry Point + toolDef

Follow the `charcount.ts` pattern exactly. Dual-mode: CLI + programmatic via registry.

**Sub-commands:**
```
bun run tool lyrics-analyze syllable --text "..." [--json]
bun run tool lyrics-analyze stress --text "..." [--json]
bun run tool lyrics-analyze rhyme --file path.md [--json]
bun run tool lyrics-analyze balance --file path.md [--json]
bun run tool lyrics-analyze --file path.md [--json]       (all tools)
bun run tool lyrics-analyze --file path.md --tools syllable,stress [--json]
```

**toolDef:**
```typescript
export const toolDef: ToolDefinition = {
  name: "lyrics-analyze",
  description: "Analyze lyrics for syllable counts, stress patterns, rhyme quality, section balance, and more.",
  inputSchema: z.object({
    subcommand: z.string().optional(),
    text: z.string().optional(),
    file: z.string().optional(),
    json: z.boolean().optional(),
    tools: z.string().optional(),
  }),
  tags: ["creative", "quality", "lyrics"],
  execute: async (args) => { /* ... */ },
};
```

**CLI arg parsing:** Match existing pattern from `phonetics.ts` and `charcount.ts` --
`process.argv.slice(2)`, loop with `--flag value` pattern. First positional arg = subcommand.

## Acceptance Criteria

- [ ] AC-1: `bun run tool lyrics-analyze --text "Walking through the morning light"` returns unified report with all T1 tools
- [ ] AC-2: `bun run tool lyrics-analyze syllable --text "..."` returns only syllable counter output
- [ ] AC-3: `bun run tool lyrics-analyze --file out/rune-draft.md --json` returns JSON UnifiedReport
- [ ] AC-4: `bun run tool lyrics-analyze --file path.md --tools syllable,stress` runs only specified tools
- [ ] AC-5: Unified report includes language detection summary
- [ ] AC-6: Status rollup: tool with high flag -> overall "fail"
- [ ] AC-7: Individual tool error does not crash other tools. Error tool shows "fail" status
- [ ] AC-8: `toolDef` exported and functional for programmatic use
- [ ] AC-9: Execution time reported in metadata
- [ ] AC-10: Total execution < 500ms for ~50 line song
- [ ] AC-11: `import { analyze } from "src/libs/lyrics-analyzer"` works (barrel export)

Maps to PRD: US-10 (AC-10.1 through AC-10.8)

## Files in Scope

- `src/libs/lyrics-analyzer/report.ts` -- **create**
- `src/libs/lyrics-analyzer/index.ts` -- **create**
- `src/tools/lyrics-analyze.ts` -- **create**
- `src/libs/lyrics-analyzer/tools/*.ts` -- **read-only** (import)
- `src/libs/lyrics-analyzer/shared/*.ts` -- **read-only** (import)
- `src/registry/types.ts` -- **read-only** (import ToolDefinition)
- `src/tools/charcount.ts` -- **read-only reference** (pattern to follow)

## Architectural Constraints

- Follow `charcount.ts` dual-mode pattern exactly: `toolDef` export + `if (import.meta.main)` CLI
- Import `z` from "zod" for inputSchema
- Import `ToolDefinition` from `../registry/types.js`
- Import `readFileSync` from "fs" for `--file` handling
- Tool auto-discovery will find `lyrics-analyze.ts` in `src/tools/` automatically
- The tokenization step (building WordToken[]) happens ONCE in `analyze()`, NOT per-tool

## Out of Scope

- T2/T3 tools (added in OT-0023 through OT-0026)
- Tool runner alias (OT-0022)
- Deletion of old files (OT-0022)
- Workflow integration (Phase 3)

## Test Plan

```bash
# Full pipeline smoke test
bun run tool lyrics-analyze --text "[Verse 1]
Walking through the morning light
Shadows dancing left and right
Coffee stains on yesterday
Words I never meant to say"

# JSON output
bun run tool lyrics-analyze --text "Hello world" --json

# Sub-command
bun run tool lyrics-analyze syllable --text "Walking through the morning light"

# File mode
echo "[Verse 1]\nTest line one\nTest line two" > out/test-lyrics.md
bun run tool lyrics-analyze --file out/test-lyrics.md

# Performance
time bun run tool lyrics-analyze --file <50-line-lyrics-file>
```

## Notes

- This is the integration task. All four T1 tools must be complete before starting.
- The `analyze()` orchestrator is where the per-run word cache lives. Create a `Map<string, WordToken>` at the start, pass to each lookup. Same word = same result.
- Formatted output should be clean and scannable. Think: a report that a songwriter reads in 30 seconds and knows exactly what to fix. Follow the existing `phonetics.ts` `formatReport()` style but extended for multiple tools.