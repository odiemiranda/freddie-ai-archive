---
source: reflection
agent: freddie
task: "Build tool registry system with native MCP tool calling"
workflow: research
outcome: success
scope: self
confidence: high
tags: [tool-design, mcp, sdk-compatibility, data-structure, api-design]
session: 2026-03-26T16:21:22
---

## Observation
Tool results were made protocol-agnostic, and a `toMcp()` adapter was created to handle MCP SDK compatibility, specifically for `ZodObject` shapes.

## Learning
For robust tool development, design tool functions to return protocol-agnostic results (e.g., simple strings or structured data). Implement a dedicated adapter (e.g., `toMcp()`) to transform these internal results and data shapes (like `ZodObject.shape`) into the specific format required by the MCP SDK for tool server communication.

## Evidence
(3) made ToolResult protocol-agnostic (output string, not MCP content array), (4) added toMcp() adapter that extracts .shape from ZodObject for MCP SDK compatibility, (Gotchas: ... ZodObject vs plain shape for server.tool())
