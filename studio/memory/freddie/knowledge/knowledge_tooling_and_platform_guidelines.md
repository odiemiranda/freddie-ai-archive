---
name: Tooling and Platform Guidelines
description: Best practices for tool execution, development, prioritizing CLI-first calls, and specific technical considerations for developing and managing tools on Windows, including process management and PowerShell integration. This also covers platform-specific behaviors, non-deterministic platform outputs, and known tool limitations, ensuring robustness and security.
type: knowledge
agent: freddie
tags: [tool_execution, user_preference, debuggability, architecture, workflow, windows, powershell, nodejs, tool-development, process-management, obsidian, cli_first, readability, maintainability, code_style, parser, serializer, testing, data_integrity, library_design, infrastructure, security, robustness, defensive_coding, bun, standardization, execution, tool_limitation, bug, false_positive, tool_behavior, minimax-video, bug_fix, local_files, video_generation, suno, platform_behavior, prompting, audio_artifacts, content_filters, isTTY, workaround, environment_variable, chokidar, file_watching, polling, delay, failure_mode, child_process, debugging, observability, agent_improvement, workflow_automation, human_in_the_loop, review_process, non_determinism, drift_study]
---

## Tool Execution Preference
- **CLI-first Principle:** For all internal tool executions, whether by the main agent or any subagents, prioritize CLI-first calls over MCP stdio calls. This aligns with user preference.

## Standardized Tool Execution
- **System-wide `bun run`:** `bun run` has been established as the standardized, system-wide execution mechanism for scripts across all agent components, expanding its utility beyond just package management.

## Tool Development Best Practices
- **Avoid Inline Code Blocks:** Do not use inline code blocks (e.g., `bun -e`) directly within workflows.
- **Encapsulate Logic:** Instead, encapsulate such logic into dedicated CLI tools.
- **Dedicated CLI Router:** Developing a dedicated CLI router with advanced features like auto-discovery, aliases, subprocess isolation, and defensive coding (e.g., path traversal guard) significantly enhances the robustness, maintainability, and security of internal tool execution. This approach effectively extends the CLI-first principle and integrates defensive coding reflexes into core infrastructure.
- **Dedicated CLI Tools for Workflow & Observability:** Dedicated CLI tools significantly enhance system operations by providing crucial visibility for debugging, monitoring, and understanding internal agent state and lifecycle. They also streamline human-in-the-loop processes, improving efficiency, reducing friction, and formalizing interaction points for review and approval stages.
- **Defensive Coding Mechanisms:** Implement explicit defensive coding mechanisms such as path traversal guards and PreToolUse bash-guards (blocking dangerous commands like `cd&&`, absolute paths, and direct tool paths) to ensure the robustness and security of core infrastructure components and tool execution.
- **Enforce Rule:** A `no-inline-code` rule should be enforced across all skill workflows.
- **Benefit:** This approach significantly improves readability, maintainability, and aligns with user preference for explicit, well-defined tool calls, while enhancing security and robustness.

## Library/Data Parsing Development Best Practices
- **Modular Architecture:** For developing robust data parsing and manipulation libraries, a modular architecture separating `types`, `parser`, `serializer`, and `operations` components is highly effective.
- **Comprehensive Testing:** Couple modular architecture with comprehensive roundtrip testing to ensure data integrity and reliable functionality across various data inputs.

## Windows Specifics for Tool Development
### Process Management on Windows
- **Node.js `spawn` Behavior:** When using Node.js `spawn({ detached: true })` on Windows for console applications, a console window will always pop up due to an unfixed Node.js bug. The `windowsHide: true` option is ineffective.
- **Bun.spawn vs. Node.js child_process.spawn:** It is necessary to differentiate between `Bun.spawn` and Node.js `child_process.spawn` based on specific use cases, as they are not always interchangeable and have distinct optimal applications for process management.
- **Reliable Window Hiding:** The only reliable method to manage window state for spawned processes on Windows is to use PowerShell's `Start-Process -WindowStyle Hidden`. This manages the window state at the operating system level.
- **`pm2` on Windows:** `pm2` also exhibits the terminal window popping issue on Windows (showing `node.exe` titles), indicating the same underlying `spawn` problem. Furthermore, `pm2` on Windows may be susceptible to user-driven invalidation or prove less robust for daemonization. Direct PowerShell approaches are preferred to avoid these issues and ensure more stable process management for critical services.

