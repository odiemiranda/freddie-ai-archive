---
title: "Claude Code WebSocket Communication Protocol"
type: research
project: shirozen
topic: communication
date: 2026-04-02
sources:
  - notebook: 4ca3782d (Claude Code Stream communication)
  - https://github.com/Kurogoma4D/claude-code-server
  - https://github.com/gvorwaller/claude-relay
  - https://github.com/dzhng/claude-agent-server
  - https://code.claude.com/docs/en/sub-agents
  - https://buildwithaws.substack.com/p/inside-the-claude-agent-sdk-from
tags: [websocket, protocol, channels, remote-control, mcp, stdio]
---

# Claude Code WebSocket Communication Protocol

## Key Finding: Three Communication Layers

### 1. Reverse-Engineered WebSocket Protocol (NDJSON)
- Hidden `--sdk-url` flag switches CLI from terminal mode into WebSocket client
- Launch: `claude --sdk-url ws://<server-address> --print --output-format stream-json`
- Protocol: **NDJSON (newline-delimited JSON)**
- **13 control request subtypes**: `initialize`, `can_use_tool`, `interrupt`, `set_model`, MCP operations
- Bidirectional: user prompts → CLI, streaming events + session lifecycle + permission flows → client
- Token-by-token streaming supported

### 2. IDE Integrations — Local WebSocket (JSON-RPC 2.0)
- VS Code extension: local WebSocket server on `localhost`
- Standard JSON-RPC 2.0 messages: `ping`, `tools/list`, `tools/call`
- **Security note**: CVE-2025-52882 — initially lacked auth, patched with auth token in lock file

### 3. Channels System — Push Events into Running Sessions
- MCP server spawned as subprocess via `stdio`
- External service → MCP server's HTTP/WS port → `mcp.notification()` → `notifications/claude/channel`
- Payload: `content` (string) + optional `meta` dict (chat_id, severity)
- Injected as `<channel source="my-server" chat_id="123">...</channel>` XML tag
- **Permission relay**: outbound `notifications/claude/channel/permission_request` with `request_id`, `tool_name`, `description`, `input_preview`
- Response: `notifications/claude/channel/permission` with `request_id` + `behavior` (allow/deny)

### Remote Control Feature
- `claude remote-control` — native feature
- Controls local terminal from phone/tablet/browser via claude.ai
- Transfers session state as single atomic unit through **encrypted tunnel**
- Includes file references, conversation history, tool configs, pending operations

## Three Approaches for Persistent Service Communication

### Approach 1: MCP Channels (Anthropic's native solution)
- Claude Code spawns MCP server as subprocess (stdio)
- MCP server runs own HTTP/WS listener for external events
- Push via `notifications/claude/channel` → `<channel>` XML tag
- Reply via declared MCP tool (e.g., "reply" tool)
- **Best for**: Human at terminal, assistant-to-the-assistant

### Approach 2: Agent SDK (Subprocess spawning)
- Persistent service spawns Claude Code CLI as child process
- Flags: `--print`, `--output-format stream-json`
- Bidirectional via stdin/stdout streaming JSON
- Total control over session lifecycle
- **Best for**: Headless orchestrator, cloud service, web dashboard, bot

### Approach 3: HTTP Hooks (Reactive only)
- `settings.json` with `type: "http"` hooks
- POST requests on lifecycle events: `UserPromptSubmit`, `PreToolUse`, `PostToolUse`
- Returns `allow`/`deny`, updated tool inputs, or `additionalContext`
- **Limitation**: Cannot initiate — entirely reactive, waits for trigger

## Shirozen Implications

For Shirozen as a persistent cognitive service:
- **Approach 1 (MCP Channels)** for human-facing sessions — push predicted context into active terminal
- **Approach 2 (Agent SDK)** for headless background analysis — spawn Claude for batch processing
- **Approach 3 (HTTP Hooks)** for session lifecycle — inject context at startup, validate decisions
- **Hybrid recommended**: Channels for real-time push + Hooks for lifecycle injection + Stdio for batch

## Key Repos
- [claude-code-server](https://github.com/Kurogoma4D/claude-code-server) — WebSocket + Socket.io, manages child processes
- [claude-relay](https://github.com/gvorwaller/claude-relay) — Inter-instance WS + MCP communication
- [claude-agent-server](https://github.com/dzhng/claude-agent-server) — E2B sandbox, WS bidirectional
- [companionfork](https://github.com/joshuaworth/companionfork) — Reverse-engineered WS protocol web UI
