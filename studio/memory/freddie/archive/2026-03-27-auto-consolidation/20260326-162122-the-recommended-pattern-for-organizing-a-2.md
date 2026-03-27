---
source: reflection
agent: freddie
task: "Build tool registry system with native MCP tool calling"
workflow: research
outcome: success
scope: self
confidence: high
tags: [tool-registry, architecture, best-practice, code-structure, mcp]
session: 2026-03-26T16:21:22
---

## Observation
A `ToolRegistry` system was built with auto-discovery and robust path management for native MCP tool calling.

## Learning
The recommended pattern for organizing and exposing tools in the Claude Code environment is a `ToolRegistry` that features auto-discovery via `toolDef` exports and uses a centralized `paths.ts` module for robust, shared path management, replacing fragile `__dirname` hops.

## Evidence
(1) restructured into src/ with shared paths.ts replacing all fragile __dirname hops, (2) built ToolRegistry with auto-discovery via pre-scan for toolDef exports
