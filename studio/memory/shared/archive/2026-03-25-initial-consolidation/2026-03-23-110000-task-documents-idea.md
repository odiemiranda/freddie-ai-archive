---
id: mem-shared-20260323-110000-02
title: Persistent Task Documents in Vault
created: 2026-03-23 11:00:00
tags: [obsidian, tasks, vault, penny, architecture]
type: decision
agent: shared
connected: []
---

User idea: dedicated `vault/notes/tasks/` folder for persistent task documents. Tasks that span multiple days get a single source-of-truth doc instead of being duplicated/carried across daily notes.

**Concept:**
- Task doc: `vault/notes/tasks/ot-1598-tixie-auth.md` with status, description, sub-tasks, linked dailies
- Daily notes reference the task doc via wikilink: `- [ ] [3] [[notes/tasks/ot-1598-tixie-auth|OT-1598]]`
- Task doc tracks which dailies worked on it (bidirectional linking)
- Sub-tasks can live in the task doc or inline in dailies (TBD)

**Benefits:**
- Task history across days without duplication
- Click task → see all days it was touched
- Penny can manage via new MCP tool (`penny_task`)
- Obsidian graph shows task → daily relationships

**Status:** Idea documented. User is still thinking about the approach. Needs design decisions on: sub-task ownership (task doc vs daily), status field format, how Penny creates/updates task docs, how daily task entries link vs embed.

**How to apply:** Build alongside or after the studio index tool. Both are next-session tasks.
