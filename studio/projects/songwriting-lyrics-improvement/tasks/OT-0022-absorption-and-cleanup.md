---
title: "Absorption and Cleanup"
type: task
project: songwriting-lyrics-improvement
status: done
created: 2026-04-03
author: mccall
architect: "vault/studio/projects/songwriting-lyrics-improvement/ARCHITECT.md"
group: 4
parallel: false
depends_on: ["OT-0021"]
---

# OT-0022: Absorption and Cleanup

## What to Build

Delete the old phonetics files and update all callers to use the new
`lyrics-analyzer` library. Add backward compatibility alias in the tool runner.

**Files to delete:**
1. `src/libs/phonetics.ts` (438 lines) -- fully absorbed by lyrics-analyzer
2. `src/tools/phonetics.ts` (127 lines) -- replaced by `lyrics-analyze.ts`
3. `src/libs/prompt-data/syllable-heuristic.ts` (70 lines) -- logic ported to syllable-counter

**Callers to update:**

1. `src/libs/prompt-data/index.ts` -- remove the line `export * from "./syllable-heuristic.js";`

2. `src/tool.ts` -- add alias:
```typescript
const aliases: Record<string, string> = {
  research: "notebooklm",
  db: "brain",
  phonetics: "lyrics-analyze",  // backward compat
};
```

3. Any test files importing from `phonetics.ts` or `syllable-heuristic.ts` -- update imports

**Grep for all importers before deleting:**
```bash
bun run tool grep "phonetics" --type ts
bun run tool grep "syllable-heuristic" --type ts
```

## Acceptance Criteria

- [ ] AC-1: `src/libs/phonetics.ts` deleted
- [ ] AC-2: `src/tools/phonetics.ts` deleted
- [ ] AC-3: `src/libs/prompt-data/syllable-heuristic.ts` deleted
- [ ] AC-4: `src/libs/prompt-data/index.ts` no longer exports `syllable-heuristic`
- [ ] AC-5: `bun run tool phonetics --text "test"` still works (routes to lyrics-analyze via alias)
- [ ] AC-6: No remaining imports referencing the deleted files (grep clean)
- [ ] AC-7: `bun run tool --list` shows `lyrics-analyze` and `phonetics` alias
- [ ] AC-8: `src/libs/prompt-data/homographs.ts` unchanged and still imported by new library

Maps to PRD: US-1 (AC-1.7), M7 (no orphaned code)

## Files in Scope

- `src/libs/phonetics.ts` -- **delete**
- `src/tools/phonetics.ts` -- **delete**
- `src/libs/prompt-data/syllable-heuristic.ts` -- **delete**
- `src/libs/prompt-data/index.ts` -- **modify** (remove syllable-heuristic re-export)
- `src/tool.ts` -- **modify** (add phonetics alias)
- Any test files referencing old imports -- **modify**

## Architectural Constraints

- `src/libs/prompt-data/homographs.ts` is NOT deleted. It is imported by the new library
- The alias `phonetics -> lyrics-analyze` ensures any workflow or script referencing the old tool name still works
- Backward compat alias: `bun run tool phonetics --text "..."` must route to `lyrics-analyze` and produce output. The old format (separate stress/texture/homograph sections) is NOT preserved -- callers get the new unified format
- Run full grep before deleting anything. If an unexpected caller exists, update it

## Out of Scope

- Modifying `lyrics-analyze.ts` behavior (done in OT-0021)
- Workflow updates (Phase 3)
- Any new tool implementations

## Test Plan

```bash
# Before deletion: grep for all references
grep -r "phonetics" src/ --include="*.ts" -l
grep -r "syllable-heuristic" src/ --include="*.ts" -l

# After deletion: verify no broken imports
bun build src/libs/prompt-data/index.ts --no-bundle
bun run tool lyrics-analyze --text "Walking through the morning light"
bun run tool phonetics --text "Walking through the morning light"  # alias test
bun run tool --list | grep -E "phonetics|lyrics-analyze"

# Verify homographs still works
bun -e "import { HOMOGRAPH_LOOKUP } from './src/libs/prompt-data/homographs.js'; console.log(HOMOGRAPH_LOOKUP.get('live'))"
```

## Notes

- This is a high-risk task. Deleting files that other code depends on causes build failures. That's why it runs LAST in Phase 1, after everything else is verified working.
- R-NEW-2 from the architecture: absorption regression risk. Keep the old files in git history. If something breaks, `git checkout` recovers them.
- If any unexpected caller surfaces during the grep, update the task doc (ping McCall) before proceeding with deletion.