---
name: Windows Tool Development Notes
description: Specific technical considerations and best practices for developing tools on Windows, particularly concerning process management and PowerShell integration.
type: knowledge
agent: freddie
tags: [windows, powershell, nodejs, tool-development, process-management, debugging, obsidian]
---

## Process Management on Windows
- **Node.js `spawn` Behavior:** When using Node.js `spawn({ detached: true })` on Windows for console applications, a console window will always pop up. The `windowsHide: true` option is ineffective due to an unfixed Node.js bug.
- **Reliable Window Hiding:** The only reliable method to manage window state for spawned processes on Windows is to use PowerShell's `Start-Process -WindowStyle Hidden`. This manages the window state at the operating system level.
- **`pm2` on Windows:** `pm2` also exhibits the terminal window popping issue on Windows (showing `node.exe` titles), indicating the same underlying `spawn` problem. Direct PowerShell approaches are preferred to avoid this.

## PowerShell Integration Best Practices
- **Argument Handling:** When passing arguments to PowerShell scripts using `-ArgumentList`, provide arguments as comma-separated quoted items (e.g., `'-sync','--path','vault'`) rather than a single concatenated string (`'-sync --path vault'`).
- **Logging:** PowerShell's `RedirectStandardOutput` and `RedirectStandardError` cannot point to the same file. Separate log files must be used for standard output and standard error streams.

## Obsidian CLI Specifics
- **Continuous Sync:** The `ob sync --continuous` command initiates a long-running watcher process, not an interval-based synchronization. Tools should manage its PID file for daemonization.
- **Portable Vault Path Resolution:** Vault paths can be resolved dynamically by matching an `OBSIDIAN_VAULT_ID` environment variable against the output of `ob sync-list-local`, enabling portability across different machine setups.

**Why:** These specific platform and tool interactions can lead to unexpected behavior (e.g., unwanted console windows, incorrect command execution) and impact tool reliability and user experience. Understanding these nuances is crucial for robust tool development on Windows.

**How to apply:**
- When spawning background processes on Windows, especially console applications, favor PowerShell's `Start-Process -WindowStyle Hidden` over Node.js `spawn` with `detached: true` for silent execution.
- Avoid `pm2` for daemonizing Node.js applications on Windows if silent execution is critical.
- Ensure correct argument formatting for PowerShell `-ArgumentList`.
- Implement separate logging for stdout and stderr when redirecting PowerShell output.
- Design Obsidian CLI integrations to account for `ob sync --continuous` being a watcher and use dynamic vault path resolution for portability.

*Consolidated from: 20260328-110500-obsidian-sync-tool.md*
