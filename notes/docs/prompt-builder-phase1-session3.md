# Phase 1 Session 3: CLI + Workflow Integration

**Project:** Prompt Builder Engine
**Phase:** 1 of 5 (Suno Style Builder)
**Session:** 3 of 3
**Estimated:** 2-3 hours
**Depends on:** Session 2 complete (`index.ts`, `strategies/suno.ts` exist, all tests pass)

---

## What to Build

Four things:

1. **CLI entry point** -- `prompt-builder.ts` tool with `toolDef` export
2. **Tool registry integration** -- add to registry discovery
3. **Workflow doc updates** -- update P3-002 and P3-003 in create-track workflow to use the builder
4. **Tala agent file update** -- add builder instructions to suno.md
5. **Integration tests** -- CLI end-to-end and validate-prompt subsumption proof

---

## Files to Create/Modify

| File | Action | Purpose |
|------|--------|---------|
| `src/tools/prompt-builder.ts` | **CREATE** | CLI entry point + toolDef export |
| `src/tools/prompt-builder.test.ts` | **CREATE** | Integration tests |
| `.claude/skills/create-track/workflow.md` | **MODIFY** | Update P3-002 and P3-003 to use builder |
| `.claude/agents/suno.md` | **MODIFY** | Add builder tool + instructions |

---

## TypeScript Implementation Contract

### src/tools/prompt-builder.ts

```typescript
import { z } from "zod";
import { parseArgs } from "util";
import { readFileSync } from "fs";
import { buildStyle, StyleInputSchema, type StyleOutput } from "../libs/prompt-builder/index.js";
import type { ToolDefinition, ToolResult } from "../registry/types.js";

// ============================================================
// CLI
// ============================================================

/**
 * CLI interface:
 *   bun src/tools/prompt-builder.ts --build style --input out/style-input.json
 *   bun src/tools/prompt-builder.ts --build style --input-json '{"platform":"suno",...}'
 *
 * Also supports individual flags for quick CLI usage:
 *   bun src/tools/prompt-builder.ts --build style \
 *     --platform suno --type bgm \
 *     --genres "lo-fi, jazz" \
 *     --lead "shamisen" --lead-behavior "melodic plucks" \
 *     --support "harp" --support-constraint "low-register pad chords, no melody" \
 *     --rhythm "brushed drums" --rhythm-constraint "sparse" \
 *     --key "G Major" --bpm 88 \
 *     --moods "warm, lazy" \
 *     --textures "vinyl crackle"
 */

// Parse args, build input object, call buildStyle(), format output.
// Two input modes:
//   1. --input <path> or --input-json <json-string>: pass raw JSON to buildStyle()
//   2. Individual flags: assemble StyleInput from flags, pass to buildStyle()

// Output format (stdout):
//   Line 1: the Style block text (ready to copy)
//   Line 2: blank
//   Line 3+: --- Validation ---
//   Chars: N (target: 75-120)  OK/WARNING/ERROR
//   Elements: N (target: 3-7)  OK/WARNING
//   Trimmed: none / list of trimmed slots
//   Conflicts: none / list
//   Warnings: none / bulleted list
//   Errors: none / bulleted list

// Exit codes:
//   0 = success (may have warnings)
//   1 = errors (validation failures or char hard max exceeded)

// ============================================================
// toolDef (Registry)
// ============================================================

export const toolDef: ToolDefinition = {
  name: "prompt_builder",
  description:
    "Deterministic Style block builder for Suno and MiniMax. " +
    "Takes structured JSON input (genres, instruments, moods, key, BPM) " +
    "and produces a validated Style block with conflict checks, " +
    "genre normalization, vibe mapping, and auto-trim. " +
    "Returns the Style text plus diagnostics.",
  inputSchema: z.object({
    build: z.enum(["style"]).describe("What to build. Currently only 'style' is supported."),
    input: z.string().optional().describe("Path to JSON input file"),
    inputJson: z.string().optional().describe("Raw JSON string input (alternative to file path)"),
  }),
  tags: ["creative", "suno", "minimax", "deterministic"],
  async execute(args) {
    // Read input from file or inline JSON
    // Call buildStyle()
    // Format as text output
    // Return { output, isError }
  },
};
```

