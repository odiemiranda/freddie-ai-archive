---
title: "Stress Mapper Tool"
type: task
project: songwriting-lyrics-improvement
status: done
created: 2026-04-03
author: mccall
architect: "vault/studio/projects/songwriting-lyrics-improvement/ARCHITECT.md"
group: 2
parallel: true
depends_on: ["OT-0016"]
---

# OT-0018: Stress Mapper Tool

## What to Build

Implement the T1 stress mapper at `src/libs/lyrics-analyzer/tools/stress-mapper.ts`.
This tool outputs a Strong/Weak pattern per line, classifies metrical feet, and flags
rhythmic dead zones.

**Logic:**
- English: CMU stress markers. 0 = unstressed (W), 1 = primary stress (S), 2 = secondary stress (S)
- Tagalog: Penultimate syllable stressed (malumay default). Check `TAGALOG_STRESS_EXCEPTIONS` for overrides
- Unknown/heuristic words: no stress data available. Mark as "?" in pattern

**Metrical foot classification:**
- Analyze pairs/triples of S/W in the pattern
- Iambic: W S W S (most common in English)
- Trochaic: S W S W
- Anapestic: W W S W W S
- Dactylic: S W W S W W
- Mixed: no dominant pattern

**Flagging rules:**
- 3 consecutive unstressed syllables -> "rhythmic dead zone", severity: medium
- 4+ consecutive unstressed syllables -> severity: high
- Function words (articles, prepositions, pronouns) on positions that map to strong beats -> advisory warning, severity: low

**Function word list:** "a", "an", "the", "of", "in", "on", "at", "to", "for", "is",
"am", "are", "was", "were", "be", "been", "it", "its", "my", "your", "his", "her",
"our", "their", "this", "that", "and", "but", "or", "if", "so"

**Export:**
```typescript
export function mapStress(input: ToolInput): StressReport;

interface StressReport extends ToolReport {
  tool: "stress-mapper";
  perLine: Array<{
    lineNumber: number;
    text: string;
    pattern: string;            // "S W S W S"
    metricalFeet: "iambic" | "trochaic" | "anapestic" | "dactylic" | "mixed";
    deadZones: Array<{ start: number; length: number }>;
  }>;
}
```

**Status rollup:**
- Any high-severity flag -> status: "fail"
- Any medium-severity flag (no high) -> status: "warn"
- All low or none -> status: "pass"

## Acceptance Criteria

- [ ] AC-1: "Walking through the morning light" produces S/W pattern with metrical classification
- [ ] AC-2: English stress derived from CMU markers (0/1/2). Both primary (1) and secondary (2) count as S
- [ ] AC-3: Tagalog word stress defaults to penultimate. "maganda" stressed on second syllable
- [ ] AC-4: Known Tagalog exception overrides penultimate default
- [ ] AC-5: Line with 4 consecutive unstressed syllables flagged severity: high
- [ ] AC-6: Line with 3 consecutive unstressed flagged severity: medium
- [ ] AC-7: Function word "the" on a strong-beat position generates advisory warning
- [ ] AC-8: Words with no stress data (heuristic source) show "?" in pattern
- [ ] AC-9: Metrical foot correctly classified for standard iambic pentameter line
- [ ] AC-10: This tool BLOCKS at node 008 -- high-severity flags must trigger rewrite (policy enforced by workflow, not by this tool, but the flags must be present)

Maps to PRD: US-2 (AC-2.1 through AC-2.6)

## Files in Scope

- `src/libs/lyrics-analyzer/tools/stress-mapper.ts` -- **create**
- `src/libs/lyrics-analyzer/shared/*` -- **read-only** (import from OT-0016)
- `src/libs/prompt-data/tagalog-stress-exceptions.ts` -- **read-only** (import from OT-0016)
- `src/libs/phonetics.ts` -- **read-only reference** (port `checkMeterStress()` logic, lines 153-256)

## Architectural Constraints

- Port dead-zone detection logic from `src/libs/phonetics.ts` `analyzeSectionMeter()` (lines 206-256)
- Do NOT import from `phonetics.ts` -- copy and improve the logic
- Tool is a pure function: input in, report out. No side effects
- Blocking policy is NOT in the tool. Tool returns flags with severities. Workflow decides what blocks

## Out of Scope

- Syllable counting (uses `WordToken.syllables` from ToolInput, computed by OT-0017)
- Prosody alignment / beat mapping (OT-0026)
- CLI entry point (OT-0021)
- Modification of existing `phonetics.ts`

## Test Plan

```bash
bun -e "
import { mapStress } from './src/libs/lyrics-analyzer/tools/stress-mapper.js';
// Build ToolInput with pre-tokenized words
// Test: standard iambic line, dead zone line, Tagalog line
"
```

Test cases:
- "to BE or NOT to BE" -> W S W S W S (iambic)
- "in the middle of a conversation" -> should flag dead zone in unstressed run
- Line with only heuristic words -> pattern has "?" markers

## Notes

- The existing `checkMeterStress()` in `phonetics.ts` does both syllable counting AND stress checking in one pass. This task separates concerns: syllable counts come pre-computed in `WordToken`, this tool only does stress analysis.
- The function word list is intentionally short. It covers English articles, prepositions, and common pronouns. Tagalog function words (ang, ng, sa) can be added later.