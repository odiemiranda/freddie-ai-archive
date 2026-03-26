---
name: Obsidian Vault & Daily Note Conventions
description: Core knowledge for interacting with the existing Obsidian vault structure and formatting daily logs.
type: knowledge
agent: penny
tags: [obsidian, vault-structure, daily-notes, formatting]
---

# Obsidian Vault & Daily Note Conventions

## Vault Structure and Templates
The vault was established with a pre-defined folder structure and template system prior to Penny's integration. Adhering to these existing conventions is critical to avoid duplication or overriding the user's manual configuration.

**Why:** Using established patterns ensures consistency and prevents friction with the user's manual Obsidian workflows.
**How to apply:**
- Always check `vault/templates/` for the latest version of any template (`daily.md`, `meeting.md`, `doc.md`, `inbox.md`, `project.md`, `resource.md`) before creating new notes.
- Use the standard folder structure: `daily/`, `meetings/`, `weekly/`, `inbox/`, `projects/`, `resources/`, `docs/`.

## Daily Log Formatting
The "Log" section of daily notes follows a specific "digested summary" style rather than raw activity dumps.

**Why:** The user requires a scannable overview of the day's "shape" and high-level outcomes without being overwhelmed by verbose or copy-pasted details.
**How to apply:**
- Write each entry as a single-line bullet point.
- Capture the "what happened" and "why/outcome" concisely.
- Group related items together to maintain scannability.
- Avoid copy-pasting raw terminal output or verbose tool logs.

**Consolidated from:**
- `2026-03-22-230000-daily-log-style.md`
- `2026-03-22-230000-first-session-setup.md`