**CLI flag-to-StyleInput mapping:**

| CLI Flag | StyleInput Field | Parsing |
|----------|-----------------|---------|
| `--platform` | `platform` | string |
| `--type` | `type` | string |
| `--genres` | `genres` | split on `, ` |
| `--lead` | `instruments.lead.name` | string |
| `--lead-behavior` | `instruments.lead.behavior` | string |
| `--support` | `instruments.support.name` | string |
| `--support-constraint` | `instruments.support.constraint` | string |
| `--rhythm` | `instruments.rhythm.name` | string |
| `--rhythm-constraint` | `instruments.rhythm.constraint` | string |
| `--vocal-style` | `vocal.style` | string |
| `--vocal-technique` | `vocal.technique` | string |
| `--key` | `key` | string |
| `--bpm` | `bpm` | parseInt |
| `--moods` | `moods` | split on `, ` |
| `--textures` | `textures` | split on `, ` |
| `--chords` | `chords` | string |
| `--production` | `production` | split on `, ` |
| `--realism` | `realism` | parseInt |

---

## Workflow Doc Changes

### P3-002 (Build Style block) -- MODIFY

Current P3-002 (line ~238 in workflow.md) has Tala manually building the Style block. Replace with builder call.

**New P3-002 content:**

```markdown
## [P3-002] Build Style block

> **Runs in:** Tala sub-agent

**Input:** Locked ingredients from Phase 2
**Output:** Style block string (~75-120 chars) + diagnostics

Write the StyleInput JSON to `out/style-input.json`:

```json
{
  "platform": "suno",
  "type": "bgm",
  "genres": ["lo-fi", "jazz"],
  "instruments": {
    "lead": { "name": "shamisen", "behavior": "melodic plucks" },
    "support": { "name": "harp", "constraint": "low-register pad chords, no melody" },
    "rhythm": { "name": "brushed drums", "constraint": "sparse" }
  },
  "key": "G Major",
  "bpm": 88,
  "moods": ["warm", "lazy"],
  "textures": ["vinyl crackle"]
}
```

Run the builder:
```bash
bun src/tools/prompt-builder.ts --build style --input out/style-input.json
```

**Review the output:**
- If errors: fix the input JSON and re-run
- If warnings: review each. Mood conflicts and BPM range warnings should be resolved. Vibe translations and genre normalizations are informational.
- The Style block text is ready to use -- but Tala MAY override specific elements if creative intent requires it. Log any overrides for reflect.

**Override rules:**
- Tala can modify the output text, but must note what was changed and why
- Do NOT re-run the builder after manual edits -- the builder is advisory
- If the edit changes the char count significantly, re-validate with charcount tool

**Next:** -> P3-003
```

### P3-003 (Charcount validation) -- MODIFY

Current P3-003 runs charcount as a separate step. The builder already includes char count in its output. Simplify to a verification step.

**New P3-003 content:**

```markdown
## [P3-003] Charcount validation

> **Runs in:** Tala sub-agent

**Input:** Style block from P3-002
**Output:** Verified char count

The builder output already includes `charCount`. If Tala made manual edits in P3-002, re-validate:

```bash
bun src/tools/charcount.ts "{edited style block text}"
```

- If within 75-120: proceed
- If over 120: trim mood words first, then texture, then production tags
- If under 75: add supporting descriptors

**If no manual edits were made:** Skip this step -- the builder already validated.

**Next:** -> P3-004
```

### P3-007b (Prompt validation gate) -- ADD NOTE

Add a note to P3-007b that the builder has already run conflict checks. The validate-prompt call is now a transitional safety net:

Add to the top of P3-007b:

```markdown
**Note:** The prompt builder (P3-002) already runs genre normalization, mood/genre/production conflict checks, BPM range validation, and artist blocklist scanning. This validate-prompt call is a transitional safety net. Once the builder has run clean for 20+ tracks without validate-prompt catching anything new, this step will be removed.
```

---

## Tala Agent File Changes

Add to the tools table in `.claude/agents/suno.md`:

```markdown
| **prompt-builder** | `bun src/tools/prompt-builder.ts` | `--build style --input {file}` | Deterministic Style block builder (replaces manual Style construction) |
```

