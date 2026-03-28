---
source: reflection
agent: freddie
task: "CLI-first tool call review"
workflow: review
outcome: success
scope: self
confidence: high
tags: [tool_execution, user_preference, debuggability, architecture]
session: 2026-03-28T07:45:02
---

## Observation
The user explicitly stated a preference for CLI-first tool calls over MCP stdio calls for internal operations.

## Learning
For all internal tool executions (main agent and subagents), prioritize CLI-first calls to enhance visibility and debuggability, aligning with user preference.

## Evidence
User confirmed MCP stdio calls work but prefer CLI for visibility and debuggability. Switched all tool calls to CLI-first for main conversation and subagents.
