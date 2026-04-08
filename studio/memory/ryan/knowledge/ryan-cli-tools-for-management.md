---
name: Ryan's CLI Tools for Agent and Staged Content Management
description: Details Ryan's development of command-line interface tools for inspecting agent state, reviewing staged content, validating prompt envelopes, and reconciling critic feedback.
type: knowledge
agent: ryan
tags: [cli, agent-management, staged-content, review, tools, testing, data-integrity, asl-0009, asl-0012, asl-0025, validation, prompt-engineering, critics, reconciliation]
---

## Knowledge
Ryan developed essential command-line interface (CLI) tools to facilitate the management and inspection of agent states, staged content, and prompt engineering artifacts within the system.

**Agent State CLI Tool (ASL-0009):**
Ryan implemented the `agent-state` CLI tool, located at `src/tools/agent-state.ts`, along with its comprehensive test suite (`src/tools/__tests__/agent-state.test.ts`). All 56 tests passed, verifying the tool's functionality. A critical read-only invariant was confirmed, ensuring that the hash of the agent state remains unchanged before and after executing any subcommands, thus guaranteeing data integrity during inspection. The implementation required no changes to `package.json` or core `src/libs` files.

**Review Staged CLI Tool (ASL-0012):**
The `review-staged` CLI tool was developed to manage staged proposals. All 38 tests passed, and its `list` and `show` functionalities were smoke tested successfully against 39 real proposals. The `approve` and `reject` functionalities were validated through fixture tests. The tool includes a TTY guard to ensure proper interactive behavior. It was confirmed that `review-staged.ts` does not use `writeFileSync`, preventing unintended direct file modifications, and the tool runner successfully auto-discovered it.

**Prompt Envelope Validation CLI Tool (ASL-0025 T1):**
Ryan created the `validate-envelope` CLI tool (`src/tools/validate-envelope.ts`) with 28 passing tests. This tool is designed to validate key components of musical prompts, including BGM/vocal envelope slots, BPM/Key tokens, and canonical section markers, ensuring adherence to structural requirements.

**Reconcile Critics CLI Tool (ASL-0025 T4):**
A `reconcile-critics.ts` CLI tool was implemented as part of the critic authority reversal mechanism. This tool facilitates the reconciliation of feedback from multiple critics, leveraging a new `src/libs/critic-reconciliation/` library.

## Why
These CLI tools provide essential utilities for developers and operators, enabling efficient command-line interaction with core system components. The `agent-state` tool is crucial for debugging and verifying agent configurations, while the `review-staged` tool streamlines the content review process. The `validate-envelope` tool ensures the structural integrity of musical prompts, and `reconcile-critics` is vital for managing and resolving conflicting feedback in the prompt engineering workflow.

## How to apply
Use the `agent-state` CLI tool for quick diagnostics and verification of agent internal states, configurations, and hashes. Utilize the `review-staged` CLI tool to efficiently manage the lifecycle of proposals, including listing, viewing, approving, and rejecting them. Employ the `validate-envelope` CLI tool to pre-check musical prompt structures for correctness before submission. Leverage the `reconcile-critics` CLI tool to manage and apply critic feedback, especially in scenarios involving conflicting or multi-source evaluations.

## Consolidated from
- `raw-log-20260408-084140-note.md`
- `raw-log-20260408-091500-note.md`
- `raw-log-20260409-015448-note.md`
- `raw-log-20260409-022407-note.md`
