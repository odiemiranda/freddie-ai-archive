---
name: Memory system priority
description: Use claude-mem plugin as primary memory, file-based vault memory as backup. Saves context window and tokens.
type: feedback
agent: freddie
tags: [memory, claude-mem, context, tokens, workflow]
---

Use claude-mem as the primary memory system for reads and writes. File-based memory (vault/studio/memory/freddie/) serves as backup.

**Why:** Loading MEMORY.md index into context every conversation wastes tokens. claude-mem queries are targeted — pull only what's relevant, lighter on the context window.

**How to apply:** Search claude-mem first when recalling past context. Save important observations to claude-mem. Only fall back to file-based memory if claude-mem doesn't have what's needed, or for durable reference memories that should persist in the vault.
