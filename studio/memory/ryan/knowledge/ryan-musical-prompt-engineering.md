---
name: Ryan's Musical Prompt Engineering and Critic Reconciliation
description: Details Ryan's work on enhancing musical prompt generation, validation, and critic reconciliation, including new CLI tools, structured output formats, and workflow updates for various agents.
type: knowledge
agent: ryan
tags: [music-generation, prompt-engineering, validation, critics, reconciliation, workflow, cli, suno, tala, rune, lyrics, asl-0025, envelopes, structured-json, conflict-resolution]
---

## Knowledge
Ryan has significantly advanced the system's capabilities in musical prompt engineering, focusing on structured input validation, enhanced generation strategies, and robust critic feedback reconciliation.

**Prompt Envelope Validation Tool (ASL-0025 T1):**
A new CLI tool, `src/tools/validate-envelope.ts`, was created with 28 passing tests. This tool is designed to validate critical elements within musical prompts, such as BGM/vocal envelope slots, BPM/Key tokens, and canonical section markers, ensuring that prompts adhere to predefined structural and semantic rules.

**Musical Strategy and Output Enhancements (ASL-0025 T2+T3):**
The `suno-strategy.test.ts` was extended with 5 new BGM envelope tests. Workflow documentation (`workflow.md P2-001`, `SKILL.md Phase 2 tables`) was updated to include Section markers and Style envelope rows, formalizing these elements in the prompt generation process. Additionally, a structured JSON output format section was added to `production-musical.md` and `production-effectiveness.md`, and a new `critic-prompts.test.ts` (11 tests) was created to validate critic prompt generation.

**Critic Authority Reversal and Reconciliation (ASL-0025 T4+T5):**
Ryan implemented a comprehensive system for critic authority reversal and N-way conflict reconciliation.
*   A new `src/libs/critic-reconciliation/` library (including types, parse, apply, conflicts, index modules) was developed with 47 passing tests.
*   A `src/tools/reconcile-critics.ts` CLI tool was created to manage this process.
*   Workflow documentation (`workflow.md P3-007b, P3-008, P3-009, P4-001, P4-002, P5-001`) was updated to reflect the new reconciliation protocols.
*   Agent-specific documentation (`tala.md soul`, `suno.md agent`, `rune.md`, `lyrics.md agent`) was updated to define critic authority boundaries, tool usage, and reconciliation protocols.
*   N-way conflict grouping was implemented in `apply.ts` using a union-find algorithm, with corresponding updates to `types.ts` (ConflictCandidate, candidates[]) and `conflicts.ts`.
*   Workflow updates for `create-track/workflow.md`, `create-track-variant/workflow.md` (including `validate-envelope` and authority notes), and `lyrics/workflow.md` (migrating to `reconcile-critics`, diff review, 5-option choice, `validate-envelope` gate) were completed.

## Why
These features significantly improve the quality, consistency, and control over musical prompt generation. The validation tool ensures structural integrity, reducing errors. Enhanced strategies and structured outputs lead to more predictable and higher-quality musical results. The critic authority reversal and reconciliation system is crucial for effectively managing diverse feedback, resolving conflicts, and ensuring that the final prompts align with desired creative and technical specifications.

## How to apply
Utilize the `validate-envelope` CLI tool to pre-check musical prompt structures, ensuring they meet required formats for BGM/vocal envelopes, BPM/Key, and section markers. Leverage the extended Suno strategies and structured JSON output formats to generate more sophisticated and consistent musical prompts. Employ the `reconcile-critics` CLI tool and the underlying `critic-reconciliation` library to manage and resolve conflicting feedback from multiple critics, following the updated workflow and agent documentation for `tala`, `suno`, `rune`, and `lyrics` agents to ensure coherent and high-quality prompt refinement.

## Consolidated from
- `raw-log-20260409-015448-note.md`
- `raw-log-20260409-020432-note.md`
- `raw-log-20260409-022407-note.md`
- `raw-log-20260409-023909-note.md`
