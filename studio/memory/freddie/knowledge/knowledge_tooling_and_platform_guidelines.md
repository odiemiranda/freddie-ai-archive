---
name: Tooling and Platform Guidelines
description: Best practices for tool execution, development, prioritizing CLI-first calls, and specific technical considerations for developing and managing tools on Windows, including process management and PowerShell integration, ensuring robustness and security, and noting known tool limitations.
type: knowledge
agent: freddie
tags: [tool_execution, user_preference, debuggability, architecture, workflow, windows, powershell, nodejs, tool-development, process-management, obsidian, cli_first, readability, maintainability, code_style, parser, serializer, testing, data_integrity, library_design, infrastructure, security, robustness, defensive_coding, bun, standardization, execution, tool_limitation, bug, false_positive, tool_behavior, minimax-video, bug_fix, local_files, video_generation]
---

## Tool Execution Preference
- **CLI-first Principle:** For all internal tool executions, whether by the main agent or any subagents, prioritize CLI-first calls over MCP stdio calls. This aligns with user preference.

## Standardized Tool Execution
- **System-wide `bun run`:** `bun run` has been established as the standardized, system-wide execution mechanism for scripts across all agent components, expanding its utility beyond just package management.

## Tool Development Best Practices
- **Avoid Inline Code Blocks:** Do not use inline code blocks (e.g., `bun -e`) directly within workflows.
- **Encapsulate Logic:** Instead, encapsulate such logic into dedicated CLI tools.
- **Dedicated CLI Router:** Developing a dedicated CLI router with advanced features like auto-discovery, aliases, subprocess isolation, and defensive coding (e.g., path traversal guard) significantly enhances the robustness, maintainability, and security of internal tool execution. This approach effectively extends the CLI-first principle and integrates defensive coding reflexes into core infrastructure.
- **Defensive Coding Mechanisms:** Implement explicit defensive coding mechanisms such as path traversal guards and PreToolUse bash-guards (blocking dangerous commands like `cd&&`, absolute paths, and direct tool paths) to ensure the robustness and security of core infrastructure components and tool execution.
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

## Known Tool Limitations
- **`section-balance` Tool:** The `section-balance` tool has a specific known limitation where it misinterprets typed brackets (e.g., `[Chorus]`) as structural elements, leading to incorrect balance assessments. This behavior needs to be considered when interpreting its output or when designing lyrics inputs.

## Tool Specific Improvements
- **`minimax-video` Local File Path Support:** The `minimax-video` tool now reliably supports local file paths for its `--first-frame` argument, enhancing its flexibility and integration within local video generation workflows.

**Why:** Prioritizing CLI-first calls and encapsulating logic in dedicated tools, including robust CLI routers and specific defensive coding mechanisms, significantly enhances visibility, debuggability, readability, maintainability, security, and robustness of internal processes, aligning directly with user preference. Standardizing `bun run` as the system-wide execution mechanism improves consistency and efficiency. Understanding Windows-specific platform and tool interactions (e.g., unwanted console windows, incorrect command execution) is crucial for robust tool development and impacts tool reliability and user experience. Adopting a modular architecture with comprehensive testing for data parsing libraries ensures data integrity and reliable functionality, which is critical for system stability. Documenting known tool limitations helps in interpreting tool outputs correctly and informs future development or workarounds, while specific tool improvements like `minimax-video`'s local file path support enhance flexibility and integration.

**How to apply:**
- When designing or implementing tool calls within any agent or subagent workflow, ensure the primary method of execution is via CLI commands where a CLI alternative exists.
- Utilize `bun run` as the standardized execution mechanism for scripts across all agent components.
- Avoid using inline code blocks (`bun -e`) in workflows; instead, create and use dedicated CLI tools for such logic, adhering to the `no-inline-code` rule.
- Consider developing dedicated CLI routers for internal tools to enhance auto-discovery, aliases, subprocess isolation, and integrate defensive coding practices like path traversal guards and PreToolUse bash-guards.
- When developing data parsing or manipulation libraries, implement a modular architecture (types, parser, serializer, operations) and include comprehensive roundtrip testing.
- When spawning background processes on Windows, especially console applications, favor PowerShell's `Start-Process -WindowStyle Hidden` over Node.js `spawn` with `detached: true` for silent execution.
- Avoid `pm2` for daemonizing Node.js applications on Windows if silent execution is critical.
- Ensure correct argument formatting for PowerShell `-ArgumentList`.
- Implement separate logging for stdout and stderr when redirecting PowerShell output.
- Design Obsidian CLI integrations to account for `ob sync --continuous` being a watcher and use dynamic vault path resolution for portability.
- Be aware of the `section-balance` tool's limitation regarding typed brackets and account for this when interpreting its output or designing lyrics inputs. Leverage the `minimax-video` tool's enhanced support for local file paths in `--first-frame` arguments for more flexible video generation workflows.

*Consolidated from: knowledge_tool_execution_guidelines.md, knowledge_windows_development.md, 20260401-020156-avoid-using-inline-code-blocks-e-g-bun-e-3.md, 20260401-175949-for-developing-robust-data_parsing_and_m-2.md, 20260403-070905-developing-a-dedicated-cli-router-with-a-1.md, 20260403-082246-bun-run-has-been-established-as-the-stan-2.md, 20260403-082246-the-combination-of-rigorous-mccall-revie-1.md, 20260405-124725-the-section-balance-tool-has-a-specific--2.md, 20260406-081746-the-minimax-video-tool-now-reliably-supp-1.md*
