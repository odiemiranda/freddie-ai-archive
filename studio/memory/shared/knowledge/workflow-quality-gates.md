---
name: Workflow Quality Gates
description: Guidance on Gemini critique loops and session orchestration status.
type: knowledge
agent: shared
tags: [workflow, quality-gate, gemini, critique, jam-skill]
---

# Workflow Quality Gates

## Gemini Critique Loops
Critique loops are implemented for **Tala** (Step 8b: Style Critique Loop) and **Rune** (Step 8b: Lyrics Critique Loop) to identify instrument overloads, frequency clashes, and narrative clichés.

### The "Explicit Nudge" Requirement
While documented in agent files, agents (Tala/Rune) often skip these multi-pass tool calls during autonomous execution. 
- **How to apply:** When spawning an agent, you MUST include an explicit instruction in the prompt: *"After building each Style block, run the Gemini critique loop (step 8b)"* or *"Execute the Gemini critique loop for each variation."*

### Critique Mechanics
- **Limits:** Up to 3 passes per variation.
- **Exit Criteria:** Early exit if no high-confidence issues remain.
- **Integration:** Suggestions are filtered against reference rules; fixes are mined individually rather than applying rewrites wholesale. Results are logged in Production/Style Notes.

## Orchestration: The /jam Skill
The creation of a `/jam` skill (automated chaining of Rune → Tala/Sol) is **intentionally deferred**. 
- **Why:** The current focus is on instrumental/lo-fi tracks where Tala operates independently. 
- **Trigger for Action:** Activate this skill once the user manually chains lyrics-to-track generation more than twice.

**Consolidated from:** 
- `2026-03-21-183900-gemini-critique-loops-live.md`
- `2026-03-21-184000-jam-session-skill-deferred.md`
- `2026-03-21-201500-gemini-critique-needs-explicit-nudge.md`
