---
name: Workflow and Quality Control
description: Guidelines for agent critique loops, metatag management, and strategic generation approaches.
type: knowledge
agent: shared
tags: [workflow, quality-gate, critique, metatags, production, gemini, minimax, parallel-processing, jam-skill, tala, rune]
---

# Workflow and Quality Control

## Gemini Critique Loops
Tri-critic loops are implemented for Tala (Step 8c: Gemini → MiniMax → Grok) and Rune (Step 8c: Gemini → MiniMax → Grok) to identify issues like instrument overloads, frequency clashes, and narrative clichés.

**Why:** Ensures higher quality outputs and prevents common generation pitfalls.
**How to apply:** When spawning an agent, you MUST include an explicit instruction in the prompt: *"After building each Style block, run the Gemini critique loop (step 8b)"* or *"Execute the Gemini critique loop for each variation."*
- **Limits:** Up to 3 passes per variation.
- **Exit Criteria:** Early exit if no high-confidence issues remain.
- **Integration:** Suggestions are filtered against reference rules; fixes are mined individually rather than applying rewrites wholesale. Results are logged in Production/Style Notes.

## Metatag Management
A structured system for metatag reliability, including defined tiers and a weighting mechanism, is a core component of the production process.

**Why:** Ensures consistent and effective application of tags, maximizing their impact and clarity in prompts.
**How to apply:** Agents should prioritize higher-tier, weighted tags. Aim for an optimal count of 3 to 5 tags per section or prompt.
- **Reliability Tiers & Weighting:** `production-musical.md` critic includes metatag reliability tiers and a 60/25/10/5% weighting system.
- **Optimal Count:** Target 3-5 tags per section/prompt for maximum impact.

## Creative Generation Strategies
For creative generation tasks, a dual-model parallel processing strategy can leverage the distinct strengths of multiple LLMs.

**Why:** Allows for independent interpretation of input, leading to a more diverse and robust set of outputs, and validates the integration of new models.
**How to apply:** Consider using parallel processing with models like MiniMax and Gemini for tasks requiring diverse creative outputs, such as track naming.
- **Example:** The `/tracknames` tool generates 18 track names (6 per model — MiniMax, Gemini, Grok) tagged by source.

## Orchestration: The `/jam` Skill
The creation of a `/jam` skill (automated chaining of Rune → Tala/Sol) is **intentionally deferred**.

**Why:** The current focus is on instrumental/lo-fi tracks where Tala operates independently, and the complexity of chaining requires careful consideration.
**How to apply:** Activate this skill only once the user manually chains lyrics-to-track generation more than twice, indicating a clear need.

**Consolidated from:**
- `workflow-and-quality-control.md`
