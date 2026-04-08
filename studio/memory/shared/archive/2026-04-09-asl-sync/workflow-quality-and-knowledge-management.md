---
name: Workflow, Quality Control, and Knowledge Management
description: Guidelines for agent critique loops, metatag management, strategic generation approaches, including advanced prompt generation methodologies, LLM capabilities and selection (e.g., for creative notation), and best practices for managing knowledge, general development, tooling, architectural vision, and external service reliability.
type: knowledge
agent: shared
tags: [workflow, workflow_management, workflow-pattern, quality-gate, critique, multi-pass, metatags, production, gemini, minimax, parallel-processing, jam-skill, tala, rune, knowledge-management, consolidation, llm, token-limits, data-integrity, best-practices, llm-capabilities, llm-selection, notation, prompt-engineering, text-generation, music-lyrics, linguistics, phonetics, homograph-detection, structure, accuracy, tool_development, cli_first, user_preference, workflow_design, readability, maintainability, code_style, architecture, parser, serializer, testing, library_design, agent-design, tool-capability, interaction_protocols, adaptability, scope_management, tactical_pivot, feasibility_spike, vision, ecosystem, foundation, strategy, platform-limits, timeout, critic-reconciliation, genre-limits, validation, implementation, spec-drift, dependencies, debugging, assumptions, spec-compliance, communication, issue-resolution, quality-assurance, failure_mode, external_service, reliability, error_handling, recovery]
---

# Workflow, Quality Control, and Knowledge Management

## Strategic Vision and Architectural Principles
The strategic direction emphasizes building a robust, composable tool ecosystem with a strong foundational infrastructure, prioritizing stability and interoperability before engaging in extensive creative experimentation. This aligns with the user's articulated vision for the system as an 'ecosystem not framework,' where 'tools compound on each other' and the 'foundation must be solid before creative testing.'

**Why:** Ensures long-term system stability, scalability, and composability, aligning with user preference for a robust, interconnected tool ecosystem.
**How to apply:** Prioritize the development of stable, interoperable foundational tools and infrastructure. Defer extensive creative experimentation until the underlying ecosystem is solid and proven. Design tools to be composable and to build upon each other's capabilities.

## LLM Token Limits and Silent Data Loss
When using Large Language Models (LLMs) for structured, multi-file output, it is critical to be aware of `maxOutputTokens` limits. If the output exceeds this limit, the model will silently truncate its response, leading to incomplete or lost data without error notification.

**Specific MiniMax Limitation:** The MiniMax LLM has a strict 256 token limit, which can be easily exhausted by complex reasoning tasks (e.g., during 'reasoning' within the AAR Phase 1 pipeline), leading to silent truncation and bugs.

