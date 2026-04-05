---
title: "Eidetic Memory Task 2 — writeRaw() Utility"
type: task
status: pending
parent: 2026-04-01-000700-eidetic-memory-raw-layer
task: 2
depends: []
created: 2026-04-01
author: mccall
---

# Task 2 — writeRaw() Utility

## Objective

Create `src/libs/raw.ts` — the shared utility that writes raw context files to `vault/studio/memory/raw/`. This is a pure file-writer with no brain.db dependency. Any skill or agent calls `writeRaw()` at end-of-workflow.

## What to Build

### File: `src/libs/raw.ts`

```typescript
export interface RawSection {
  name: string;     // "User Input", "Discover", "Variations", etc.
  content: string;  // markdown body of the section
}

export interface RawFileInput {
  agents: string[];          // which agents participated
  skill?: string;            // e.g. "/create-track", "/architect"
  tags?: string[];           // topic tags
  summary?: string;          // one-line summary
  sections: RawSection[];    // ordered sections to write
}

export interface RawFileResult {
  timestamp: number;         // unix timestamp (file identity)
  filePath: string;          // absolute path to written file
}

/**
 * Write a raw context file to memory/raw/{timestamp}.md.
 *
 * Atomic write: writes to .{timestamp}.md.tmp then renames.
 * Omits sections with empty content.
 * Returns { timestamp, filePath } for threading into reflect().
 */
export function writeRaw(input: RawFileInput): RawFileResult { ... }
```

### Behavior

1. Generate `timestamp` from `Math.floor(Date.now() / 1000)`.
2. Compute `date` as `YYYY-MM-DD` from the timestamp.
3. Build frontmatter:
   ```yaml
   ---
   ts: {timestamp}
   date: "{date}"
   agents: [{agents joined}]
   skill: {skill or omit}
   tags: [{tags joined or omit}]
   summary: "{summary or omit}"
   ---
   ```
4. Build body: for each section where `content.trim()` is non-empty, emit `## {name}\n{content}\n\n`.
5. Ensure `vault/studio/memory/raw/` directory exists (`mkdirSync recursive`).
6. Write to `.{timestamp}.md.tmp` (hidden temp file in same directory).
7. Rename `.{timestamp}.md.tmp` to `{timestamp}.md` (atomic on same filesystem).
8. Return `{ timestamp, filePath }` where `filePath` is the absolute path.

### Edge Cases

- If the file already exists (two calls in same second), append a `-1` suffix: `{timestamp}-1.md`. Check with `existsSync` before write.
- Empty `sections` array (all sections empty after trim): still write the file with frontmatter only. The file records that a session happened even if no structured content was captured.

### Imports

Use `fromRoot` from `./paths.js` for the memory root path. Use `fs` for `mkdirSync`, `writeFileSync`, `renameSync`, `existsSync`. Use `dayjs` from `./dayjs.js` for date formatting.

## Files to Create

| File | Purpose |
|------|---------|
| `src/libs/raw.ts` | `writeRaw()` function + types |

## Files Unchanged

No existing files modified.

## Acceptance Criteria

1. `writeRaw({ agents: ["tala"], skill: "/create-track", tags: ["jazz"], summary: "test", sections: [{ name: "User Input", content: "make jazz" }] })` creates `vault/studio/memory/raw/{ts}.md`
2. File has correct YAML frontmatter with ts, date, agents, skill, tags, summary
3. File has `## User Input\nmake jazz\n\n` section
4. Empty sections are omitted from output
5. Atomic write: `.tmp` file does not persist after successful write
6. Return value has correct `timestamp` and `filePath`
7. Calling twice in same second produces two distinct files (suffix handling)
8. `vault/studio/memory/raw/` directory is created if missing

## Test Criteria

```bash
bun -e "
  import { writeRaw } from './src/libs/raw.js';
  const result = writeRaw({
    agents: ['tala', 'echo'],
    skill: '/create-track',
    tags: ['jazz', 'lo-fi'],
    summary: 'Test raw file',
    sections: [
      { name: 'User Input', content: 'Jazz lo-fi track with brush drums' },
      { name: 'Discover', content: 'Matched 3 references' },
      { name: 'Critic Feedback', content: '' },  // should be omitted
    ],
  });
  console.log('Written:', result.filePath);
  console.log('Timestamp:', result.timestamp);
"
```

Verify the file exists and inspect its contents. Then delete the test file.
