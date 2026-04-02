---
title: "Shirozen Service Management — Background Daemon on Windows"
type: research
project: shirozen
topic: infrastructure
date: 2026-04-02
sources:
  - https://bun.com/docs/guides/ecosystem/pm2
  - https://dev.to/zakpie/why-i-stopped-using-pm2-and-built-my-own-bun-process-manager-4ehe
  - https://docs.claude-mem.ai/architecture/pm2-to-bun-migration
tags: [pm2, bm2, daemon, windows, service, process-manager]
---

# Shirozen Service Management — Background Daemon on Windows

## Options for Windows

### PM2 (Production Ready)
- Works on Windows (stable), Linux, macOS
- `pm2 start shirozen.ts --interpreter bun`
- Features: auto-restart, log management, monitoring, clustering
- Overhead: runs Node.js alongside Bun (two runtimes)

### BM2 (Bun-Native)
- Process manager written in Bun for Bun
- No Node.js dependency
- Significantly less memory overhead
- Newer, less battle-tested on Windows

### Windows Task Scheduler
- Native Windows service
- `schtasks /create /tn "Shirozen" /tr "bun run src/services/shirozen/index.ts" /sc onstart`
- No process manager overhead
- Less sophisticated restart/monitoring

### npm Scripts (Simple)
- `npm run shirozen:start` / `npm run shirozen:stop` / `npm run shirozen:status`
- Pattern already used in project: `npm run ob:{start,stop,status}` for Obsidian sync
- Could use same pattern for Shirozen

## Recommended Approach

**Phase 1**: npm scripts (like Obsidian headless pattern)
```json
{
  "scripts": {
    "shirozen:start": "pm2 start src/services/shirozen/index.ts --name shirozen --interpreter bun",
    "shirozen:stop": "pm2 stop shirozen",
    "shirozen:status": "pm2 status shirozen",
    "shirozen:logs": "pm2 logs shirozen"
  }
}
```

**Phase 2**: If PM2 overhead is a concern, migrate to BM2 or native Bun daemon

## Health Check Integration

### With existing brain.ts --check
- SessionStart hook already runs `brain.ts --check`
- Extend: also check if Shirozen service is running
- Auto-start if not running

### HTTP Health Endpoint
```
GET /api/health → { ok: true, uptime: 3600, patterns: 42, working_memory: 15 }
```

## Reference: claude-mem's PM2 Migration
- claude-mem migrated from PM2 to native Bun
- Documented at docs.claude-mem.ai/architecture/pm2-to-bun-migration
- Suggests Bun's native process management is maturing
