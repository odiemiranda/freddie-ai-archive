---
title: "Node 013 Advisory Gate"
type: task
project: songwriting-lyrics-improvement
status: ready
created: 2026-04-03
author: mccall
architect: "vault/studio/projects/songwriting-lyrics-improvement/ARCHITECT.md"
group: 10
parallel: false
depends_on: ["OT-0026", "OT-0029"]
---

# OT-0030: Node 013 Advisory Gate

## What to Build

Update node 013 (Present to user) in the lyrics workflow to include an optional
"Technical Notes (ADVISORY)" section showing singability score and prosody warnings
when T3 tools are available.

### Workflow Node 013 Update

**Current (lines 250-259):**
```markdown
Show: Approach, Lyrics (ready to paste), Style Block Notes. Do NOT write to disk yet.
```

**Add after the current presentation format:**
```markdown
**Technical Notes (ADVISORY):**
If T3 tools (singability-score, prosody-alignment) produced output during
the node 008 analysis pass, include a "Technical Notes" section:

```
### Technical Notes (ADVISORY)
- Singability Score: {composite}/100
  - Lowest sub-score: {name} ({score}/100) — {brief explanation}
- Prosody Warnings: {count} lines with meter alignment notes
  - {summary of most significant prosody flag, if any}
```

Rules:
- Label clearly as "ADVISORY" — these do not block approval
- If singability score > 80: note "Good singability" with no detail needed
- If singability score < 60: list the two lowest sub-scores
- If no T3 tools ran (Phase 2 not yet shipped): omit this section entirely. Do not show empty output
- Prosody warnings: summarize only, do not list every line
```

## Acceptance Criteria

- [ ] AC-1: Node 013 includes Technical Notes section format
- [ ] AC-2: Section clearly labeled "ADVISORY"
- [ ] AC-3: Singability score shown with lowest sub-score highlighted
- [ ] AC-4: If no T3 tools available, section is omitted (not empty)
- [ ] AC-5: Prosody warnings summarized, not exhaustively listed
- [ ] AC-6: Does not block user approval

Maps to PRD: US-13 (AC-13.1 through AC-13.3)

## Files in Scope

- `.claude/skills/lyrics/workflow.md` -- **modify** (node 013 only)

## Architectural Constraints

- Advisory only. NEVER block approval based on T3 output
- Graceful degradation: if T3 tools don't exist yet, nothing changes
- The presentation should be scannable in 5 seconds. No data dumps

## Out of Scope

- T3 tool implementation (already done in OT-0026)
- Nodes other than 013
- Singability calibration

## Test Plan

Manual review:
1. Read node 013 -- is the technical notes section clear and scannable?
2. Is the "omit if no T3 tools" instruction explicit?
3. Does it say "ADVISORY" prominently?

## Notes

- This is the lowest-priority task in the entire project (Could Have). It adds polish but doesn't affect the core analysis pipeline.
- The singability score is not yet calibrated (OT-0026 ships it with `calibrated: false`). The advisory framing is important -- users should not over-rely on it yet.
- This is a small task. Estimated 15-20 minutes of work.