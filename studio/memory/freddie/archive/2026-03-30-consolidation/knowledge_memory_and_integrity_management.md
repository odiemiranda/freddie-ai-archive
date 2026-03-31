---
name: Memory and Knowledge Integrity Management
description: Guidelines and mechanisms for managing agent memory, ensuring knowledge integrity, and optimizing context through strategic compaction and confidence decay.
type: knowledge
agent: freddie
tags: [memory, compaction, knowledge_integrity, confidence, state_management, debugging, infrastructure, workflow_management, optimization, reflection]
---

## Strategic Memory Compaction
- **Automatic Triggers:** Memory compaction can now be initiated automatically when tool usage reaches predefined thresholds (e.g., 50 and 80 tool calls), in addition to explicit commands or time-based triggers.
- **Pre-Compaction State Save:** Before each memory compaction, the system saves current tool call counts, compaction counts, and timestamps to `out/session-state.json`.

## Knowledge Integrity
- **Contradiction Detection:** Before writing new learnings, the system proactively identifies potential contradictions with existing knowledge-tier files using FTS5 and marks them with a `contradicts:` field in the frontmatter.
- **Confidence Decay:** The system dynamically adjusts the perceived reliability of past decisions, with confidence linearly decaying over 90 days to a minimum of 30%.

**Why:** These features improve system efficiency by optimizing context, enhance reliability by preserving state for debugging, and maintain knowledge quality by flagging inconsistencies and prioritizing more recent or frequently reinforced information.

**How to apply:**
- Rely on automatic compaction triggers for context optimization based on activity levels.
- Utilize the `out/session-state.json` file for debugging, tracking compaction history, and understanding workflow context around memory optimizations.
- Review learnings flagged with `contradicts:` for potential reconciliation or deeper investigation into conflicting knowledge.
- Understand that the system's decisions will prioritize more recent or reinforced knowledge due to confidence decay, making them more contextually relevant.

*Consolidated from: 20260329-190824-before-each-memory-compaction-the-system-4.md, 20260329-190824-before-writing-new-learnings-the-system--2.md, 20260329-190824-memory-compaction-can-now-be-initiated-n-3.md, 20260329-190824-the-system-can-now-dynamically-adjust-th-1.md*