### PowerShell Integration Best Practices
- **Argument Handling:** When passing arguments to PowerShell scripts using `-ArgumentList`, provide arguments as comma-separated quoted items (e.g., `'-sync','--path','vault'`) rather than a single concatenated string (`'-sync --path vault'`).
- **Logging:** PowerShell's `RedirectStandardOutput` and `RedirectStandardError` cannot point to the same file. Separate log files must be used for standard output and standard error streams.

### Obsidian CLI Specifics
- **Continuous Sync:** The `ob sync --continuous` command initiates a long-running watcher process, not an interval-based synchronization. Tools should manage its PID file for daemonization.
- **Portable Vault Path Resolution:** Vault paths can be resolved dynamically by matching an `OBSIDIAN_VAULT_ID` environment variable against the output of `ob sync-list-local`, enabling portability across different machine setups.

### File Watching on Windows
- **Chokidar Polling Delay:** When implementing file watching with `chokidar` on Windows using polling, it is critical to include a 600ms settling delay after the 'ready' event to ensure stability, prevent race conditions, and avoid missed events.

## Bun-Specific Behaviors
- **`Bun.isTTY` Unreliability:** `Bun.isTTY` was consistently found to be undefined and is unreliable. It should be replaced by an environment variable for accurate TTY detection in Bun-based applications to ensure correct conditional logic.

## Known Tool Limitations and Platform Behaviors
- **`section-balance` Tool:** The `section-balance` tool has a specific known limitation where it misinterprets typed brackets (e.g., `[Chorus]`) as structural elements, leading to incorrect balance assessments. This behavior needs to be considered when interpreting its output or when designing lyrics inputs.
- **Suno Platform Behavior:** Specific musical terms ('skank') can trigger platform content flags or unexpected interpretations.
- **Prompt-Induced Audio Artifacts:** Certain descriptive terms or effect combinations ('Building', 'echo/delay/ambient') in prompts can lead to undesirable sonic artifacts ('twang', 'wash') during music generation.
- **Suno Non-Deterministic Analytical Outputs:** Suno's analytical outputs (such as BPM and display tags) are non-deterministic, even for identical audio inputs. This implies that single-reading drift studies or direct comparisons based on these outputs will be inherently noisy and unreliable. Future analyses relying on these metrics should account for this variability, potentially by averaging multiple readings.

## Tool Specific Improvements
- **`minimax-video` Local File Path Support:** The `minimax-video` tool now reliably supports local file paths for its `--first-frame` argument, enhancing its flexibility and integration within local video generation workflows.

