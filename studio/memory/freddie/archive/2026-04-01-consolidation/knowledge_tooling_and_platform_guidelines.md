---
name: Tooling and Platform Guidelines
description: Best practices for tool execution, prioritizing CLI-first calls, and specific technical considerations for developing and managing tools on Windows, including process management and PowerShell integration.
type: knowledge
agent: freddie
tags: [tool_execution, user_preference, debuggability, architecture, workflow, windows, powershell, nodejs, tool-development, process-management, obsidian]
---

## Tool Execution Preference
- **CLI-first Principle:** For all internal tool executions, whether by the main agent or any subagents, prioritize CLI-first calls over MCP stdio calls. This aligns with user preference.

## Windows Specifics for Tool Development
### Process Management on Windows
- **Node.js `spawn` Behavior:** When using Node.js `spawn({ detached: true })` on Windows for console applications, a console window will always pop up due to an unfixed Node.js bug. The `windowsHide: true` option is ineffective.
- **Reliable Window Hiding:** The only reliable method to manage window state for spawned processes on Windows is to use PowerShell's `Start-Process -WindowStyle Hidden`. This manages the window state at the operating system level.
- **`pm2` on Windows:** `pm2` also exhibits the terminal window popping issue on Windows (showing `node.exe` titles), indicating the same underlying `spawn` problem. Direct PowerShell approaches are preferred to avoid this.

### PowerShell Integration Best Practices
- **Argument Handling:** When passing arguments to PowerShell scripts using `-ArgumentList`, provide arguments as comma-separated quoted items (e.g., `'-sync','--path','vault'`) rather than a single concatenated string (`'-sync --path vault'`).
- **Logging:** PowerShell's `RedirectStandardOutput` and `RedirectStandardError` cannot point to the same file. Separate log files must be used for standard output and standard error streams.

### Obsidian CLI Specifics
- **Continuous Sync:** The `ob sync --continuous` command initiates a long-running watcher process, not an interval-based synchronization. Tools should manage its PID file for daemonization.
- **Portable Vault Path Resolution:** Vault paths can be resolved dynamically by matching an `OBSIDIAN_VAULT_ID` environment variable against the output of `ob sync-list-local`, enabling portability across different machine setups.

**Why:** Prioritizing CLI-first calls significantly enhances visibility and debuggability of internal processes, aligning directly with user preference. Understanding Windows-specific platform and tool interactions (e.g., unwanted console windows, incorrect command execution) is crucial for robust tool development and impacts tool reliability and user experience.

**How to apply:**
- When designing or implementing tool calls within any agent or subagent workflow, ensure the primary method of execution is via CLI commands where a CLI alternative exists.
- When spawning background processes on Windows, especially console applications, favor PowerShell's `Start-Process -WindowStyle Hidden` over Node.js `spawn` with `detached: true` for silent execution.
- Avoid `pm2` for daemonizing Node.js applications on Windows if silent execution is critical.
- Ensure correct argument formatting for PowerShell `-ArgumentList`.
- Implement separate logging for stdout and stderr when redirecting PowerShell output.
- Design Obsidian CLI integrations to account for `ob sync --continuous` being a watcher and use dynamic vault path resolution for portability.

*Consolidated from: knowledge_tool_execution_guidelines.md, knowledge_windows_development.md*
