---
name: Freddie AI Platform-Specific Operational Issues
description: Documents known platform-specific operational issues relevant to the Freddie AI project.
type: knowledge
agent: mccall
tags: [freddie-ai, platform-behavior, bun, windows, testing, bug, development-environment]
---

## Knowledge
The Bun test runner has a known instability issue when operating on Windows environments, which can lead to crashes during workflow execution. This is identified as a bug external to the Freddie AI codebase.

## Why
Understanding platform-specific instabilities is critical for effective environment setup, debugging, and development strategy. Acknowledging known external bugs prevents misattribution of issues and guides decisions on testing strategies or platform recommendations.

## How to apply
When setting up or troubleshooting development environments on Windows, be aware of the Bun test runner's instability. Factor this into testing strategies (e.g., using alternative runners, isolating tests, or running tests on Linux/macOS environments) or consider platform recommendations for future development. Do not spend time debugging Bun test runner crashes on Windows, as it is a known external issue.

## Consolidated from
- `20260331-145249-the-bun-test-runner-has-a-known-instabil-3.md`
