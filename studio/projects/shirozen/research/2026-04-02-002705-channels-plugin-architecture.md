---
title: "Claude Code Channels — Custom Plugin Architecture for Shirozen"
type: research
project: shirozen
topic: communication
date: 2026-04-02
sources:
  - notebook: 4ca3782d (Claude Code Stream communication)
  - https://code.claude.com/docs/en/channels
  - https://www.atcyrus.com/stories/what-are-claude-code-channels
  - https://www.datastudios.org/post/claude-code-channels-what-it-is-how-it-works-and-how-to-use-it-with-mcp-telegram-and-discord
  - https://www.techbuzz.ai/articles/channels-is-the-missing-layer-in-claude-code-s-architecture
tags: [channels, mcp, plugin, push-events, notification, security, shirozen-transport]
---

# Claude Code Channels — Custom Plugin Architecture for Shirozen

## Architecture Overview

A Channel = MCP server spawned as subprocess (stdio) that pushes events into Claude's context.

### One-Way vs Two-Way
- **One-way**: External events → Claude Code (CI/CD alerts, monitoring, predicted context)
- **Two-way**: External events → Claude Code + reply tool → external platform (chat bridges)

### Key Capability Declarations
```javascript
// In MCP server constructor:
capabilities: {
  experimental: {
    'claude/channel': {},              // required for all channels
    'claude/channel/permission': {}    // optional: relay tool approval prompts
  }
}
```

### Instructions String
- Defined in server constructor
- Appended to Claude's system prompt automatically
- Tells Claude what incoming events mean, what metadata they carry, whether to reply

## MCP Notification Format

### Push Event
```javascript
server.notification({
  method: "notifications/claude/channel",
  params: {
    content: "The actual event body text",
    meta: {
      source: "shirozen",
      type: "predicted_context",
      confidence: "0.85"
    }
  }
});
```

### Injection Result
Claude sees: `<channel source="shirozen" type="predicted_context" confidence="0.85">The actual event body text</channel>`

### Meta Rules
- Keys: strict identifiers (letters, digits, underscores only)
- Invalid keys silently dropped
- Use for routing context: sender, chat_id, severity, type

## Permission Relay (Two-Way)

### Outbound (Claude → External)
```
notifications/claude/channel/permission_request
  → request_id, tool_name, description, input_preview
```

### Inbound (External → Claude)
```
notifications/claude/channel/permission
  → request_id, behavior: 'allow' | 'deny'
```

## Three-Layer Security Model

1. **Sender Allowlists** — Plugin implements strict sender checks before notification; unauthorized messages silently dropped
2. **Session-Level Opt-In** — Must launch with `--channels plugin:name@source`; not auto-loaded
3. **Plugin Whitelisting** — Anthropic-curated allowlist by default; `channelsEnabled` + `allowedChannelPlugins` policies for teams

### Development Bypass
```bash
claude --dangerously-load-development-channels server:shirozen
```

## Building Shirozen as a Channel Plugin

### Step-by-Step Architecture
1. **Initialize MCP Server** — Bun script, declare `claude/channel` capability
2. **Instructions** — "Predicted context via <channel> tag. Supplementary info for next prompt."
3. **HTTP/WS Listener** — Local server receives predictions from Shirozen engine
4. **Sender Gating** — Verify incoming requests from trusted Shirozen service (API key, X-Sender header)
5. **Emit Notification** — `server.notification()` with `notifications/claude/channel`
6. **Connect via Stdio** — `mcp.connect()` using stdio transport
7. **Launch** — `claude --dangerously-load-development-channels server:shirozen`

### Shirozen Channel Message Types
```typescript
type ShirozenNotification = {
  content: string;
  meta: {
    source: 'shirozen';
    type: 'predicted_context' | 'contradiction_alert' | 'pattern_match' | 'decision_feedback';
    confidence: string;  // "0.0" to "1.0"
    pattern_id?: string;
    related_entries?: string;  // comma-separated IDs
  };
};
```

### Context Push Triggers
| Trigger | When | What to Push |
|---------|------|--------------|
| Session start | `SessionStart` hook | Predicted workflow context based on recent patterns |
| Topic detection | User's first prompt | Relevant knowledge + past decisions for detected topic |
| Pre-compaction | `PreCompact` hook | Critical decisions/context before compression |
| Pattern match | During session | "This looks like X workflow — loading Y context" |
| Contradiction found | Background scan | "Conflicting knowledge detected: A vs B" |
