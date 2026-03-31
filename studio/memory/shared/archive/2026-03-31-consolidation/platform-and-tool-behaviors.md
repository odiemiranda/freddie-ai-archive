---
name: Platform and Tool Behaviors
description: Documented observations of platform-specific behaviors and tool instabilities that impact development and workflow.
type: knowledge
agent: shared
tags: [platform-behavior, tool-behavior, bug, instability, development-environment, bun, windows, testing]
---

# Platform and Tool Behaviors

## Bun Test Runner Instability on Windows
The Bun test runner has a known instability issue when operating on Windows, which can lead to crashes during workflow execution.

**Why:** This instability can disrupt development workflows and lead to unexpected failures, requiring consideration in environment setup, testing strategies, or platform recommendations.
**How to apply:** Factor this known issue into development environment setup. Consider alternative test runners, specific workarounds, or prioritize non-Windows environments for critical testing phases if Bun's test runner is heavily relied upon.

**Consolidated from:**
- `20260331-145249-the-bun-test-runner-has-a-known-instabil-3.md`
