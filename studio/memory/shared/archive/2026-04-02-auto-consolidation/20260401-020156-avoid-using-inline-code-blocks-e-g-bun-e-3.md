---
source: reflection
agent: freddie
project: unknown
projectPath: ""
task: "CLAUDE.md slim + eidetic memory raw layer"
workflow: dev-session
outcome: success
scope: shared
confidence: high
memoryType: semantic
tags: [tool_development, cli_first, user_preference, workflow_design, readability, maintainability, code_style]
session: 2026-04-01T02:01:56
raw_ref: 1774979909
---

## Observation
User feedback indicated a preference against inline `bun -e` code blocks in workflows, leading to the creation of a dedicated `write-raw.ts` CLI tool and the implementation of a `no-inline-code` rule across all skill workflows.

## Learning
Avoid using inline code blocks (e.g., `bun -e`) directly within workflows. Instead, encapsulate such logic into dedicated CLI tools to improve readability, maintainability, and align with user preference for explicit, well-defined tool calls. A `no-inline-code` rule should be enforced.

## Evidence
Created write-raw.ts CLI tool after user caught inline bun -e code blocks in workflows. Updated all 16 skill workflows + added 4 missing. Added no-inline-code rule.
