---
title: "Rhyme Scheme Identifier"
type: task
project: songwriting-lyrics-improvement
status: done
created: 2026-04-03
author: mccall
architect: "vault/studio/projects/songwriting-lyrics-improvement/ARCHITECT.md"
group: 6
parallel: false
depends_on: ["OT-0023", "OT-0024"]
---

# OT-0025: Rhyme Scheme Identifier

## What to Build

Implement the T2 rhyme scheme identifier at
`src/libs/lyrics-analyzer/tools/rhyme-scheme.ts`. This tool consumes the rhyme detector
output (from OT-0019) and labels each section with its rhyme scheme pattern.

**Algorithm:**
1. Get end rhymes from `RhymeReport.endRhymes` for the current section
2. Build a line-to-rhyme-group mapping: lines that rhyme together share a letter
3. Assign alphabetic labels: first unmatched line = A, second unmatched = B, etc.
4. Lines that rhyme with an earlier line get that line's letter
5. Output the pattern string (e.g., "ABAB", "AABB", "ABCB")

**Configurable threshold:**
- End-rhyme score must be >= threshold (default 0.5) to count as a rhyme pair
- Below threshold = different letter

**Scheme consistency:**
- Compare all instances of a section type (e.g., Verse 1, Verse 2, Verse 3)
- If schemes differ, flag as inconsistency (advisory, severity: low)
- Example: Verse 1 = ABAB, Verse 2 = AABB -> inconsistency flag

**Free verse:**
- Section where no lines rhyme = "FREE"
- Not flagged as error -- free verse is a valid choice

**Export:**
```typescript
export function identifyScheme(
  input: ToolInput,
  rhymeReport: RhymeReport
): RhymeSchemeReport;

interface RhymeSchemeReport extends ToolReport {
  tool: "rhyme-scheme";
  perSection: Array<{
    section: string;
    scheme: string;
    confidence: number;
  }>;
  consistencyFlags: Array<{
    sectionType: SectionType;
    sections: string[];
    schemes: string[];
    issue: string;
  }>;
}
```

**Note on dependency:** This tool needs `RhymeReport` output. The orchestrator in
`index.ts` must run rhyme-detector first and pass its output to this tool. The tool
signature differs from the standard `ToolInput` -- it takes an extra parameter.

## Acceptance Criteria

- [ ] AC-1: AABB pattern correctly identified (lines 1-2 rhyme, lines 3-4 rhyme)
- [ ] AC-2: ABAB pattern correctly identified
- [ ] AC-3: ABCB pattern correctly identified (lines 2,4 rhyme, lines 1,3 don't)
- [ ] AC-4: Free verse section labeled "FREE"
- [ ] AC-5: Consistency flag when Verse 1 = ABAB and Verse 2 = AABB
- [ ] AC-6: Configurable threshold (lower threshold = more rhyme pairs detected)
- [ ] AC-7: Confidence value: ratio of paired lines to total lines in section

Maps to PRD: US-5 (AC-5.1 through AC-5.5)

## Files in Scope

- `src/libs/lyrics-analyzer/tools/rhyme-scheme.ts` -- **create**
- `src/libs/lyrics-analyzer/tools/rhyme-detector.ts` -- **read-only** (import RhymeReport type)
- `src/libs/lyrics-analyzer/types.ts` -- **read-only**

## Architectural Constraints

- This tool depends on `RhymeReport` from rhyme-detector. The orchestrator in `index.ts` must ensure rhyme-detector runs first. Update `index.ts` orchestration order in OT-0026 to handle this dependency
- The extra parameter pattern is intentional: some T2/T3 tools consume T1 output
- Thresholds should be configurable constants at top of file

## Out of Scope

- Rhyme detection itself (OT-0019)
- CLI/report update for this tool (OT-0026)
- Scheme recommendation (telling the user what scheme to use)

## Test Plan

```bash
# Test with known schemes
# AABB: cat/hat, dog/log
# ABAB: cat/dog, hat/log
# FREE: cat/dog/fish/bird (no rhymes)
```

## Notes

- Confidence = paired lines / total lines. AABB with 4 lines = confidence 1.0. ABCB with 4 lines = confidence 0.5 (only 2 of 4 lines paired).
- This is a consumer of rhyme-detector, not a replacement. It adds structural labeling on top of the pair-level detection.