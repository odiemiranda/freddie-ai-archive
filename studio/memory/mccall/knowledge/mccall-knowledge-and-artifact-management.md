---
name: mccall Knowledge and Artifact Management
description: Describes the standardized system for managing project documentation, artifacts, and decisions.
type: knowledge
agent: mccall
tags: [knowledge-management, document-management, project-management, naming-conventions, version-control, decision-logging, brain.db, workflow-process]
---

## Knowledge
A standardized and structured system has been implemented for managing project knowledge, documents, and artifacts. This system includes:
*   **Dual-location storage**: `vault/studio/projects/` for internal project files and `docs/` for external documentation.
*   **Context-specific naming conventions**: Timestamped names for technical and task-related documents (`tech/`, `tasks/`), and fixed names for core project documents (`BRIEF/`, `PRD/`, `ARCHITECT/`).
*   **End-of-workflow snapshots**: An 'Option C snapshot' mechanism is used on `dev-end` to capture the project state.
*   **Path management**: `resolveProjectDocsPath()` and `timestampedDocName()` functions are used for consistent path resolution and naming.
*   **Decision logging**: `brain.db` is utilized as a dedicated system for logging important decisions, ensuring they are recorded and retrievable for future reference, complementing general document storage.

## Why
Consistent knowledge and artifact management ensures project clarity, traceability, and proper separation of internal versus external artifacts. This system is critical for organizing project state, facilitating collaboration, maintaining an accurate historical record, and ensuring critical decisions are systematically recorded and accessible.

## How to apply
Adhere strictly to the established dual-location storage for all project artifacts. Utilize the specified naming conventions (timestamped for dynamic content, fixed for stable documents) to maintain order. Ensure the 'Option C snapshot' is triggered at the end of relevant workflows to preserve project state. Leverage the defined path management functions for all document operations. Ensure all significant decisions are logged to `brain.db` for comprehensive historical and operational context.

## Consolidated from
- `20260330-155239-standardized-and-structured-document-man-2.md`
- `20260331-060545-mccall-utilizes-brain-db-as-a-dedicated--2.md`
