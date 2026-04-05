---
id: OT-0005
title: NotebookLM CLI wrapper tool
status: ready
created: 2026-04-01
tags: [notebooklm, research, cli, tools]
---

# OT-0005: NotebookLM CLI Wrapper Tool

## Goal

Build a CLI wrapper at `src/tools/notebooklm/` that shells out to the `notebooklm` Python CLI and provides notebook management, Q&A with stash/save, source management, and file-based search over saved research.

## Architecture Decisions

These are final. Do not deviate.

- **CLI-only.** No `toolDef` export, no registry integration. This is a standalone CLI tool.
- **No brain.db integration.** Search is file-based (glob + grep over vault). brain.db comes in a later task.
- **Thin exec wrapper.** Every call to the external `notebooklm` CLI goes through `exec.ts`. No direct `execSync`/`spawnSync` calls from other modules.
- **JSON mode everywhere.** Always pass `--json` to the underlying CLI when available. Parse structured output, never scrape human-readable text.
- **Last-answer stash.** `ask` writes the full response to `out/notebooklm-last.json` so `save` can pick it up without re-querying.
- **Research vault path.** Saved research goes to `vault/studio/research/{timestamp}-{topic-slug}/`. Reuse existing folder if the topic-slug suffix matches.

## File Structure

```
src/tools/notebooklm/
  index.ts      — CLI entry point + command router
  types.ts      — shared types and interfaces
  exec.ts       — thin wrapper around notebooklm CLI (execSync)
  notebooks.ts  — list, create
  ask.ts        — query notebook + stash to out/
  save.ts       — promote last answer from stash to vault
  sources.ts    — add, list, refresh, delete
  search.ts     — file-based search over vault/studio/research/
```

## File-by-File Spec

### types.ts

```typescript
/** Notebook as returned by `notebooklm list --json` */
export interface Notebook {
  id: string;
  title: string;
}

/** Source as returned by `notebooklm source list --json` */
export interface Source {
  id: string;
  title: string;
  type: string;       // "url" | "text" | "youtube" | "file"
}

/** Structured ask response from `notebooklm ask --json` */
export interface AskResponse {
  answer: string;
  references?: Array<{
    sourceId: string;
    text: string;
  }>;
  conversationId?: string;
}

/** Stash file shape written to out/notebooklm-last.json */
export interface Stash {
  notebook: string;     // notebook ID
  question: string;
  response: AskResponse;
  timestamp: string;    // ISO 8601
}

/** Result from exec wrapper */
export interface ExecResult {
  stdout: string;
  stderr: string;
  exitCode: number;
}
```

Inspect the actual JSON output from `notebooklm list --json` and `notebooklm ask --json` before finalizing these interfaces. The shapes above are best-guess from the CLI help. Adjust field names to match reality, but keep the same logical structure.

### exec.ts

Single function that all other modules call. Pattern: follow `musiccard/extract.ts` for `execSync` usage.

```typescript
import { execSync } from "child_process";
import type { ExecResult } from "./types.js";

/**
 * Run a notebooklm CLI command and return structured result.
 *
 * @param args - Array of CLI arguments (e.g., ["list", "--json"])
 * @param options - Optional: timeout in ms (default 60_000)
 * @returns ExecResult with stdout, stderr, exitCode
 * @throws Error if command not found or times out
 */
export function run(args: string[], options?: { timeout?: number }): ExecResult
```

