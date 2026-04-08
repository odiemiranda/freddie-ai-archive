---
name: Windows Daemon Stability Patterns
description: Best practices and workarounds for building stable daemon processes on Windows, including process management and lease/heartbeat patterns.
type: knowledge
agent: holmes
tags: [Windows, Daemon, Stability, Bun, Chokidar, Lease, Heartbeat, Architecture, Process Management]
---

## Windows Daemon Stability Patterns

This document consolidates findings related to ensuring stability and robustness for background daemon processes running on Windows systems.

### Bun Worker Thread Instability on Windows

**Why:**
During ASL Phase 2 research, `Bun worker_threads` were found to be unstable when deployed on Windows environments. This instability can lead to unpredictable behavior, crashes, and unreliable background processing, hindering the overall system's robustness.

**How to apply:**
Avoid using `Bun worker_threads` for critical background tasks on Windows. Instead, implement `child processes` for concurrency and isolation. This provides a more stable and predictable execution environment on Windows until `Bun worker_threads` stability issues are resolved.

### Chokidar v4 Broken on Windows

**Why:**
`Chokidar v4`, a file watching library, was identified as broken on Windows during research. Using this version can lead to file change events not being detected reliably, or other unexpected behavior, impacting systems that rely on real-time file system monitoring.

**How to apply:**
Pin `Chokidar` to version `v3.6.0` in your project dependencies when targeting Windows. This ensures stable and functional file system watching capabilities. Monitor future `Chokidar` releases for fixes to v4 (or later) on Windows before attempting to upgrade.

### Lease Anti-Pattern and Heartbeat Solution

**Why:**
A fixed 5-minute lease duration for daemon processes was identified as an anti-pattern. Fixed, long leases can lead to inefficient resource utilization, slow recovery from failures, and potential race conditions if not managed carefully. A more dynamic and responsive lease management strategy is required for robust distributed systems.

**How to apply:**
Replace fixed, long leases with a shorter 60-second lease combined with a 30-second heartbeat mechanism.
-   **60-second lease:** The primary lease duration, indicating a process's active claim on a resource or task.
-   **30-second heartbeat:** A periodic signal sent by the active process within its lease period to confirm its continued operation and renew its claim.
This pattern allows for faster detection of inactive processes (if heartbeats stop) and more efficient re-assignment of tasks, improving system resilience and responsiveness.

---
*Consolidated from: `raw-log-20260407-194923-note.md`*
