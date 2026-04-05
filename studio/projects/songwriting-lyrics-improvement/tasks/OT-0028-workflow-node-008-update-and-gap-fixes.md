---
title: "Workflow Node 008 Update and Gap Fixes"
type: task
project: songwriting-lyrics-improvement
status: ready
created: 2026-04-03
author: mccall
architect: "vault/studio/projects/songwriting-lyrics-improvement/ARCHITECT.md"
group: 8
parallel: true
depends_on: ["OT-0022"]
---

# OT-0028: Workflow Node 008 Update and Gap Fixes

## What to Build

Update the lyrics workflow (``.claude/skills/lyrics/workflow.md``) for node 008
(self-critic) and fix workflow gaps (US-15).

### Part A: Node 008 Update

**Current (lines 152-174):**
```
Phonetic check (English lyrics only):
Run `bun run tool phonetics --file out/rune-draft.md` after self-critic pass.
Fix high-severity flags (LY2 dead zones, LY5 harsh clusters in soft sections)
before tri-critic. Low-severity flags are advisory.
```

**Replace with:**
```markdown
**Analysis tool check:**
Run `bun run tool lyrics-analyze --file out/rune-draft.md --json` after self-critic pass.

- **BLOCKING:** Syllable counter and stress mapper flags with severity "high" -> MUST rewrite before proceeding
- **ADVISORY:** All other tool flags (rhyme, balance, density, etc.) -> note as context, do not block
- **Iteration:** If rewrite performed, re-run tools on revised draft. Maximum 2 tool-check iterations. Log progress: "Iteration 1: {N} high flags. Iteration 2: {N} high flags. Proceeding."
- **Pass to critics:** Tool output summary included in the draft context passed to tri-critics at nodes 009-011

Applies to ALL languages — the tool handles language detection internally.
```

### Part B: Workflow Gap Fixes (US-15)

**B1. Unlimited variations (AC-15.1):**
- Find any references to hardcoded variation limits in workflow.md
- Ensure variation numbering is open-ended: V1, V2, V3, V4, V5...
- Update node 014 (User review) and node 015 (Write approved file) to support N variations

**B2. Discover integration (AC-15.2):**
- Update node 007 to make discover.ts output from node 003 a first-class input
- Currently it's "supplementary" -- change to explicit: "Include discover.ts context from node 003 as primary reference material in the subagent prompt"

**B3. Rewrite context (AC-15.3):**
- Update node 014 adjust path to include:
  - (a) Original variation text
  - (b) User's adjustment instructions
  - (c) Tool analysis report from the original
  - (d) Which specific flags the adjustment should address

**B4. Self-critic loop logging (AC-15.4):**
- Already handled in Part A's iteration logging

### Part C: Node 012 Minor Update

Add "Source" column to reconciliation table template:

```markdown
| # | Line | Source | Gemini | MiniMax | Grok | Consensus | Decision |
|---|------|--------|--------|---------|------|-----------|----------|
```

"Source" values: "tool" (from analysis tools), "critic" (from LLM critics).
Tool-originated high-severity flags that survived node 008 carry mandatory-fix weight.

## Acceptance Criteria

- [ ] AC-1: Node 008 references `lyrics-analyze` instead of `phonetics`
- [ ] AC-2: Non-English skip clause REMOVED from node 008
- [ ] AC-3: Blocking policy explicit: syllable + stress high flags = must rewrite
- [ ] AC-4: Max 2 tool-check iterations specified with logging format
- [ ] AC-5: Tool output passed to tri-critics as context
- [ ] AC-6: Variation numbering supports N variations (no hardcoded limit)
- [ ] AC-7: Node 007 receives discover.ts output as first-class input
- [ ] AC-8: Node 014 adjust path includes original + instructions + tool report + specific flags
- [ ] AC-9: Node 012 reconciliation table includes "Source" column

Maps to PRD: US-11 (AC-11.1 through AC-11.6), US-15 (AC-15.1 through AC-15.4)

## Files in Scope

- `.claude/skills/lyrics/workflow.md` -- **modify** (nodes 007, 008, 012, 014)
- `.claude/skills/lyrics/SKILL.md` -- **read-only** (reference for flow understanding)

## Architectural Constraints

- ADR-7: Blocking policy lives in the workflow layer, NOT in the tools. The tool returns flags with severities. This workflow node decides what blocks
- Do NOT modify nodes 009, 010, 011 (those are handled in OT-0029)
- Do NOT modify node 013 (that's OT-0030)
- Preserve all existing workflow logic that isn't being updated

## Out of Scope

- Node 009 critic changes (OT-0029)
- Node 013 advisory gate (OT-0030)
- Sensory anchor automation (AC-15.5, Could Have, deferred)
- Tool implementation changes

## Test Plan

Manual review: read the updated workflow.md end-to-end and verify:
1. Node 008 references the correct tool command
2. Blocking vs advisory is clear and unambiguous
3. Iteration limit is specified
4. No references to the old `phonetics` tool remain in the workflow
5. Variation numbering is open-ended
6. Discover integration is explicit at node 007

```bash
grep -n "phonetics" .claude/skills/lyrics/workflow.md  # should return 0 results
grep -n "lyrics-analyze" .claude/skills/lyrics/workflow.md  # should return results at node 008
```

## Notes

- This is a workflow doc task, not a code task. The changes are to markdown instructions that agents follow.
- The blocking policy location (ADR-7) is critical. The tools are pure functions returning facts. The workflow decides what blocks. This node is where that decision is encoded.