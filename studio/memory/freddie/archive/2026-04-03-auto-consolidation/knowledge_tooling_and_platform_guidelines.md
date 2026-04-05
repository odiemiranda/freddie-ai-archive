---
name: Tooling and Platform Guidelines
description: Best practices for tool execution, development, prioritizing CLI-first calls, and specific technical considerations for developing and managing tools on Windows, including process management and PowerShell integration.
type: knowledge
agent: freddie
tags: [tool_execution, user_preference, debuggability, architecture, workflow, windows, powershell, nodejs, tool-development, process-management, obsidian, cli_first, readability, maintainability, code_style, parser, serializer, testing, data_integrity, library_design, infrastructure, security, robustness, defensive_coding, bun]
---

## Tool Execution Preference
- **CLI-first Principle:** For all internal tool executions, whether by the main agent or any subagents, prioritize CLI-first calls over MCP stdio calls. This aligns with user preference.

## Tool Development Best Practices
- **Avoid Inline Code Blocks:** Do not use inline code blocks (e.g., `bun -e`) directly within workflows.
- **Encapsulate Logic:** Instead, encapsulate such logic into dedicated CLI tools.
- **Dedicated CLI Router:** Developing a dedicated CLI router with advanced features like auto-discovery, aliases, subprocess isolation, and defensive coding (e.g., path traversal guard) significantly enhances the robustness, maintainability, and security of internal tool execution. This approach effectively extends the CLI-first principle and integrates defensive coding reflexes into core infrastructure.
- **Enforce Rule:** A `no-inline-code` rule should be enforced across all skill workflows.
- **Benefit:** This approach significantly improves readability, maintainability, and aligns with user preference for explicit, well-defined tool calls, while enhancing security and robustness.

## Library/Data Parsing Development Best Practices
- **Modular Architecture:** For developing robust data parsing and manipulation libraries, a modular architecture separating `types`, `parser`, `serializer`, and `operations` components is highly effective.
- **Comprehensive Testing:** Couple modular architecture with comprehensive roundtrip testing to ensure data integrity and reliable functionality across various data inputs.

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

**Why:** Prioritizing CLI-first calls and encapsulating logic in dedicated tools, including robust CLI routers, significantly enhances visibility, debuggability, readability, maintainability, security, and robustness of internal processes, aligning directly with user preference. Understanding Windows-specific platform and tool interactions (e.g., unwanted console windows, incorrect command execution) is crucial for robust tool development and impacts tool reliability and user experience. Adopting a modular architecture with comprehensive testing for data parsing libraries ensures data integrity and reliable functionality, which is critical for system stability.

**How to apply:**
- When designing or implementing tool calls within any agent or subagent workflow, ensure the primary method of execution is via CLI commands where a CLI alternative exists.
- Avoid using inline code blocks (`bun -e`) in workflows; instead, create and use dedicated CLI tools for such logic, adhering to the `no-inline-code` rule.
- Consider developing dedicated CLI routers for internal tools to enhance auto-discovery, aliases, subprocess isolation, and integrate defensive coding practices like path traversal guards.
- When developing data parsing or manipulation libraries, implement a modular architecture (types, parser, serializer, operations) and include comprehensive roundtrip testing.
- When spawning background processes on Windows, especially console applications, favor PowerShell's `Start-Process -WindowStyle Hidden` over Node.js `spawn` with `detached: true` for silent execution.
- Avoid `pm2` for daemonizing Node.js applications on Windows if silent execution is critical.
- Ensure correct argument formatting for PowerShell `-ArgumentList`.
- Implement separate logging for stdout and stderr when redirecting PowerShell output.
- Design Obsidian CLI integrations to account for `ob sync --continuous` being a watcher and use dynamic vault path resolution for portability.

*Consolidated from: knowledge_tool_execution_guidelines.md, knowledge_windows_development.md, 20260401-020156-avoid-using-inline-code-blocks-e-g-bun-e-3.md, 20260401-175949-for-developing-robust-data_parsing_and_m-2.md, 20260403-070905-developing-a-dedicated-cli-router-with-a-1.md*
