---
name: mccall Document and Artifact Management
description: Describes the standardized system for managing project documentation and artifacts.
type: knowledge
agent: mccall
tags: [document-management, project-management, naming-conventions, version-control]
---

## Knowledge
A standardized and structured document management system has been implemented for project artifacts. This includes:
*   **Dual-location storage**: `vault/studio/projects/` for internal project files and `docs/` for external documentation.
*   **Context-specific naming conventions**: Timestamped names for technical and task-related documents (`tech/`, `tasks/`), and fixed names for core project documents (`BRIEF/`, `PRD/`, `ARCHITECT/`).
*   **End-of-workflow snapshots**: An 'Option C snapshot' mechanism is used on `dev-end` to capture the project state.
*   **Path management**: `resolveProjectDocsPath()` and `timestampedDocName()` functions are used for consistent path resolution and naming.

## Why
Consistent document management ensures project clarity, traceability, and proper separation of internal versus external artifacts. This system is critical for organizing project state, facilitating collaboration, and maintaining an accurate historical record.

## How to apply
Adhere strictly to the established dual-location storage for all project artifacts. Utilize the specified naming conventions (timestamped for dynamic content, fixed for stable documents) to maintain order. Ensure the 'Option C snapshot' is triggered at the end of relevant workflows to preserve project state. Leverage the defined path management functions for all document operations.

## Consolidated from
- `20260330-155239-standardized-and-structured-document-man-2.md`
