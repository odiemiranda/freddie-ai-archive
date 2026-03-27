---
source: reflection
agent: freddie
task: "Full-day session: Suno prompt overhaul, tool registry, tracknames tool"
workflow: research
outcome: success
scope: self
confidence: high
tags: [claude, sdk, tools, integration, zod, platform-behavior]
session: 2026-03-26T17:36:12
---

## Observation
During the integration of tools with the Claude Code Tools SDK, it was discovered that the SDK expects plain Zod shapes (not `z.object()`), numbers are passed as strings (requiring `z.coerce` for type safety), and the server name cannot match the plugin name.

## Learning
The Claude Code Tools SDK has specific, non-obvious requirements for tool schema definition (plain Zod shapes), data type handling (numbers as strings requiring `z.coerce`), and naming conventions (server vs. plugin name) that are critical for successful tool integration and must be explicitly accounted for.

## Evidence
Discovered MCP SDK expects plain zod shapes not z.object(), MCP passes numbers as strings (z.coerce needed), server name can't match plugin name.
