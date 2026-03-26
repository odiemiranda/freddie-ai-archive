---
name: Memory System Strategy
description: Priority hierarchy for agent memory and context management
type: knowledge
agent: freddie
tags: [memory, claude-mem, context, efficiency]
---

## Memory Hierarchy
1. **Primary (claude-mem):** Use for all standard reads/writes. It allows targeted queries that save context window tokens.
2. **Secondary (File-based):** `vault/studio/memory/freddie/` serves as a backup or for durable, long-form reference memories.

**Why:** Loading massive `MEMORY.md` files or directory listings into every turn is token-inefficient. `claude-mem` keeps the context lean by pulling only relevant fragments.

**How to apply:** Search `claude-mem` first. Only fall back to file-based memory if searching for specific historical documents or if `claude-mem` is unavailable.

*Consolidated from: feedback_memory_system.md*
