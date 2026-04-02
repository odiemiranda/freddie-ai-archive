---
title: "Bun HTTP + WebSocket Server Architecture for Shirozen"
type: research
project: shirozen
topic: communication
date: 2026-04-02
sources:
  - https://bun.com/docs/runtime/http/websockets
  - https://github.com/kriasoft/bun-ws-router
  - https://dev.to/robertobutti/websocket-with-javascript-and-bun-4o7c
  - https://oneuptime.com/blog/post/2026-01-31-bun-websocket-servers/view
  - https://code.claude.com/docs/en/hooks
  - https://github.com/anthropics/claude-code/issues/15923
  - https://claudefa.st/blog/tools/hooks/context-recovery-hook
tags: [bun, websocket, http, server, hooks, precompact, architecture]
---

# Bun HTTP + WebSocket Server Architecture for Shirozen

## Bun.serve() — Dual Protocol on Same Port

### Core Pattern
```typescript
const server = Bun.serve({
  port: 3456,

  // HTTP routes
  fetch(req, server) {
    const url = new URL(req.url);

    // WebSocket upgrade
    if (url.pathname === "/ws") {
      const upgraded = server.upgrade(req, {
        data: { sessionId: url.searchParams.get("session") }
      });
      if (upgraded) return undefined;
      return new Response("WebSocket upgrade failed", { status: 400 });
    }

    // HTTP API routes
    if (url.pathname === "/api/query") return handleQuery(req);
    if (url.pathname === "/api/predict") return handlePredict(req);
    if (url.pathname === "/api/health") return Response.json({ ok: true });

    return new Response("Not found", { status: 404 });
  },

  // WebSocket handlers
  websocket: {
    open(ws) {
      ws.subscribe("context-updates");
    },
    message(ws, message) {
      // Handle incoming messages from Claude Code sessions
      handleSessionMessage(ws, JSON.parse(message));
    },
    close(ws) {
      ws.unsubscribe("context-updates");
    },
  },
});
```

### Key Bun WebSocket Features
- **Native performance**: Written in Zig, optimized memory management
- **Built-in pub/sub**: `ws.subscribe("topic")` / `server.publish("topic", data)`
- **Zero dependencies**: No Socket.io, no ws package needed
- **Same port**: HTTP and WebSocket coexist on single port
- **TypeScript support**: Full type safety with `data` generic

### Known Issue
- TypeScript types currently disallow `routes` + `websocket` together (oven-sh/bun#17871)
- Workaround: Use `fetch` handler with manual routing instead of `routes`

## Shirozen Service Architecture

```
┌──────────────────────────────────────────────────┐
│  Shirozen Engine (Bun.serve on port 3456)        │
│                                                  │
│  HTTP Routes:                                    │
│  POST /api/query      → brain.db search          │
│  POST /api/predict    → pattern-based prediction  │
│  POST /api/event      → log event to event_log   │
│  POST /api/feedback   → decision feedback         │
│  GET  /api/health     → health check              │
│  GET  /api/patterns   → list active patterns      │
│  GET  /api/working-memory → current warm cache    │
│                                                  │
│  WebSocket /ws:                                  │
│  → Real-time context push to sessions            │
│  → pub/sub: "context-updates" topic              │
│  → Session registration + heartbeat              │
│                                                  │
│  Stdio (MCP Channel):                            │
│  → Channel plugin for Claude Code integration    │
│  → notifications/claude/channel for context push │
└──────────────────────────────────────────────────┘
```

## Hook Integration

### PreCompact Hook — Critical for Shirozen
```json
{
  "hooks": {
    "PreCompact": [{
      "type": "command",
      "command": "bun src/tools/shirozen/pre-compact-capture.ts"
    }]
  }
}
```

**What it does:**
- Fires right before auto-compaction
- Captures critical context (active decisions, current workflow state)
- Sends to Shirozen service via HTTP POST /api/event
- Shirozen stores in working memory for re-injection after compaction

### PostCompact Hook — Verification
```json
{
  "hooks": {
    "PostCompact": [{
      "type": "command",
      "command": "bun src/tools/shirozen/post-compact-verify.ts"
    }]
  }
}
```

**What it does:**
- Reads `compact_summary` from hook data
- Checks if critical context survived compaction
- If missing, pushes recovered context via channel notification

### SessionStart Hook — Context Prediction
```json
{
  "hooks": {
    "SessionStart": [{
      "type": "command",
      "command": "bun src/tools/shirozen/session-start-predict.ts"
    }]
  }
}
```

**What it does:**
- Queries Shirozen service for predicted context
- Outputs predicted workflow + relevant knowledge
- Injected into session startup context

### Existing Hooks to Extend
- `tool-call-counter.ts` → Add event_log writes
- `brain.ts --check` → Also warm working memory cache
- `pre-compact-save.ts` → Already captures some state; extend for Shirozen

## Transport Decision Matrix

| Use Case | Transport | Why |
|----------|-----------|-----|
| Session start context | HTTP (hook) | One-shot, hook fires and exits |
| Real-time context push | WebSocket | Persistent connection, low latency |
| Tool call logging | HTTP (hook) | PreToolUse/PostToolUse hooks |
| Pattern mining (batch) | HTTP API | Scheduled job, request/response |
| Channel plugin | Stdio (MCP) | Anthropic's native integration |
| Inter-session relay | WebSocket pub/sub | Multiple sessions, broadcast |
