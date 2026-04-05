---
title: "Node 009 and Critic Prompt Update"
type: task
project: songwriting-lyrics-improvement
status: ready
created: 2026-04-03
author: mccall
architect: "vault/studio/projects/songwriting-lyrics-improvement/ARCHITECT.md"
group: 9
parallel: false
depends_on: ["OT-0028"]
---

# OT-0029: Node 009 and Critic Prompt Update

## What to Build

Update node 009 (Gemini structural critic) in the workflow and update the
`lyrics-structural.md` critic prompt to consume tool output instead of running its
own phonetic analysis.

### Part A: Workflow Node 009 Update

**Current (lines 178-190):**
Node 009 dispatches Gemini with the critic prompt and the draft.

**Update:** Add instruction that the tool report from node 008 is included in the
critic input. The Gemini call should include the JSON report alongside the draft.

```markdown
## [009] Tri-critic: Gemini structural

**Input:** Self-critiqued draft + lyrics analysis report (JSON from node 008)
**Output:** Structural feedback (volta, density, singability, cliches, sensory detail)

Include the tool analysis JSON report from node 008 in the input file alongside the draft.

```bash
bun run tool gemini --system-file .claude/critics/lyrics-structural.md --input-file {draft-with-report} "Review for {GENRE}. Check: VOLTA, PER-SECTION DENSITY, SINGABILITY, CLICHE, SENSORY DETAIL. Refer to LYRICS ANALYSIS REPORT for deterministic metrics."
```
```

### Part B: Critic Prompt Update

Update `.claude/critics/lyrics-structural.md`:

1. **Section 5 (Phonetic Analysis) -- REWRITE:**

   Replace the current section 5 (lines 35-40 approximately) which says
   "Run `bun run tool phonetics`..." with:

   ```markdown
   ### 5. Phonetic Analysis (Tool-Validated)
   A LYRICS ANALYSIS REPORT is included with the draft. This report contains
   deterministic tool output for: syllable counts, stress patterns, rhyme quality
   and density, section balance metrics, and phonetic texture.

   - **Refer to the report for meter, stress, rhyme, and texture data.
     Do not re-derive these metrics.** The tools are more accurate than LLM estimation.
   - **LY2 — Meter/Stress:** Check the stress-mapper section of the report.
     High-severity flags should already be fixed (node 008 blocks on these).
     If any remain, flag them prominently.
   - **LY5 — Consonant Texture:** Check the vowel-consonant-density section.
     Texture-energy mismatches are advisory.
   - If the report is not present (tools did not run), fall back to manual assessment.
   ```

2. **Remove non-English skip clause:**

   Delete the line (approximately line 40):
   `- **When lyrics are non-English:** Skip phonetic analysis (CMU dict is English-only).`

   Replace with: nothing. The tool handles language detection internally. No skip needed.

3. **Leave all other sections unchanged:**
   - Section 1 (Vulnerability Filter) -- unchanged
   - Section 2 (Anti-Cliche Rule) -- unchanged
   - Section 3 (Narrative Arc / Volta) -- unchanged
   - Section 4 (Breath Test) -- unchanged
   - Lyrical Archetypes -- unchanged
   - Evaluation Output Format -- unchanged

## Acceptance Criteria

- [ ] AC-1: Node 009 input includes tool analysis report alongside draft
- [ ] AC-2: Critic prompt section 5 references tool output, not `bun run tool phonetics`
- [ ] AC-3: Critic prompt says "Do not re-derive these metrics"
- [ ] AC-4: Non-English skip clause removed
- [ ] AC-5: Fallback instruction present: "If report not present, fall back to manual assessment"
- [ ] AC-6: `lyrics-intentionality.md` critic unchanged (AC-12.5)
- [ ] AC-7: All other sections of `lyrics-structural.md` preserved

Maps to PRD: US-12 (AC-12.1 through AC-12.5)

## Files in Scope

- `.claude/skills/lyrics/workflow.md` -- **modify** (node 009 only)
- `.claude/critics/lyrics-structural.md` -- **modify** (section 5 + non-English clause)
- `.claude/critics/lyrics-intentionality.md` -- **DO NOT MODIFY** (read-only confirmation)

## Architectural Constraints

- The critic consumes tool output. It does NOT run tools itself
- The tool report may not always be present (e.g., tool crashed, first-time setup). The critic must have a fallback
- Nodes 010, 011 (MiniMax, Grok) remain UNCHANGED per architecture

## Out of Scope

- Modifying nodes 010, 011 (unchanged per architecture)
- Modifying `lyrics-intentionality.md`
- Tool implementation changes
- Node 013 (OT-0030)

## Test Plan

Manual review:
1. Read updated `lyrics-structural.md` section 5 -- does it clearly reference tool output?
2. Grep for "phonetics" in both files -- should return 0 results
3. Grep for "non-English" or "Skip phonetic" in the critic -- should return 0 results
4. Verify sections 1-4 and archetypes are unchanged

```bash
grep -n "bun run tool phonetics" .claude/critics/lyrics-structural.md  # should be 0
grep -n "non-English" .claude/critics/lyrics-structural.md  # should be 0
grep -n "lyrics-analyze" .claude/skills/lyrics/workflow.md  # should appear at node 008 and 009 context
```

## Notes

- This is a doc/prompt task. No code changes.
- The key insight: critics should not run tools. Tools run at node 008 (self-critic). Critics at 009-011 consume the results. This separation is ADR-7 in action.
- The fallback clause is important. If the tool pipeline breaks, the critic should still function (degraded but not dead). This is the containment boundary from the architecture's failure modes.