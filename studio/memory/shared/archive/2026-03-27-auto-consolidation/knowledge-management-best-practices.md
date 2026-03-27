---
name: Knowledge Management Best Practices
description: Best practices and lessons learned for knowledge consolidation, especially concerning LLM token limits.
type: knowledge
agent: shared
tags: [knowledge-management, consolidation, llm, token-limits, data-integrity, best-practices]
---

# Knowledge Management Best Practices

## LLM Token Limits and Silent Data Loss
When using Large Language Models (LLMs) for structured, multi-file output, it is critical to be aware of `maxOutputTokens` limits. If the output exceeds this limit, the model will silently truncate its response, leading to incomplete or lost data without error notification.

**Why:** Prevents silent data loss and ensures the integrity and completeness of consolidated knowledge.
**How to apply:**
1.  **Increase `maxOutputTokens`:** Set the limit significantly higher than the expected output, providing ample headroom (e.g., 4x the estimated need).
2.  **Implement Post-Parse Checks:** After the LLM generates its output, compare the planned number of output files/sections with the actual parsed count. Warn if there's a mismatch.
3.  **Implement Post-Write Verification:** After writing files to disk, verify that the contents of the knowledge directory match the consolidation plan.

**Lesson:** Always verify the output count matches the plan when using LLMs for structured multi-file output. Token limits cause silent truncation.

**Consolidated from:**
- `consolidation-token-limit-lesson.md`
