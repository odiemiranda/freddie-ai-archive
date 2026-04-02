---
title: "Claude Agent SDK — NDJSON Streaming Protocol & Message Types"
type: research
project: shirozen
topic: communication
date: 2026-04-02
sources:
  - https://platform.claude.com/docs/en/agent-sdk/streaming-output
  - https://platform.claude.com/docs/en/agent-sdk/streaming-vs-single-mode
  - https://github.com/anthropics/claude-code/issues/24594
  - https://gist.github.com/POWERFULMOVES/58bcadab9483bf5e633e865f131e6c25
  - https://sathwick.xyz/blog/claude-code.html
  - https://github.com/Yuyz0112/claude-code-reverse
tags: [ndjson, protocol, sdk, streaming, message-types, control-request]
---

# Claude Agent SDK — NDJSON Streaming Protocol & Message Types

## Protocol Overview

- SDK spawns Claude Code CLI as subprocess
- Communication: NDJSON (newline-delimited JSON) over stdin/stdout
- Each line = one complete JSON object with `type` field
- Bidirectional: prompts in via stdin, events out via stdout

## Message Types

### Output Messages (stdout)
| Type | Description |
|------|-------------|
| `system` | Session initialization data |
| `assistant` | Complete assistant response |
| `user` | User message echo |
| `result` | Final result of conversation |
| `stream_event` | Incremental streaming events (when enabled) |
| `compact_boundary` | Conversation history was compacted |

### Control Messages
| Type | Direction | Description |
|------|-----------|-------------|
| `control_request` | CLI → SDK | Permission request for tool use |
| `control_response` | SDK → CLI | Permission decision (allow/deny) |

### Control Request Format
```json
{
  "type": "control_request",
  "request_id": "unique-id",
  "request": {
    "subtype": "can_use_tool",
    "tool_name": "Bash",
    "input": { "command": "git status" },
    "decision_reason": "Running git status to check repository state",
    "tool_use_id": "tool-use-id"
  }
}
```

### Control Response Format
```json
{
  "type": "control_response",
  "request_id": "unique-id",
  "response": {
    "decision": "allow"
  }
}
```

## Streaming Configuration

### Enable Partial Messages
```python
# Python SDK
async for event in sdk.stream(prompt, include_partial_messages=True):
    if event.type == "stream_event":
        # Token-by-token streaming
        pass
    elif event.type == "assistant":
        # Complete response
        pass
```

### CLI Flags
- `--print` — Output mode (non-interactive)
- `--output-format stream-json` — NDJSON output
- `--input-format stream-json` — NDJSON input
- `--permission-prompt-tool stdio` — Enable control_request/control_response
- `--sdk-url ws://<server>` — WebSocket mode (hidden flag)

## Reverse-Engineered Architecture

### Direct Connect (server/)
- WebSocket-based protocol for external clients
- `DirectConnectSessionManager` class:
  - Opens WebSocket connections
  - Sends user messages
  - Handles tool permission prompts
  - Sends interrupts
  - Closes connections
- JSON-over-WebSocket messages

### Message Adaptation
- `bridgeMessaging.ts` / `inboundMessages.ts` — SDK format ↔ local format conversion
- 45+ tool implementations
- 100+ slash command implementations

## Shirozen Implications

### For Agent SDK Integration (Approach 2)
```typescript
import { spawn } from 'child_process';

const claude = spawn('claude', [
  '--print',
  '--output-format', 'stream-json',
  '--permission-prompt-tool', 'stdio'
]);

// Read NDJSON events
claude.stdout.on('data', (chunk) => {
  const lines = chunk.toString().split('\n').filter(Boolean);
  for (const line of lines) {
    const msg = JSON.parse(line);
    if (msg.type === 'control_request') {
      // Auto-approve or route to Shirozen for decision
      claude.stdin.write(JSON.stringify({
        type: 'control_response',
        request_id: msg.request_id,
        response: { decision: 'allow' }
      }) + '\n');
    }
    if (msg.type === 'assistant') {
      // Extract events for pattern mining
      logEvent(msg);
    }
  }
});

// Send prompt
claude.stdin.write(JSON.stringify({
  type: 'user',
  content: 'Analyze the codebase...'
}) + '\n');
```

### For Channel Integration (Approach 1)
The NDJSON protocol is used internally by the CLI. Channels use MCP notification protocol instead (documented in channels research). Shirozen should use Channels for human-facing sessions and Agent SDK NDJSON for headless batch jobs.
