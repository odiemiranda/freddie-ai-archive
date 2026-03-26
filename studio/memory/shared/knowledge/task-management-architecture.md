---
name: Task Management Architecture
description: Persistent task tracking using Obsidian and Penny.
type: knowledge
agent: shared
tags: [tasks, obsidian, penny, architecture]
---

# Task Management Architecture

## Persistent Task Documents
Tasks that span multiple sessions move out of transient daily notes and into dedicated source-of-truth documents.

### Concept
- **Location:** `vault/notes/tasks/`.
- **Format:** `OT-####-description.md` containing status, detailed description, and sub-tasks.
- **Linking:** Daily notes reference these via Wikilinks (e.g., `[[notes/tasks/OT-123|Task Name]]`).
- **History:** The task doc tracks which daily notes have "touched" the task via bidirectional links.

### Penny Integration
Penny will manage these via a specialized tool (`penny_task`). This allows for:
- Automatic creation of task docs from daily note items.
- Status updates that persist across the Obsidian graph.
- Visualizing task-to-day relationships in the Obsidian Graph View.

**Status:** Under design. Implementation follows the Studio Index tool.

**Consolidated from:**
- `2026-03-23-110000-task-documents-idea.md`
