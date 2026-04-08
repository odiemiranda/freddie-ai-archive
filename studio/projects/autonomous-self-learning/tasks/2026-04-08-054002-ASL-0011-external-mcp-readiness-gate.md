---
id: ASL-0011
title: External MCP readiness gate — settings flag + startup check
project-code: ASL
status: planned
priority: P1
created: 2026-04-08
author: mccall
implementer: ryan
reviewer: mccall
depends-on: [ASL-0001, ASL-0008]
blocks: []
files:
  - .claude/asl-readiness.json
  - .claude/hooks/startup.ts
  - src/libs/mcp-readiness.ts
  - src/libs/__tests__/mcp-readiness.test.ts
  - src/libs/process.ts
  - src/libs/__tests__/process.test.ts
  - .claude/rules/architecture.md
estimated-complexity: small-medium (2-3 hours)
tags: [task, mcp, readiness, startup, observability, asl-phase-2]
---

# ASL-0011 — External MCP readiness gate

## TL;DR

Add a session-start readiness probe for the **external** plugin-installed MCP servers this project depends on (currently `discord` and `claude-mem`). The probe is **opt-in** via a config flag; when enabled, `.claude/hooks/startup.ts` appends a per-MCP status block to the session-context briefing alongside the existing ASL daemon liveness section (ASL-0008). Probes are **never blocking**: they classify each MCP as `READY`, `UNREACHABLE`, `STALE`, or `DISABLED`, surface a one-liner remediation hint per failure, and let the user decide whether to act.

The probe **does not** auto-start MCPs, **does not** gate tool calls at PreToolUse, and **does not** modify per-tool error handling. It is a single user-visible "what's alive before I start working" check, pattern-matched to the ASL daemon probe shipped in ASL-0008 and reusing the `errorType` classification idiom from ASL-0001's `checkJudgeHealth`.

**In addition to the readiness gate, this task also delivered a reusable platform-agnostic process library (`src/libs/process.ts`)** as part of the refactor. Process enumeration, liveness checks, and kill primitives were extracted out of `mcp-readiness.ts` so they can be consumed by other call sites. See Follow-ups for the migration task.

