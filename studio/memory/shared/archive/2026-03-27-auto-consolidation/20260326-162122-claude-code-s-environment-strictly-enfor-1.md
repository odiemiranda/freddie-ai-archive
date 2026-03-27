---
source: reflection
agent: freddie
task: "Build tool registry system with native MCP tool calling"
workflow: research
outcome: success
scope: shared
confidence: high
tags: [claude-code, tool-calling, platform-limitations, architecture, mcp]
session: 2026-03-26T16:21:22
---

## Observation
During the refactoring to native MCP tool calling, it was discovered that Claude Code has no in-process mechanism for tool execution.

## Learning
Claude Code's environment strictly enforces MCP stdio as the *only* mechanism for tool execution. All tools must be exposed as external processes communicating via stdio, as there is no direct in-process tool calling capability.

## Evidence
discovered Claude Code has no in-process tool mechanism — MCP stdio is the only path