Implementation notes:
- Command: `notebooklm ${args.join(" ")}` -- the binary is on PATH, no resolution needed.
- Default timeout: 60 seconds. `ask` calls may need longer -- accept override.
- Capture stdout and stderr separately via `stdio: ["pipe", "pipe", "pipe"]`.
- On non-zero exit, still return the ExecResult (don't throw). Let callers decide how to handle errors.
- On timeout or binary-not-found, throw with a clear message.

### notebooks.ts

Two exported functions:

```typescript
import type { Notebook } from "./types.js";

/** List all notebooks. Returns parsed array. */
export function listNotebooks(): Notebook[]

/** Create a notebook. Returns the created notebook. */
export function createNotebook(title: string): Notebook
```

Implementation:
- `listNotebooks()`: calls `run(["list", "--json"])`, parses stdout as JSON, maps to `Notebook[]`.
- `createNotebook(title)`: calls `run(["create", JSON.stringify(title), "--json"])`, parses result.
- Both throw on non-zero exit with the stderr message.

### ask.ts

```typescript
import type { AskResponse, Stash } from "./types.js";

/**
 * Ask a question to a notebook, print the answer, and stash the full response.
 *
 * @param notebookId - Notebook ID (or partial prefix)
 * @param question - The question string
 * @returns The answer text (for CLI display)
 */
export function askNotebook(notebookId: string, question: string): string
```

Implementation:
- Calls `run(["ask", "-n", notebookId, "--json", question], { timeout: 120_000 })`.
- Parses stdout as `AskResponse`.
- Builds a `Stash` object with notebook ID, question, full response, and ISO timestamp.
- Writes stash to `fromRoot("out", "notebooklm-last.json")` using `writeFileSync`. Create `out/` if needed.
- Returns the `answer` field for display.
- On non-zero exit, throw with stderr.

### save.ts

```typescript
/**
 * Save the last stashed answer to the vault as a research markdown file.
 *
 * @param notebookId - Notebook ID to validate against stash (must match)
 * @param topic - Topic name for folder slug. Defaults to notebook title if omitted.
 * @returns Absolute path to the saved file.
 */
export function saveLastAnswer(notebookId: string, topic?: string): string
```

Implementation:
1. Read `out/notebooklm-last.json`. Throw if missing ("No stashed answer. Run `ask` first.").
2. Parse as `Stash`. Validate `stash.notebook` matches `notebookId` (prefix match is fine, same as the CLI). Throw if mismatch.
3. Derive `topicSlug` from `topic` param (or fall back to notebook title via `listNotebooks()` lookup). Slugify: lowercase, replace non-alphanumeric with hyphens, collapse, trim, max 40 chars.
4. Build research directory: `vault/studio/research/`. Scan existing folders for any ending in `-{topicSlug}`. If found, reuse that folder. Otherwise create `{YYYY-MM-DD-HHmmss}-{topicSlug}/`.
5. Derive answer slug from the first 40 chars of the question. Write file as `{YYYY-MM-DD-HHmmss}-{answerSlug}.md`.
6. File content format:

```markdown
---
source: notebooklm
notebook: {notebookId}
topic: {topic}
question: "{question}"
date: {YYYY-MM-DD}
---

# {question}

{answer text}

## References

{numbered list of references if present, otherwise omit section}
```

7. Return the absolute path to the written file.

Use `dayjs` from `../../libs/dayjs.js` for all timestamps. Use `fromRoot()` for all paths.

### sources.ts

Four exported functions:

```typescript
import type { Source } from "./types.js";

/** Add a source to a notebook. Auto-detects type (URL, file, text). */
export function addSource(notebookId: string, content: string): Source

/** List all sources in a notebook. */
export function listSources(notebookId: string): Source[]

/** Refresh a source (URL/Drive only). */
export function refreshSource(notebookId: string, sourceId: string): void

/** Delete a source. */
export function deleteSource(notebookId: string, sourceId: string): void
```

Implementation:
- `addSource`: calls `run(["source", "add", "-n", notebookId, "--json", content])`. If content starts with `http`, it goes through as URL/YouTube auto-detect. If it's a file path (check `existsSync`), pass it. Otherwise treat as inline text.
- `listSources`: calls `run(["source", "list", "-n", notebookId, "--json"])`.
- `refreshSource`: calls `run(["source", "refresh", "-n", notebookId, sourceId])`. No `--json` available for this command.
- `deleteSource`: calls `run(["source", "delete", "-n", notebookId, "-y", sourceId])`. The `-y` skips confirmation.
- All throw on non-zero exit with stderr.

### search.ts

```typescript
export interface SearchResult {
  file: string;       // absolute path
  topic: string;      // folder name
  question: string;   // from frontmatter
  date: string;       // from frontmatter
  matchLine?: string; // first matching line (content search)
}

/**
 * Search saved research files by query string.
 * Searches filenames and file contents under vault/studio/research/.
 *
 * @param query - Search term
 * @returns Array of matching results
 */
export function searchResearch(query: string): SearchResult[]
```

Implementation:
- Base path: `fromRoot("vault", "studio", "research")`.
- If directory doesn't exist, return empty array.
- Recursively glob for `**/*.md` files in research dir.
- For each file: read content, check if query appears in filename OR content (case-insensitive).
- Parse frontmatter (simple regex: between `---` fences, extract `question:` and `date:` fields). Do NOT pull in a YAML parser dependency -- regex is sufficient for our controlled frontmatter.
- Return matches sorted by date descending.
- Cap at 20 results.

### index.ts

CLI entry point. Pattern: follow `musiccard/index.ts` structure.

```typescript
#!/usr/bin/env bun
```

**Command router using `process.argv`:**

| CLI invocation | Dispatches to |
|---|---|
| `bun src/tools/notebooklm/index.ts list` | `notebooks.listNotebooks()` |
| `bun src/tools/notebooklm/index.ts create --title "Name"` | `notebooks.createNotebook(title)` |
| `bun src/tools/notebooklm/index.ts ask --notebook <id> "question"` | `ask.askNotebook(id, question)` |
| `bun src/tools/notebooklm/index.ts save --notebook <id>` | `save.saveLastAnswer(id)` |
| `bun src/tools/notebooklm/index.ts save --notebook <id> --topic "Name"` | `save.saveLastAnswer(id, topic)` |
| `bun src/tools/notebooklm/index.ts search "query"` | `search.searchResearch(query)` |
| `bun src/tools/notebooklm/index.ts source add --notebook <id> --url <url>` | `sources.addSource(id, url)` |
| `bun src/tools/notebooklm/index.ts source add --notebook <id> --file <path>` | `sources.addSource(id, path)` |
| `bun src/tools/notebooklm/index.ts source list --notebook <id>` | `sources.listSources(id)` |
| `bun src/tools/notebooklm/index.ts source refresh --notebook <id> --source <id>` | `sources.refreshSource(nbId, srcId)` |
| `bun src/tools/notebooklm/index.ts source delete --notebook <id> --source <id>` | `sources.deleteSource(nbId, srcId)` |

Implementation:
- Guard with `if (import.meta.main)`.
- Parse args manually (same pattern as musiccard). No arg-parsing library.
- First positional arg is the command. For `source`, second positional is the subcommand.
- On unknown command or missing required args, print usage and exit 1.
- Wrap all calls in try/catch. On error, print `[notebooklm error] {message}` to stderr and exit 1.
- On success, print result to stdout. For structured data (list, source list), format as a readable table. For ask, print the answer text directly.

**Usage text:**

```
NotebookLM CLI wrapper

Usage:
  bun src/tools/notebooklm/index.ts list
  bun src/tools/notebooklm/index.ts create --title "Name"
  bun src/tools/notebooklm/index.ts ask --notebook <id> "question"
  bun src/tools/notebooklm/index.ts save --notebook <id> [--topic "Name"]
  bun src/tools/notebooklm/index.ts search "query"
  bun src/tools/notebooklm/index.ts source add --notebook <id> --url <url>
  bun src/tools/notebooklm/index.ts source add --notebook <id> --file <path>
  bun src/tools/notebooklm/index.ts source list --notebook <id>
  bun src/tools/notebooklm/index.ts source refresh --notebook <id> --source <id>
  bun src/tools/notebooklm/index.ts source delete --notebook <id> --source <id>
```

## Acceptance Criteria

Every item below must work from the CLI. Test manually -- no test framework (Bun test is broken on Windows).

1. **`list`** -- prints a table of notebooks with ID and title. Returns exit 0.
2. **`create --title "Test Notebook"`** -- creates notebook, prints its ID and title. Verify with `list`.
3. **`ask --notebook <id> "What is this about?"`** -- prints answer to stdout. Verify `out/notebooklm-last.json` exists and contains valid Stash JSON.
4. **`save --notebook <id>`** -- creates file under `vault/studio/research/`. Verify markdown has correct frontmatter and content. Verify topic folder was created with timestamp prefix.
5. **`save --notebook <id> --topic "My Research"`** -- uses "my-research" as topic slug. If run twice, reuses the same folder (does not create a second one).
6. **`search "keyword"`** -- returns matching files from research dir. Returns empty gracefully if no research exists yet.
7. **`source add --notebook <id> --url https://example.com`** -- adds URL source, prints confirmation.
8. **`source add --notebook <id> --file ./some-file.md`** -- adds file source.
9. **`source list --notebook <id>`** -- prints table of sources with ID, title, type.
10. **`source refresh --notebook <id> --source <id>`** -- refreshes, prints confirmation.
11. **`source delete --notebook <id> --source <id>`** -- deletes, prints confirmation.
12. **Unknown command** -- prints usage, exits 1.
13. **Missing `--notebook`** on commands that require it -- prints error, exits 1.
14. **`ask` with no stash then `save`** -- prints clear error about missing stash.

## Files to Reference

Study these before implementing. They define the patterns to follow.

| File | What to learn |
|------|---------------|
| `src/tools/musiccard/index.ts` | CLI entry point pattern, `import.meta.main` guard, arg parsing loop, error handling |
| `src/tools/musiccard/extract.ts` | `execSync` usage, binary resolution, timeout handling, `stdio` config |
| `src/libs/paths.ts` | `fromRoot()` helper, `PROJECT_ROOT` constant |
| `src/libs/dayjs.ts` | Import as `import dayjs from "../../libs/dayjs.js"` |

## Out of Scope

Do NOT build any of these:

- **No `toolDef` export.** This is CLI-only. Registry integration is a separate task.
- **No brain.db integration.** Search is file-based only. brain.db indexing of research comes later.
- **No Gemini/AI post-processing.** Raw answers from NotebookLM are saved as-is.
- **No `notebooklm login`/`auth` wrapping.** Auth is handled directly by the user via `notebooklm login`.
- **No artifact/audio/note subcommands.** Only the commands listed in the architecture table.
- **No tests.** Manual CLI testing only (Bun test is broken on Windows).
- **No new dependencies.** Use only what's already in package.json (bun built-ins, dayjs, path, fs).