**Hard locks:**
- Do NOT spawn or start any MCP server. The probe is read-only.
- Do NOT add a PreToolUse hook that blocks `mcp__*` tool calls. Out of scope.
- Do NOT touch `.mcp.json` (it's empty in this project — MCPs come from plugin installs).
- Do NOT add network calls to direct CLI tools (`gemini`, `grok`, `minimax`). Those are NOT MCP-backed and have their own pre-flight in `judge-panel.checkJudgeHealth` (ASL-0001).
- Do NOT default the flag to `true`. First release ships **opt-in only** so the probe can't add latency or spurious warnings to users who don't care.

---

## Context

### Why this task exists

The ASL Phase 2 readiness checklist (BRIEF §7) lists `external_mcp_ready` as a required flag before exposing pipeline tools to external clients. ASL-0001 closed the judge-health side (Gemini/MiniMax/Grok pre-flight). ASL-0008 added a daemon liveness probe to SessionStart so the user knows whether the background learning loop is running. ASL-0011 closes the analogous gap for **third-party MCP servers** that this project's plugin layer depends on.

**The problem:** When an external MCP is unreachable (plugin not installed, server crashed, IPC stale), tools that depend on it fail mid-workflow with no early warning. The user discovers the breakage three steps into a `/discord` reply or a memory recall through `claude-mem`. We want a startup-time signal: "Discord MCP is dead — restart the plugin or skip the reply tool today."

**Inventory of currently-installed external MCPs** (from `C:\Users\odiem\.claude-team\plugins\installed_plugins.json` 2026-04-08):

| Plugin | MCP server name | Scope | What it provides | In scope? |
|---|---|---|---|---|
| `discord@claude-plugins-official` | `discord` | local (this project) | Reading/replying in user's Discord channels | YES |
| `claude-mem@thedotmack` | `mcp-search` | local (this project) | Cross-session memory recall via plugin | YES |
| `freddie-ai@freddie-ai-local` | (this repo's MCP) | user-global | Penny + time + registry tools | **NO** — local, not external |

The discord MCP and claude-mem MCP both spawn as child processes via `bun run --cwd ${CLAUDE_PLUGIN_ROOT}` and communicate over stdio. They are "external" in the sense that this codebase did not author them and cannot fix them — when they break, the user must reinstall, restart, or wait. That's the failure class this gate covers.

### What "external" means in this project

Boundary defined for this task:

| Tool category | MCP-backed? | Covered by this gate? | Why |
|---|---|---|---|
| `freddie-ai` MCP server (`src/servers/freddie-ai/`) | Yes (we own it) | NO | Local, in-tree, has its own daemon lifecycle (`npm run mcp:*`) |
| `discord` plugin MCP | Yes (third-party) | YES | Spawned by Claude Code from plugin cache; we cannot debug it |
| `claude-mem` plugin MCP (`mcp-search`) | Yes (third-party) | YES | Same as above |
| `gemini` CLI (`bun run tool gemini`) | NO — direct CLI | NO | Direct subprocess; covered by `checkJudgeHealth` in pipeline contexts |
| `grok` CLI | NO — direct CLI | NO | Same |
| `minimax` CLI | NO — direct CLI | NO | Same |
| NotebookLM (`bun run tool research`) | NO — direct CLI | NO | Direct API client |
| Bash/Read/Write/Grep/etc. | NO | NO | Built-in |

**Decision:** "External MCP" = any entry in `~/.claude-team/plugins/installed_plugins.json` whose `projectPath` matches `fromRoot()` AND whose plugin ships an `.mcp.json`. The probe library auto-discovers them — no hard-coded list in the source. New plugins added later are picked up automatically on next session start.

### Scoping calls (the six critical questions)

| # | Question | Decision |
|---|---|---|
| 1 | Which MCPs count as "external"? | Plugin-installed MCPs whose `projectPath` matches the current project root. Auto-discovered from `~/.claude-team/plugins/installed_plugins.json` + each plugin's `.mcp.json`. The local `freddie-ai` MCP is excluded by name. |
| 2 | What does "ready" mean per MCP? | **Process-existence probe only** for first release. We check whether a child process matching the plugin's `command + args` signature is alive (parent-pid descended from Claude Code OR a PID file the plugin maintains). **No JSON-RPC ping.** Reason: each MCP has different RPC handlers and we don't own the protocol surface; a generic ping would require authoring per-MCP probes which is not worth it for the first release. Future enhancement: add a per-MCP `probe()` adapter registry if process-existence proves too coarse. |
| 3 | Where does the config flag live? | **`.claude/asl-readiness.json`** (NOT `.claude/settings.json` — see ADR below). Shape: `{ "externalMcp": { "enabled": false, "ignore": [] } }`. Default `enabled: false`. `ignore` is an array of plugin names the user wants suppressed (e.g., `["discord"]` if they don't use it but it's installed). |
| 4 | What does the check DO when not ready? | **Display warning only.** Match ASL-0008's daemon probe pattern exactly. Never block session start. Never gate tool calls. The user reads the briefing and decides whether to act. |
| 5 | Where does the check run? | `.claude/hooks/startup.ts` only. NOT in PreToolUse. NOT inside individual tools. Single surface = single source of truth. |
| 6 | Reuse vs new code? | **Reuse the structural pattern of `checkJudgeHealth`** (ASL-0001 `src/libs/judge-panel.ts:261`) — `Promise.allSettled` over a probe array, classify each result with an `errorType`, return a `{ alive, dead, checkedAt }` shape. **Do not** import `judge-panel.ts` itself — different domain, different result types. New file `src/libs/mcp-readiness.ts` mirrors the idiom but stays independent. |

### ADR — Config file location (`.claude/asl-readiness.json`, not `.claude/settings.json`)

**Decision:** ASL-tier user-facing feature flags live in dedicated `.claude/asl-*.json` files, NOT as new top-level keys on `.claude/settings.json`.

**Why:** Claude Code's runtime validator rejects unrecognized top-level keys in `.claude/settings.json` with `Unrecognized field: <key>`. The file has a strict allowlist. Adding `aslReadiness` there produced a hard validation error on session start. We cannot rely on the ability to add project-scoped ASL flags to `settings.json` going forward.

**Convention:** Any future ASL user-facing flag follows this pattern:
- File: `.claude/asl-{feature}.json`
- Shape: `{ "{feature}": { "enabled": boolean, ...flags } }`
- Loader in `src/libs/{feature}.ts` must accept an optional `configPath` argument so tests can use synthetic paths.
- Default state when file is missing, empty, or malformed is `enabled: false`.

**Inventory of current ASL config files:**

| File | Feature | Loader |
|---|---|---|
| `.claude/asl-readiness.json` | External MCP readiness probe | `src/libs/mcp-readiness.ts :: loadReadinessConfig` |

Future ASL flags (agent-state enable, review-staged toggles, etc.) follow this same convention.

### Read these before touching anything

1. **`vault/studio/projects/autonomous-self-learning/BRIEF.md`** — §7 "External MCP readiness checklist". This task fills the last unchecked structural box: `external_mcp_ready` flag.
2. **`vault/studio/projects/autonomous-self-learning/tasks/2026-04-07-181755-ASL-0001-minimax-parser-fix.md`** — `checkJudgeHealth` ADR + the `Promise.allSettled` + classify pattern. Reuse the **shape**, not the file.
3. **`vault/studio/projects/autonomous-self-learning/tasks/2026-04-08-051140-ASL-0008-hook-decoupling.md`** — `loadDaemonStatus()` in `.claude/hooks/startup.ts`. Your new section in `startup.ts` mounts adjacent to it and uses the same `═══ HEADING ═══` framing.
4. **`.claude/hooks/startup.ts`** lines 87-125 — the `loadDaemonStatus()` function. Your `loadMcpReadiness()` function copies its error-swallowing structure exactly.
5. **`.claude/hooks/startup.ts`** lines 197-202 — the wiring point in `main()`. Your new section appends after the ASL DAEMON section.
6. **`.claude/asl-readiness.json`** — the canonical config file for this feature. Shape documented in the ADR above.
7. **`src/libs/judge-panel.ts:261-297`** — `checkJudgeHealth` reference implementation. Note the `_pingJudge` helper, the `Promise.allSettled` orchestration, the `errorType` taxonomy, and the stderr summary line.
8. **`C:\Users\odiem\.claude-team\plugins\installed_plugins.json`** — the truth source for which plugins exist for this project. Read it (gracefully handle missing/malformed) and filter to entries where `projectPath` matches `fromRoot()` (case-insensitive on Windows).
9. **`src/servers/freddie-ai/`** — the LOCAL MCP. Read enough to confirm its plugin name is `freddie-ai` so the probe correctly excludes it. **Do not** treat it as external.

---

## Target state

### Config schema (`.claude/asl-readiness.json`)

```jsonc
{
  "externalMcp": {
    "enabled": false,
    "ignore": []
  }
}
```

| Field | Type | Default | Meaning |
|---|---|---|---|
| `externalMcp.enabled` | boolean | `false` | Master switch. When `false`, the probe is fully skipped — zero work, zero output. |
| `externalMcp.ignore` | string[] | `[]` | Plugin names to skip even when enabled. Use this to suppress noise from installed-but-unused plugins. |

**Why not `.claude/settings.json`:** See ADR above. Claude Code rejects unrecognized top-level keys in that file at runtime, so ASL-tier flags must live in their own files.

### New library `src/libs/mcp-readiness.ts`

Public API (mirrors `checkJudgeHealth` shape — same idiom, same result envelope). See the shipped file for the complete type definitions and implementation — the final error type union is:

```ts
export type McpReadinessErrorType = "no-process" | "config-error" | "api";
```

(Originally drafted as 5 values; `"not-installed"` and `"stale-pid"` were dropped because they are unreachable in the process-existence-only probe strategy. Add them back if a PID-file probe path is implemented.)

**Implementation rules:**
- Pure TypeScript, Bun runtime. No new dependencies.
- All filesystem reads use `try/catch + return default`. The probe must never crash session start.
- Process enumeration is delegated to `src/libs/process.ts :: listProcesses` (platform-agnostic library shared with future call sites). The `mcp-readiness` module calls `listProcesses({ names: ["bun.exe", "node.exe"] })` once per probe cycle, and shares the result across all per-plugin probes.
- The orchestrator's total wall-clock budget is **1500ms**. Process enumeration participates in the race via `Promise.race` — if the full probe (enumeration + per-plugin matching) doesn't resolve within the ceiling, every unresolved plugin is marked `errorType: "api", reason: "probe timeout"`.

### Reusable process library `src/libs/process.ts` (NEW — delivered by this task)

Extracted from `mcp-readiness.ts` during the review cycle so other subsystems can consume platform-agnostic process primitives without duplicating OS branching.

**Public API:**

```ts
export interface ProcessInfo {
  pid: number;
  name: string;        // "bun.exe", "node.exe", etc
  commandLine: string; // full command line
}

export interface ProcessFilter {
  names?: string[];                // OS-level executable name filter (faster)
  commandLineIncludes?: string;    // post-filter substring match
}

export interface KillOptions {
  force?: boolean;  // taskkill /F on Windows, SIGKILL on Unix
  tree?: boolean;   // taskkill /T on Windows, kill process group on Unix
  timeout?: number;
}

export function isAlive(pid: number): boolean;
export function killProcess(pid: number, opts?: KillOptions): boolean;
export async function listProcesses(filter?: ProcessFilter): Promise<ProcessInfo[]>;
export async function findProcess(matcher: ProcessFilter): Promise<ProcessInfo | null>;
export function parsePowerShellCsv(raw: string): ProcessInfo[];
export function parsePsOutput(raw: string): ProcessInfo[];
```

**Rules:**
- Uses `detectPlatform()` from `src/libs/platform.ts` for OS branching — no raw `process.platform` checks (except signal literals inside `killProcess`, which are unavoidable).
- All public functions are non-throwing — errors are logged to stderr and mapped to default return values (`false`, `null`, `[]`).
- Uses `execFile` not `execSync`/`exec` for enumeration — bypasses cmd.exe shell and avoids shell-quoting bugs.
- Hard 3000ms timeout on enumeration spawn (`ENUM_TIMEOUT_MS`). The 1500ms probe ceiling lives in `mcp-readiness.ts` and races against the full enumeration + matching pipeline.
- `parsePowerShellCsv` and `parsePsOutput` are **pure functions** — no filesystem, no spawns, no env. This enables fixture-based unit testing without spawning real processes.
- Windows enumeration uses PowerShell `Get-CimInstance Win32_Process | Select-Object ProcessId,Name,CommandLine | ConvertTo-Csv -NoTypeInformation`. The parser handles quoted CSV with embedded commas and doubled-quote escaping correctly.
- Unix enumeration uses `ps -eo pid,comm,command` so both the executable name and the full command line are available.

This library exists so `mcp-readiness.ts`, the ASL daemon, the Obsidian sync tool, the backup scheduler, and any other subsystem that needs process state can share one audited implementation. See Follow-ups for the ASL-0014 migration task.

### Hook wiring — `.claude/hooks/startup.ts`

**Add a new loader function `loadMcpReadiness(): Promise<string[]>`** that:

1. Calls `checkExternalMcpReadiness()` from `src/libs/mcp-readiness.ts`.
2. If `result.enabled === false` → return `[]` (no section rendered).
3. If `result.ready.length === 0 && result.notReady.length === 0` → return `["External MCP: none installed"]` (single info line — confirms the probe ran).
4. Otherwise return one line per MCP:
   - Ready: `External MCP: {name} READY`
   - Not ready: `External MCP: {name} {ERRORTYPE}` followed by an indented `  → {remediation}` line.
5. Swallow ALL errors and return `[]` on catastrophic failure.

**Wire it into `main()`** after the ASL DAEMON section (lines 197-202):

```ts
const mcpReadiness = await loadMcpReadiness();
if (mcpReadiness.length > 0) {
  lines.push("═══ EXTERNAL MCP ═══");
  lines.push(mcpReadiness.join("\n"));
  lines.push("");
}
```

**`main()` becomes async** because of the await. The `try { main(); }` at the bottom needs to become `main().catch(...)`. The catch block must still write to stderr and exit 0 — never block session start.

**Do NOT** import any heavy modules at the top of `startup.ts`. Use a dynamic `await import("../../src/libs/mcp-readiness.ts")` inside `loadMcpReadiness()` so the cost is only paid when the section is actually rendered. The existing `loadDaemonStatus()` uses pure stdlib for the same reason.

### Remediation hint table

The `remediation` field in each `McpReadinessEntry` carries a one-liner the user can act on. Hardcoded by `errorType`:

| errorType | Reason example | Remediation hint |
|---|---|---|
| `no-process` | Plugin installed, no matching process found | `The MCP server is not running. Restart Claude Code to respawn the MCP server` |
| `config-error` | `.mcp.json` malformed or missing fields | `Check {installPath}/.mcp.json — server entry malformed` |
| `api` | Probe threw or timed out | `Probe failed: {reason} — re-run session or report bug` |

**Note on `/plugin reload`:** verified — does not exist as a first-class command. Ryan ships "Restart Claude Code" as the fallback remediation. No fabricated remediations.

---

## File-by-file changes

### 1. `src/libs/mcp-readiness.ts` (NEW)

Implement the API above. Use `fromRoot()` from `src/libs/paths.ts` for project-root resolution. Use `os.homedir()` for the plugins json path. Delegates all process enumeration to `src/libs/process.ts`.

Key helpers:
- `loadReadinessConfig(configPath?)` — read `.claude/asl-readiness.json` (or a supplied path), JSON.parse with try/catch, default to `{ enabled: false, ignore: [] }`. Accepts an optional `configPath` parameter for testing.
- `discoverExternalMcps(pluginsJsonPath?)` — read `~/.claude-team/plugins/installed_plugins.json` (or supplied path), filter by `projectPath` matching `fromRoot()` case-insensitively, exclude `freddie-ai-local`, then for each plugin read `installPath/.mcp.json` to extract server entries. Accepts an optional `pluginsJsonPath` parameter for testing.
- `probeMcp(entry, processList)` — takes a discovered plugin plus a pre-fetched `ProcessInfo[]` and returns one `McpReadinessEntry` per server, using installPath substring match.

### 2. `src/libs/__tests__/mcp-readiness.test.ts` (NEW)

Unit tests for:
1. `loadReadinessConfig` — synthetic tmp-path configs covering missing, empty, malformed, absent `externalMcp` key, wrong `enabled` type, valid config, ignore array, and ignore array with non-string entries.
2. `discoverExternalMcps` — synthetic tmp plugin manifests covering missing file, malformed file, `freddie-ai-local` exclusion, `scope !== "local"` filter, `projectPath` mismatch filter, and malformed `.mcp.json` producing a `config-error` placeholder.
3. `checkExternalMcpReadiness` — shape assertions, multi-call idempotency, and a latency assertion that the call returns under `PROBE_TIMEOUT_MS + 500ms` buffer.
4. `probeMcp` — exhaustive coverage: config-error on empty servers, no-process on empty processList, ok:true when processList contains a matching commandLine, and verification that every errorType branch produces a remediation string.

**Parser tests (`parsePowerShellCsv`, `parsePsOutput`) live in `src/libs/__tests__/process.test.ts`** since those parsers now live in `src/libs/process.ts`.

**Note on Bun test on Windows:** per the user-profile note (`Bun test broken on Windows`), test runs may segfault. If `bun test` segfaults, Ryan documents in the commit message and runs smoke verification via direct execution.

### 3. `src/libs/process.ts` (NEW — refactor delivery)

Platform-agnostic process enumeration and control library. See the Target state section for the public API. Uses `detectPlatform()` from `src/libs/platform.ts`. No imports outside stdlib + `./platform.js` — stays circular-dependency free. Replaces ~264 lines of private process handling code that originally lived inside `mcp-readiness.ts`.

### 4. `src/libs/__tests__/process.test.ts` (NEW — refactor delivery)

20 tests covering:
- `parsePowerShellCsv`: happy path, embedded commas, doubled-quote escaping, empty result, malformed/truncated output, empty-CommandLine filtering, LF line endings, column-order tolerance.
- `parsePsOutput`: happy path with `pid,comm,command`, empty result, whitespace edge cases, missing separators, non-numeric PID, two-column `pid,command` fallback.
- `isAlive`: current pid returns true, impossible pid (999999) returns false, pid 0 returns false, negative pid returns false.

`listProcesses` is intentionally not end-to-end tested (spawns PowerShell; slow and machine-dependent). Parsers and `isAlive` cover the pure-logic surface.

### 5. `.claude/hooks/startup.ts`

Apply the wiring described in "Hook wiring" above:
1. Add `loadMcpReadiness(): Promise<string[]>` function near `loadDaemonStatus()`.
2. Make `main()` async. Convert `try { main(); }` at the bottom to `main().catch(err => process.stderr.write(...))`.
3. Append the `═══ EXTERNAL MCP ═══` section after the `═══ ASL DAEMON ═══` block.
4. Use dynamic import for `mcp-readiness.ts` to keep the cold-start cost zero when the flag is off.

### 6. `.claude/asl-readiness.json` (NEW)

Create with `{ "externalMcp": { "enabled": false, "ignore": [] } }`. Ship the file committed so users have a template to flip when they want to enable the probe.

### 7. `.claude/rules/architecture.md`

Append a short subsection:

> **External MCP readiness probe (ASL-0011):** When `externalMcp.enabled` is `true` in `.claude/asl-readiness.json`, `startup.ts` reads installed plugin manifests from `~/.claude-team/plugins/installed_plugins.json`, scans for matching MCP server processes (via `src/libs/process.ts :: listProcesses`), and prints a per-MCP status block in the session-start briefing. Probe is process-existence only — no RPC ping. Plugins listed in `externalMcp.ignore` are skipped. The local `freddie-ai` MCP is always excluded. Probe is non-blocking and never fails session start. Default: disabled. Config file is `.claude/asl-readiness.json` (not `.claude/settings.json` — Claude Code rejects unknown top-level keys in settings.json, so ASL flags live in dedicated files).

---

## Acceptance criteria

1. **Flag off (default):** With `externalMcp.enabled` absent or `false`, session start produces NO `═══ EXTERNAL MCP ═══` section. Confirm via fresh session boot. No measurable latency change in `startup.ts` cold time.
2. **Flag on, plugins present:** With `enabled: true` and the discord + claude-mem plugins installed (current state), session start shows the section listing both. Each MCP shows either READY (with no remediation) or a non-READY status with a one-liner remediation.
3. **Flag on, no external plugins:** With `enabled: true` but no project-scoped plugins discovered, the section shows `External MCP: none installed`. Verifies the probe ran and discovered nothing rather than crashing.
4. **Local freddie-ai NEVER appears in the section.** Confirmed by grepping the rendered output for `freddie-ai`. Zero matches.
5. **Ignore list works:** With `enabled: true, ignore: ["discord"]`, the discord entry does not appear in the rendered section. claude-mem still appears.
6. **Plugins json missing → graceful default:** Temporarily rename `~/.claude-team/plugins/installed_plugins.json` to `.bak`. Boot a session. The probe runs with `enabled: true` and renders `External MCP: none installed`. No errors in stderr that mention the missing file at the user level (a single debug stderr line is acceptable). Restore the file after the test.
7. **Malformed `.mcp.json` for one plugin → that plugin is `config-error`, others continue.** Simulate by temporarily appending invalid JSON to one plugin's `.mcp.json`. Other plugins still probe correctly. Restore after test.
8. **Probe latency budget (REVISED 2026-04-08):** **<500ms typical on Unix, <2s typical on Windows.** 1500ms hard ceiling on the probe call itself enforced via `Promise.race`. Windows latency is dominated by PowerShell cold-start (~880ms) + `Get-CimInstance` query (~650ms), totalling ~1530ms — this is a platform constant. Faster alternatives were investigated and rejected: `claude mcp list` is authoritative but takes ~9.6s consistent (6x worse), and `tasklist /FO CSV` lacks the `CommandLine` column needed for installPath matching. User accepted the tradeoff on 2026-04-08: "let's go with 1, let just accept that latency for windows. not a deal breaker for me." The original `<500ms typical` wording is retired for Windows.
9. **Probe never blocks session start:** Inject a synthetic crash by setting `installed_plugins.json` to invalid JSON. Session boots normally. The section either renders nothing or an empty entry — never crashes.
10. **Stderr summary line present:** When `enabled: true` and probe runs, a single `[mcp-readiness] check: ready=[...] not-ready=[...] ignored=[...]` line appears in stderr (not stdout — stdout is the briefing only). Mirrors the `[judge-panel] health check:` pattern.
11. **No PreToolUse interception added.** Grep `.claude/settings.json` for any new hook entries — zero net hook additions.
12. **Tests file authored.** `src/libs/__tests__/mcp-readiness.test.ts` exists with all cases listed in §2 of the changes. Parser tests live in `src/libs/__tests__/process.test.ts`. If `bun test` segfaults on Windows, document in commit message.
13. **Remediation hints exist for every errorType.** Manual code review: every branch in the probe library that sets `errorType` also sets a `remediation` string from the table.
14. **Idempotent re-runs:** Boot two sessions in quick succession. Both render the same EXTERNAL MCP section. The probe has no global state.
15. **Process library lives in `src/libs/process.ts`.** `mcp-readiness.ts` no longer contains enumeration or parser code — it imports `listProcesses` and `ProcessInfo` from `./process.js`. `process.ts` uses `detectPlatform()` from `./platform.js` and imports nothing else outside stdlib.

---

## Risks and mitigations

| Risk | Severity | Mitigation |
|---|---|---|
| **Process enumeration is slow on Windows** (PowerShell cold start + CimInstance ~1500ms). | **Accepted** | User decision 2026-04-08: accept the Windows latency. Faster alternatives are worse — `claude mcp list` is 9.6s, `tasklist` lacks CommandLine. Hard 1500ms ceiling on the probe via `Promise.race`. |
| **False negatives — MCP slow to start.** Plugin process not yet spawned at session-start time. This is the *fundamental* race: at SessionStart, the MCPs we probe are being spawned by the very session that runs the probe — we may look up processes that haven't registered yet. | Medium | Acceptable risk for v1 — the probe is inferential by design. Documented in Follow-ups as the driver for a future protocol-level probe. Users who care about precision can re-open a session. |
| **False positives — generic `bun.exe` process matches the wrong thing.** Process detection uses installPath substring matching. | Medium | Match against the FULL installPath substring (includes `\plugins\cache\{plugin}\{version}\`). Document the heuristic in the source. |
| **`installed_plugins.json` schema changes** in a future Claude Code release. | Low | The probe is defensive — any read or parse error returns empty discovery with a stderr warning. No crash. |
| **Settings flag schema collides with Claude Code validator.** | **Resolved** | ADR: config moved to dedicated `.claude/asl-readiness.json`. Settings.json is never touched for ASL flags. |
| **User enables the flag, ignores the warnings, then complains tools are broken.** | Low | The whole point of the gate is to surface state, not enforce it. Document explicitly in the hint copy that the probe is informational. |
| **Probe added latency to every session, even with flag off.** | Low | Flag check is the FIRST thing the loader does. When `enabled: false`, the loader returns the disabled envelope immediately. Dynamic import means the heavy module is never even loaded when the flag is off. Verify in acceptance criterion #1. |
| **Plugins json path differs on macOS/Linux.** | Low | Use `os.homedir()` + `.claude-team/plugins/installed_plugins.json`. Claude Code uses the same path on all platforms. |
| **`/plugin reload` may not exist in Claude Code.** | **Resolved** | Verified: does not exist. Shipped remediation copy is "Restart Claude Code to respawn the MCP server." |

---

## Test plan (Ryan's manual smoke test)

Run these in order. Report pass/fail and any unexpected behavior in the commit message.

1. **Baseline cold-start latency.** Note current session-start time before any edits. Use `time bun .claude/hooks/startup.ts` (or wall-clock observation) as a baseline.
2. **Apply edits:** create `src/libs/process.ts`, `src/libs/__tests__/process.test.ts`, `src/libs/mcp-readiness.ts`, `src/libs/__tests__/mcp-readiness.test.ts`; edit `.claude/hooks/startup.ts`; create `.claude/asl-readiness.json`; edit `.claude/rules/architecture.md`.
3. **Flag-off latency check.** With `externalMcp.enabled: false` (default), re-run `time bun .claude/hooks/startup.ts`. Latency should be within +/- 20ms of baseline.
4. **Discovery test.** Add a temporary one-liner `console.error(JSON.stringify(discoverExternalMcps(), null, 2))` to `mcp-readiness.ts`, run it directly via `bun src/libs/mcp-readiness.ts`, confirm discord + claude-mem appear and freddie-ai-local does NOT. Remove the debug line.
5. **Flag-on, both plugins present.** Set `enabled: true`. Boot a fresh session. Confirm the EXTERNAL MCP section renders with both plugins. Document READY/not-ready status observed.
6. **Ignore list test.** Set `ignore: ["discord"]`. Boot a session. Confirm discord is absent from the section, claude-mem still present.
7. **Latency budget test.** With flag on, time the hook 3 times. Record the numbers. Target: <2s typical on Windows, <500ms on Unix.
8. **Missing plugins json test.** Rename `~/.claude-team/plugins/installed_plugins.json` to `.bak`. Boot a session with flag on. Confirm `External MCP: none installed`. Restore.
9. **Malformed plugin .mcp.json test.** Append `{{{` to one plugin's `.mcp.json`. Boot a session. Confirm that plugin shows `config-error` and the OTHER plugin is unaffected. Restore.
10. **Crash protection test.** Corrupt `installed_plugins.json` to invalid JSON. Boot a session. Confirm the session boots normally. Restore.
11. **Local MCP exclusion test.** Grep the rendered briefing output for `freddie-ai`. Zero matches.
12. **Remediation hints test.** Cause a `no-process` state and confirm the rendered line has both the status and the indented remediation hint.
13. **Idempotency test.** Boot two sessions back-to-back. Both render the same content.
14. **Test file smoke run.** Run `bun test src/libs/__tests__/` for both mcp-readiness.test.ts and process.test.ts. If segfaults occur on Windows, document in commit.

---

## Dependencies

- **ASL-0001** — `checkJudgeHealth` is the structural template. The new `checkExternalMcpReadiness` function follows its idiom (allSettled + classify + envelope) without importing it.
- **ASL-0008** — `loadDaemonStatus()` in `startup.ts` is the wiring template. The new `loadMcpReadiness()` function mounts adjacent to it and follows the same error-swallowing pattern.

No downstream blockers. ASL-0011 can ship independently of ASL-0009 and ASL-0012.

---

## What Ryan must NOT do

- **Do not** add any PreToolUse hook to `.claude/settings.json`. This task is startup-only.
- **Do not** import `judge-panel.ts` or `mcp-readiness.ts` from each other. They share an idiom, not code.
- **Do not** touch `src/servers/freddie-ai/`. The local MCP is out of scope.
- **Do not** fabricate remediation commands. Verified: `/plugin reload` does not exist — ship "Restart Claude Code" instead.
- **Do not** spawn or restart any MCP. The probe is read-only.
- **Do not** modify any tool to self-check on first call. The probe lives in one place.
- **Do not** add new dependencies. Pure Bun + Node stdlib + existing project libs.
- **Do not** make the flag default to `true`. First release ships opt-in only.
- **Do not** synchronously block session start. All probe work happens behind the dynamic import + 1500ms ceiling.
- **Do not** edit BRIEF.md. It's a historical design doc; readiness checklist update belongs in the next briefing pass.
- **Do not** add ASL-tier config keys to `.claude/settings.json` — use dedicated `.claude/asl-*.json` files.

---

## Reporting back

When you finish, report:

1. Latency numbers from test plan steps 1, 3, 7 (baseline / flag-off / flag-on).
2. Output of test step 4 (discovered MCPs) — confirm freddie-ai is absent.
3. Pass/fail for all 15 acceptance criteria with one-line notes on any that required interpretation.
4. Confirmation that `/plugin reload` does not exist and which remediation copy shipped.
5. Whether `bun test` ran cleanly or segfaulted on Windows for both new test files.
6. Any false positives/negatives observed during process enumeration.
7. The exact rendered EXTERNAL MCP section from a real session boot (paste from session context).

---

## Follow-ups

### ASL-0014 — Migrate call sites to `src/libs/process.ts`

**Scope:** Replace ad-hoc process enumeration, liveness, and kill patterns across the codebase with calls into the new `src/libs/process.ts` library. Five existing sites duplicate the logic and should consume the shared implementation:

| File | Pattern to migrate |
|---|---|
| `src/services/self-learning-daemon/daemon.ts` (x2) | Process liveness checks — currently uses `process.kill(pid, 0)` inline; should use `isAlive(pid)`. |
| `src/tools/obsidian/sync.ts` | Obsidian process detection — currently uses custom enumeration; should use `findProcess({ names: [...] })`. |
| `src/tools/asl-sync.ts` | Daemon liveness pre-flight; should use `isAlive(pid)` or `findProcess`. |
| `.claude/hooks/startup.ts :: loadDaemonStatus` | Standalone `process.kill(pid, 0)` for daemon PID check; should use `isAlive(pid)`. |
| `src/servers/freddie-ai/daemon.ts` | Server daemon lifecycle; should use `isAlive` + `killProcess`. |

Migrating these reduces the audit surface for OS-specific bugs and ensures any future fixes to process enumeration (e.g., new PowerShell edge cases, `ps` format changes) apply consistently.

**Task ID:** ASL-0014 (next free number — confirmed against `vault/studio/projects/autonomous-self-learning/tasks/TASKS.md`).

**Estimated complexity:** small — pure refactor, no behavior change, guided by existing tests in `process.test.ts`.

### Fundamental SessionStart race — documented acceptable risk

At SessionStart, the MCPs being probed are spawned by the very session that runs the probe. Process enumeration may report false-negatives in the early window because the child process has not yet registered with the OS process table when `startup.ts` runs.

**v1 decision:** Accept this as an inferential limitation. The probe is a "best-effort heads-up," not a deterministic guarantee. Users who see a NO-PROCESS warning on a freshly-booted session can re-boot or ignore it if the MCP is actually working.

**Future fix (not scheduled):** Replace the process-existence probe with a protocol-level probe — send a JSON-RPC ping through the Claude Code MCP bridge (if a first-class API becomes available) and classify by response latency. This requires Claude Code to expose a programmatic MCP server enumeration or ping endpoint, which does not exist at the time of this task. When/if it appears, a new task should supersede this one and fold the process-existence path into a fallback.

This observation is logged here so any future reviewer of the readiness feature understands the v1 contract: informational only, inferential, best-effort, accepting early-window false negatives as the cost of shipping without a protocol-level API.