Add to the "Hard rules" section:

```markdown
- **Use prompt-builder for Style blocks.** Write StyleInput JSON to `out/style-input.json`, run builder, review output. Override specific elements only when creative intent requires it -- log what you changed. Do NOT manually construct Style blocks from scratch.
```

---

## Integration Test Specifications

### src/tools/prompt-builder.test.ts

```
describe("CLI - JSON file input")
  test("reads JSON from file, produces valid Style output")
  test("nonexistent file: error message with path")
  test("malformed JSON: error message with parse details")

describe("CLI - Flag input")
  test("all flags produce same output as equivalent JSON file")
  test("minimal flags (platform, type, genres, lead, key, bpm): valid output")

describe("CLI - Output format")
  test("first line is the Style block text")
  test("validation section shows char count, element count, trimmed, warnings")
  test("exit code 0 for valid input with warnings")
  test("exit code 1 for invalid input")

describe("Integration - Builder subsumes validate-prompt")
  // For each conflict pair in MOOD_CONFLICTS, GENRE_CONFLICTS, PRODUCTION_CONFLICTS:
  // Build an input that includes the conflicting pair.
  // Assert the builder emits a warning diagnostic.
  // This proves the builder catches everything validate-prompt catches
  // (except cross-prompt coherence).
  test("all mood conflicts detected by builder")
  test("all genre conflicts detected by builder")
  test("all production conflicts detected by builder")
  test("BPM range warnings match validate-prompt behavior")

describe("Integration - Full pipeline")
  test("JSON file -> CLI -> Style text -> within char budget -> no errors")
  test("BGM output: starts with [Instrumental], ends with [No Vocals]")
  test("Vocal output: starts with vocal style, no instrumental metatags")
```

---

## Acceptance Criteria

1. `bun src/tools/prompt-builder.ts --build style --input out/style-input.json` works end-to-end
2. CLI flag mode produces identical output to equivalent JSON input
3. `bun test src/tools/prompt-builder.test.ts` passes -- all integration tests green
4. Builder output passes validate-prompt with no errors (proving subsumption)
5. Tala agent file updated with builder tool and instructions
6. Workflow doc P3-002 and P3-003 updated to use builder
7. P3-007b has transitional safety net note
8. `toolDef` export is discoverable by the registry (`bun src/tools/prompt-builder.ts` runs without error)

---

## Dependencies

| From | Used By |
|------|---------|
| Session 2: `buildStyle()` from `src/libs/prompt-builder/index.ts` | CLI + toolDef |
| Session 2: `StyleInputSchema` from `src/libs/prompt-builder/types.ts` | Input validation |
| Existing: `src/registry/types.ts` - `ToolDefinition` | toolDef shape |
| Existing: `src/tools/validate-prompt.ts` | Integration test comparison |
| Existing: `src/tools/charcount.ts` | Referenced in updated workflow |

---

## Out of Scope

- MiniMax strategy (Phase 4)
- Lyrics builder integration (Phase 2-3)
- Deprecating validate-prompt.ts (transitional period, not this session)
- Phonetics integration (Phase 5)
- Registry modifications -- the registry auto-discovers `toolDef` exports, no manual registration needed
- Changes to tri-critic workflow (P3-008) -- builder output feeds into the existing critic pipeline unchanged

---

## Architectural Notes

- The CLI supports both JSON file input and individual flags. JSON file is the primary mode (Tala writes JSON, calls builder). Flags are for quick manual testing.
- The output format is human-readable text, not JSON. This matches the pattern of other tools in the repo (charcount, validate-prompt). If JSON output is needed later, add a `--json` flag.
- The `toolDef` uses a simple schema (build mode + input path/json). It does NOT replicate all CLI flags -- programmatic callers use JSON input.
- The workflow doc changes are minimal and surgical. P3-002 changes from "manually build Style" to "write JSON + call builder". P3-003 becomes conditional (only if Tala edited the output). The rest of the workflow is untouched.
- The transitional note on P3-007b is important. It sets the expectation that validate-prompt will be removed once the builder proves stable. The 20-track threshold is from the tech doc.