**Why:** Prevents silent data loss and ensures the integrity and completeness of consolidated knowledge. Understanding specific LLM limitations (like MiniMax's 256 token limit) is crucial for effective prompt engineering.
**How to apply:**
1.  **Increase `maxOutputTokens`:** Set the limit significantly higher than the expected output, providing ample headroom (e.g., 4x the estimated need).
2.  **Implement Post-Parse Checks:** After the LLM generates its output, compare the planned number of output files/sections with the actual parsed count. Warn if there's a mismatch.
3.  **Implement Post-Write Verification:** After writing files to disk, verify that the contents of the knowledge directory match the consolidation plan.
4.  **For MiniMax:** When using MiniMax for cognitive or reasoning tasks, employ careful prompt engineering, including explicit output constraints or chunking of reasoning steps, to prevent truncation and ensure complete processing.

**Lesson:** Always verify the output count matches the plan when using LLMs for structured multi-file output. Token limits cause silent truncation.

## External Service Reliability and Recovery
External service reliability, specifically 5xx errors indicating server overload, can directly block internal agent dispatch and workflow progression. A robust workflow must account for such external failures, requiring internal agents to identify the blocking condition, issue 'NEEDS-FIX' directives, and implement a pause/resume strategy.

**Why:** External service failures can disrupt development workflows and lead to unexpected blockages. Implementing explicit recovery protocols ensures workflow resilience and efficient issue resolution.
**How to apply:** Design workflows to anticipate and handle external service failures (e.g., 5xx errors). Internal agents (e.g., McCall, Ryan) should be equipped to identify blocking conditions, issue 'NEEDS-FIX' directives, and execute predefined pause/resume sequences to manage tasks (e.g., ASL-0017) effectively.

## Gemini Critique Loops
Tri-critic loops are implemented for Tala (Step 8c: Gemini → MiniMax → Grok) and Rune (Step 8c: Gemini → MiniMax → Grok) to identify issues like instrument overloads, frequency clashes, narrative clichés, and subtle audio quality issues stemming from genre archetypes. Critics are effective at pinpointing these genre-specific pitfalls (refer to `suno-prompt-engineering-and-references.md` for details on 'instrument bleed' and other genre archetype pitfalls, including MiniMax's ability to detect sonic redundancy). Multi-critic reconciliation is highly effective in enforcing established prompt engineering principles, such as limiting genres (e.g., 'Max 2 genres'), thereby preventing prompt dilution and improving output quality.

**Gemini is also a capable tool for critiquing agent design documents**, demonstrating proficiency in identifying structural bloat, tactical deficiencies, and qualitative aspects like opinion strength and voice consistency. A **multi-stage critique workflow** is effective, utilizing an initial 'flash' pass for high-level structural issues (e.g., redundancy, intentionality) followed by a 'pro-level' pass for deeper content and qualitative analysis (e.g., tactical depth, voice, opinion quality).

**Note on MiniMax Reliability:** MiniMax may exhibit timeout issues during intensive council reviews, suggesting a preference for Gemini and Grok for reliable real-time collaborative architectural discussions.

**Why:** Ensures higher quality outputs and prevents common generation pitfalls, including those specific to genre interpretation and agent design. The multi-stage approach provides comprehensive critique, and understanding LLM-specific behaviors ensures reliable tool selection.
**How to apply:** When spawning an agent, you MUST include an explicit instruction in the prompt: *"After building each Style block, run the Gemini critique loop (step 8b)"* or *"Execute the Gemini critique loop for each variation."* For document critique, consider a multi-stage approach with 'flash' for initial structural review and 'pro-level' for detailed content analysis. For critical, real-time collaborative reviews, prioritize Gemini and Grok over MiniMax due to potential timeout issues.
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
For creative generation tasks, a dual-model parallel processing strategy can leverage the distinct strengths of multiple LLMs. Gemini 2.5 Flash demonstrates superior capability for creative text generation tasks, specifically musical notation, making it a preferred choice for nuanced and imaginative outputs compared to other considered models.

**Why:** Allows for independent interpretation of input, leading to a more diverse and robust set of outputs, and validates the integration of new models. Leveraging specific LLM strengths (e.g., Gemini 2.5 Flash for creative notation) improves output quality for nuanced tasks.
**How to apply:** Consider using parallel processing with models like MiniMax and Gemini for tasks requiring diverse creative outputs, such as track naming. For tasks requiring highly creative or nuanced text generation, such as musical notation, prioritize Gemini 2.5 Flash.
- **Example:** The `/tracknames` tool generates 18 track names (6 per model — MiniMax, Gemini, Grok) tagged by source.

### Prompt Builder Engine Methodologies
The Prompt Builder Engine incorporates a 'MiniMax 6-section prose strategy' for structuring content and 'Phonetics syllable fallback' with 'LY6 homograph detection' for linguistic accuracy.

**Why:** Effective prompt generation for music-related tasks benefits significantly from structured content strategies (like the MiniMax 6-section prose) and advanced linguistic processing (phonetics, homograph detection) to ensure high-quality, nuanced outputs. These specific techniques are integrated into the Prompt Builder.
**How to apply:** Utilize structured content strategies like the MiniMax 6-section prose for overall prompt organization. Employ advanced linguistic processing, including phonetics syllable fallback and homograph detection, to enhance the accuracy and nuance of generated music-related prompts.

## Agent Interaction Protocols
Formalizing interaction protocols for specific complex workflow scenarios (e.g., scope change, tactical pivot, feasibility spike) significantly enhances agent coordination, adaptability, and decision-making under pressure.

**Why:** Improves agent coordination, adaptability, and decision-making in complex or high-pressure workflow scenarios.
**How to apply:** Develop and adhere to specific interaction protocols for handling workflow events such as scope changes, tactical pivots, and feasibility spikes to ensure consistent and effective agent responses.

## Tooling and Development Best Practices

### CLI-First Tool Design
Avoid using inline code blocks (e.g., `bun -e`) directly within workflows. Instead, encapsulate such logic into dedicated CLI tools. A `no-inline-code` rule should be enforced across all skill workflows.

**Why:** Improves readability, maintainability, and aligns with user preference for explicit, well-defined tool calls, contributing to more robust workflow design.
**How to apply:** For any new or refactored workflow logic, develop dedicated CLI tools for encapsulation rather than embedding inline code. Ensure all skill workflows adhere to a `no-inline-code` rule.

### Modular Data Parsing Architecture
For developing robust data parsing and manipulation libraries, a modular architecture separating `types`, `parser`, `serializer`, and `operations` components, coupled with comprehensive roundtrip testing, is a highly effective pattern.

**Why:** Ensures data integrity and reliable functionality by promoting clear separation of concerns and thorough validation throughout the development lifecycle.
**How to apply:** When building new parsing or data manipulation libraries, implement a modular design with distinct components for `types`, `parser`, `serializer`, and `operations`. Verify integrity using 'roundtrip gate' tests across multiple data files.

## Orchestration: The `/jam` Skill
The creation of a `/jam` skill (automated chaining of Rune → Tala/Sol) is **intentionally deferred**.

**Why:** The current focus is on instrumental/lo-fi tracks where Tala operates independently, and the complexity of chaining requires careful consideration.
**How to apply:** Activate this skill only once the user manually chains lyrics-to-track generation more than twice, indicating a clear need.

## Specification Discrepancies and Resolution
Specifications may contain outdated assumptions, omit necessary dependencies, overlook side effects in different operational modes, or fail to account for the need to test critical guardrails. These are common areas where spec drift and missing details can occur during implementation.

**Why:** Identifying and addressing these discrepancies proactively ensures alignment, prevents technical debt from silent workarounds, and leads to more robust and correctly implemented features.
**How to apply:** When encountering discrepancies between a specification and reality during implementation, proactively flag and escalate these issues to relevant stakeholders for clarification and resolution, rather than attempting silent workarounds.

**Consolidated from:**
- `knowledge-management-best-practices.md`
- `consolidation-token-limit-lesson.md`
- `workflow-and-quality-control.md`
- `20260329-002043-specific-genre-archetypes-can-introduce--1.md` (high-level mention of genre pitfalls)
- `20260331-165434-gemini-2-5-flash-demonstrates-superior-c-1.md`
- `20260331-184555-effective-prompt-generation-for-music-re-2.md`
- `20260401-020156-avoid-using-inline-code-blocks-e-g-bun-e-3.md`
- `20260401-175949-for-developing-robust-data-parsing-and-m-2.md`
- `20260402-200615-a-multi-stage-critique-workflow-utilizin-2.md`
- `20260402-200615-gemini-is-a-capable-tool-for-critiquing--1.md`
- `20260402-083610-formalizing-interaction-protocols-for-sp-1.md`
- `20260403-082246-the-strategic-direction-emphasizes-build-3.md`
- `20260403-163656-minimax-may-exhibit-timeout-issues-durin-2.md`
- `20260406-064001-multi-critic-reconciliation-is-highly-ef-1.md`
- `20260407-095424-minimax-has-a-strict-256-token-limit-tha-2.md`
- `20260408-031138-specifications-may-contain-outdated-assu-1.md`
- `20260408-031138-when-encountering-discrepancies-between--2.md`
- `20260408-145812-external-service-reliability-specificall-5.md`
