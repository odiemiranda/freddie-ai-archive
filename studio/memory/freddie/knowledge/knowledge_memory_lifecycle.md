---
name: Memory and Knowledge Lifecycle
description: Three-tier automated memory management and context-efficient retrieval strategy
type: knowledge
agent: freddie
tags: [memory, consolidation, inbox, knowledge, archive, claude-mem]
---

## Memory Hierarchy
1. **Primary (claude-mem):** Use for standard reads/writes to save context window tokens.
2. **Knowledge Files:** `vault/studio/memory/freddie/knowledge_*.md` for durable, consolidated reference.
3. **Inbox:** Temporary storage for new findings before consolidation.
4. **Archive:** Long-term storage of processed inbox files and superseded knowledge.

## Automation Pipeline
- **Storage:** `tools/remember.ts` saves new findings to the `inbox`.
- **Consolidation:** `tools/consolidate-memory.ts` uses Gemini to cluster themes, resolve contradictions, and update `knowledge` files.
- **Cleanup:** Processed files move to `archive` via a copy+delete pattern (fixing Windows `renameSync` issues).

**Why:** Massive memory files are token-heavy. The 3-tier system keeps the "active" context lean while ensuring new findings don't contradict established rules.

**How to apply:** Use `remember` to log session findings. Run `npm run consolidate` to merge findings. Always search `claude-mem` before reading file-based knowledge.

*Consolidated from: knowledge_memory_strategy.md, 2026-03-25-session-summary.md*
