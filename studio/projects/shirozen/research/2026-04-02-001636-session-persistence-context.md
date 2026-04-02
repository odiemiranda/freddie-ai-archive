---
title: "Claude Code Session Persistence & Context Management"
type: research
project: shirozen
topic: storage-retention
date: 2026-04-02
sources:
  - notebook: 10f1a658 (Claude Code)
  - https://code.claude.com/docs/en/memory
tags: [session, persistence, context, memory, hooks, auto-memory, compaction]
---

# Claude Code Session Persistence & Context Management

## Session Persistence Architecture

### Disk-Based Transcripts
- Sessions saved as JSONL: `agent-{agentId}.jsonl`
- Complete message transcript
- Disabled with `persistSession: false`

### Session Branching (Git-like)
- Pause, checkpoint, resume (`claude -c` / `claude -r`)
- Fork sessions (`--fork-session`) for alternative paths
- **Relevant to Shirozen**: Session transcripts are the raw material for pattern detection

### Context Isolation via Subagents
- Subagents spawn with clean context window
- Execute own loops, return concise summary
- Prevents main session overflow during heavy exploration

## Context Management (Within Session)

### Auto-Compaction
- Triggers at ~50% context window capacity
- Condenses older turns, preserves critical decisions/code/errors
- **Summary Instructions** in CLAUDE.md control what to prioritize
- `PreCompact` / `PostCompact` hooks fire during transition

### Stateless API Reality
- Entire conversation history sent back on every turn
- Prompt caching + compaction strictly necessary
- Deeply chained subagent execution → API timeouts possible

## 6 Layers of Memory (Cross-Session)

Loaded at session start, spanning:
1. Enterprise policies
2. User preferences
3. Project rules (CLAUDE.md)
4. Local overrides
5. Auto-Memory (dynamic, 200 lines max)
6. Manual memory (user-pinned via `#` prefix)

### CLAUDE.md (Static Context)
- Declarative ground truth
- Enterprise → User → Project → Local levels
- Injected into context on every turn

### Auto-Memory (Dynamic State)
- Extracts observations: architectural patterns, conventions, directory structures
- Writes to persistent `MEMORY.md`
- Timestamps for staleness prioritization (recent update)
- First 200 lines loaded into system prompt

### Hooks (Lifecycle Events)
- `SessionStart` — set up env, fetch external context
- `SessionEnd` — teardown
- `PreCompact` / `PostCompact` — logging, cache optimization
- Deterministic shell scripts, fire at specific events

## Limitations

1. **Coupled memory toggles** — `--no-memory` disables ALL memory including CLAUDE.md
2. **One-way teleportation** — Can pull cloud agent to local, can't push local to cloud
3. **No team session resumption** — Agent Teams can't be restored after close
4. **Stateless API overhead** — Compaction may remove nuanced context

## Shirozen Implications

### What this means for Shirozen:
- **JSONL transcripts are gold** — Parse these for pattern mining input
- **Auto-compaction is the enemy** — Critical decisions may be lost during compaction; Shirozen should capture pre-compaction state
- **PreCompact hook = Shirozen's lifeline** — Use PreCompact to extract and save critical context before it's compressed
- **6 memory layers are static** — Shirozen adds a 7th: predicted/proactive context based on patterns
- **200-line auto-memory limit** — Shirozen's context streamer can push predicted context beyond this limit via channels

### Architecture Decision:
Shirozen should hook into:
1. `SessionStart` — Push predicted context based on recent patterns
2. `PreCompact` — Capture decision state before compression
3. Channels — Push real-time context updates during session
4. `SessionEnd` — Trigger reflection + pattern extraction
