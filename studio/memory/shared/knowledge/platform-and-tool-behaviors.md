---
name: Platform and Tool Behaviors
description: Documented observations of platform-specific behaviors and tool instabilities that impact development and workflow.
type: knowledge
agent: shared
tags: [platform-behavior, tool-behavior, bug, instability, development-environment, bun, windows, testing, tool_limitation, lyrics_analysis, false_positive]
---

# Platform and Tool Behaviors

## Bun Test Runner Instability on Windows
The Bun test runner has a known instability issue when operating on Windows, which can lead to crashes during workflow execution.

**Why:** This instability can disrupt development workflows and lead to unexpected failures, requiring consideration in environment setup, testing strategies, or platform recommendations.
**How to apply:** Factor this known issue into development environment setup. Consider alternative test runners, specific workarounds, or prioritize non-Windows environments for critical testing phases if Bun's test runner is heavily relied upon.

## Section Balance Tool Limitation with Typed Brackets
The `section-balance` tool has a known limitation where it misinterprets typed brackets (e.g., `[Chorus]`) as structural elements, leading to incorrect balance assessments and false-positives.

**Why:** This limitation can lead to inaccurate analysis of lyrical structure, potentially misguiding prompt adjustments or workflow decisions.
**How to apply:** Be aware of this behavior when using the `section-balance` tool. Consider manual verification or alternative methods for balance assessment when lyrics contain typed brackets.

**Consolidated from:**
- `20260331-145249-the-bun-test-runner-has-a-known-instabil-3.md`
- `20260405-124725-the-section-balance-tool-has-a-specific--2.md`