**Why:** Prioritizing CLI-first calls and encapsulating logic in dedicated tools, including robust CLI routers and specific defensive coding mechanisms, significantly enhances visibility, debuggability, readability, maintainability, security, and robustness of internal processes, aligning directly with user preference. Dedicated CLI tools for workflow and observability provide crucial insights and streamline human interaction points. Standardizing `bun run` as the system-wide execution mechanism improves consistency and efficiency. Understanding Windows-specific platform and tool interactions (e.g., unwanted console windows, `Bun.spawn` vs. `child_process.spawn` differentiation, `pm2`'s robustness issues, `chokidar` polling delays) is crucial for robust tool development and impacts tool reliability and user experience. Adopting a modular architecture with comprehensive testing for data parsing libraries ensures data integrity and reliable functionality, which is critical for system stability. Documenting known tool limitations and platform-specific behaviors helps in interpreting tool outputs correctly, informs future development or workarounds, and prevents undesirable outcomes. **The non-deterministic nature of Suno's analytical outputs is a critical platform behavior to understand for reliable evaluation and data integrity in drift studies.** Specific tool improvements like `minimax-video`'s local file path support enhance flexibility and integration.

**How to apply:**
- When designing or implementing tool calls within any agent or subagent workflow, ensure the primary method of execution is via CLI commands where a CLI alternative exists.
- Utilize `bun run` as the standardized execution mechanism for scripts across all agent components.
- Avoid using inline code blocks (`bun -e`) in workflows; instead, create and use dedicated CLI tools for such logic, adhering to the `no-inline-code` rule.
- Consider developing dedicated CLI routers for internal tools to enhance auto-discovery, aliases, subprocess isolation, and integrate defensive coding practices like path traversal guards and PreToolUse bash-guards.
- Develop dedicated CLI tools for inspecting agent state and lifecycle, and for streamlining human-in-the-loop processes like review and approval stages.
- When developing data parsing or manipulation libraries, implement a modular architecture (types, parser, serializer, operations) and include comprehensive roundtrip testing.
- When spawning background processes on Windows, especially console applications, favor PowerShell's `Start-Process -WindowStyle Hidden` over Node.js `spawn` with `detached: true` for silent execution.
- Differentiate between `Bun.spawn` and `child_process.spawn` based on specific process management needs.
- Avoid `pm2` for daemonizing Node.js applications on Windows if silent execution and robustness against user invalidation are critical; prefer direct PowerShell approaches.
- Ensure correct argument formatting for PowerShell `-ArgumentList`.
- Implement separate logging for stdout and stderr when redirecting PowerShell output.
- Design Obsidian CLI integrations to account for `ob sync --continuous` being a watcher and use dynamic vault path resolution for portability.
- When implementing file watching with `chokidar` on Windows using polling, include a 600ms settling delay after the 'ready' event.
- For TTY detection in Bun-based applications, use an environment variable instead of relying on `Bun.isTTY`.
- Be aware of the `section-balance` tool's limitation regarding typed brackets and account for this when interpreting its output or designing lyrics inputs.
- When using music generation platforms like Suno, be mindful that certain terms ('skank') might trigger content flags or unexpected interpretations. Avoid descriptive terms ('Building', 'echo/delay/ambient') in prompts that have been been observed to cause undesirable sonic artifacts ('twang', 'wash').
- **When analyzing Suno's analytical outputs (BPM, display tags), account for their non-deterministic nature by potentially averaging multiple readings to reduce noise, especially in drift studies or direct comparisons.**
- Leverage the `minimax-video` tool's enhanced support for local file paths in `--first-frame` arguments for more flexible video generation workflows.

*Consolidated from: knowledge_tool_execution_guidelines.md, knowledge_windows_development.md, 20260401-020156-avoid-using-inline-code-blocks-e-g-bun-e-3.md, 20260401-175949-for-developing-robust-data_parsing_and_m-2.md, 20260403-070905-developing-a-dedicated-cli-router-with-a-1.md, 20260403-082246-bun-run-has-been-established-as-the-stan-2.md, 20260403-082246-the-combination-of-rigorous-mccall-revie-1.md, 20260405-124725-the-section-balance-tool-has-a-specific--2.md, 20260406-081746-the-minimax-video-tool-now-reliably-supp-1.md, 20260406-222827-specific-musical-terms-skank-can-trigger-1.md, 20260408-031043-bun-istty-is-unreliable-always-undefined-3.md, 20260408-031043-it-is-necessary-to-differentiate-between-2.md, 20260408-031043-pm2-on-windows-may-be-susceptible-to-use-1.md, 20260408-031043-when-implementing-file-watching-with-cho-4.md, 20260408-145812-dedicated-cli-tools-for-inspecting-inter-2.md, 20260408-145812-streamlining-human-in-the-loop-processes-3.md, 20260410-021708-suno-s-analytical-outputs-such-as-bpm-an-2.md*
