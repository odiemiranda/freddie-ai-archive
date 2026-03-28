---
name: Tool Execution Strategy
description: Guidelines for prioritizing CLI-first tool calls for enhanced visibility and debuggability.
type: knowledge
agent: freddie
tags: [tool_execution, user_preference, debuggability, architecture, workflow]
---

## User Preference for Tool Calls
The user explicitly prefers CLI-first tool calls over MCP stdio calls for all internal operations.

## Principle
For all internal tool executions, whether by the main agent or any subagents, prioritize CLI-first calls.

**Why:** This approach significantly enhances visibility and debuggability of internal processes, aligning directly with user preference. It provides clearer logs and easier introspection into tool interactions.

**How to apply:** When designing or implementing tool calls within any agent or subagent workflow, ensure the primary method of execution is via CLI commands rather than direct stdio interfaces where a CLI alternative exists.

*Consolidated from: 20260328-074502-for-all-internal-tool-executions-main-ag.md*
