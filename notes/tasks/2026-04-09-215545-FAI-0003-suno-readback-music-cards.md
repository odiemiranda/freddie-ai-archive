---
project-code: FAI
task-id: FAI-0003
status: todo
agent: wick
created: 2026-04-09
title: Suno readback in music cards + local file input
---

# FAI-0003 — Suno readback in music cards + local file input

## Summary

Integrate the FAI-0002 `suno-meta` extractor into the `/create-music-card` pipeline so every new music card captures Suno's own lyrics + style prose + display tags + full raw response alongside librosa analysis and Gemini refinement. At the same time, expand `musiccard` to accept local audio files via `--file`, and widen `suno-api` to accept both `.mp3` and `.wav`. Purpose: build a clean corpus for studying Suno's interpretation drift against librosa ground truth.

## Context

FAI-0002 shipped `suno-meta` (commit `ae2a570`) — a working tool that uploads audio to Suno's upload-audio API and extracts Suno's own Lyrics + Style prose. Core API logic lives in `src/libs/suno-api/index.ts`, exported as `extractMeta(filePath): Promise<SunoExtractResult>`. Single extraction ~40-50s, currently mp3-only.

FAI-0003 uses that extractor as a first-class step inside the music-card workflow. The user is wiping all existing music cards and starting clean — no backfill, no migration. Every new card going forward will carry Suno's unfiltered readback so we can empirically study where Suno drifts from librosa/Gemini over a growing corpus.

A secondary goal: unblock local-file music cards. Until now cards could only be built from YouTube URLs. With file mode, we can point the tool at any local `.mp3` or `.wav` (including bounced tracks from `vault/studio/tracks/`) and get the full card + Suno readback pipeline.

## Non-Negotiable Ground Rules

1. **Suno data does NOT feed Gemini refinement. Ever.** `src/tools/musiccard/refine.ts` is OFF-LIMITS for this task. Do not modify the prompt, the signature, the call shape, or the inputs passed to Gemini. The entire point of building this corpus is to study Suno's interpretation drift against librosa ground truth. Laundering Suno's own readback through Gemini's refinement step would contaminate the study at the source. If you are tempted to "enrich" the Gemini prompt with `sunoResult.style_prose` or `sunoResult.display_tags` because it seems helpful — STOP. That is the bug this rule exists to prevent. This is non-negotiable and applies to any future contributor as well.

2. **Suno readback must NEVER fail the card.** Missing env vars, 401 auth rejection, network errors, timeouts, or any other SunoError path must degrade gracefully: log a warning line, skip the Suno step, continue building the card with librosa + Gemini only. The card is still valid without Suno fields — it just loses the drift-study value. A hard failure here would block the user from building cards whenever Suno tokens expire, which is unacceptable.

3. **No format conversion, no transcoding.** We accept `.mp3` or `.wav` as provided, pass them through as-is. If the user hands us a `.flac` or `.m4a`, we hard-error at validation. Do not introduce ffmpeg transcoding, do not auto-convert.

4. **No backfill.** Existing music cards are being wiped by the user. Do not write migration code, do not attempt to add suno_* frontmatter to old cards.

5. **CLI only, no MCP.** Tools are invoked via `bun run tool musiccard ...`. Never call via MCP from local workflows.

## Architecture Decisions (Locked)

These were decided by McCall + user before this task was written. Do not revisit.

### AD-1: Suno readback runs inside `musiccard`, not as a skill orchestration step

Native import of `extractMeta()` from `src/libs/suno-api/index.ts`. No subprocess shelling to `suno-meta` CLI. This keeps the tool self-contained, avoids process-spawn overhead, and lets us share the `SunoError` type for structured error handling. The skill workflow simply calls `bun run tool musiccard ...` and the tool handles the readback internally.

### AD-2: Pipeline ordering

```
metadata/extract → librosa → Suno readback (NEW) → build card → Gemini refine
```

Suno readback slots in AFTER librosa (so librosa always runs, even if Suno fails) and BEFORE card build (so the card renderer has `sunoResult` in hand). Gemini refinement runs last, unchanged, and does not see `sunoResult`.

### AD-3: Suno data does NOT feed Gemini refinement

See Non-Negotiable Ground Rule #1 above. This is an architectural constraint, not a stylistic preference. `refine.ts` stays byte-identical.

### AD-4: Graceful degradation on Suno failure

Map `SunoError` exit codes to warning messages:

| exitCode | Meaning | Warning message |
|----------|---------|-----------------|
| 1 | Missing env / validation | `Suno readback skipped — {err.message}` |
| 2 | 401 auth rejected | `Suno readback skipped — Suno auth expired. Refresh tokens via `suno-meta --setup-help`.` |
| other / unknown | Network, timeout, etc. | `Suno readback skipped — {err.message}` |

Non-`SunoError` errors (generic `Error`) also fall through to the "skipped — {message}" warning. Never rethrow inside the Suno block.

### AD-5: `--no-suno` opt-out flag

Default is enabled. `--no-suno` skips the readback step entirely (no API call, no warning line, no suno_* frontmatter, no Suno sections in card body). Useful for fast iteration and for testing the non-Suno code path.

### AD-6: Multi-format support in `suno-api` (mp3 + wav)

- `validateInputs()`: expand allowlist to `[".mp3", ".wav"]`.
- `step1CreateUploadSession()`: accept `extension: "mp3" | "wav"` parameter, thread into `JSON.stringify({ extension })` body.
- `extractMeta()`: derive extension from filepath via `extname().toLowerCase().slice(1)` cast to `"mp3" | "wav"`, pass to step1.
- Steps 2 (S3 upload) and 3 (upload-finish) are already format-agnostic (they use the real filename via `basename()` / the `upload_filename` param) — NO changes to those functions.
- No conversion. Wrong extension → hard error from `validateInputs`.
- Unit tests added for wav path (see file spec below). Mock the HTTP layer — do NOT hit the real Suno API in tests.

### AD-7: Local file input mode for `musiccard`

Add `--file <path>` as an alternative to the positional YouTube URL arg. Mutually exclusive with URL mode — exactly one must be provided.

File mode behavior:
- Skips yt-dlp and ffmpeg trim entirely (file is used as-is, no re-encoding)
- Copies file into `vault/studio/references/shared/music-cards/audio/YYYY-MM-DD-HHMMSS-<slug>.<ext>` preserving original extension
- Probes duration via `ffprobe` for frontmatter
- `--title` is REQUIRED in file mode (no YouTube metadata). Missing → hard error.
- `--start` / `--duration` are IGNORED in file mode. If either is passed with `--file`, emit a warning (do not error).
- Only `.mp3` and `.wav` accepted. Other extensions → hard error before any copy or upload.

### AD-8: `--track` arg (file mode only)

Optional. Links the card back to a studio track document. Accepts either:

- Wikilink format `[[track-filename]]` → stored verbatim in frontmatter
- Repo-relative path to a `.md` file under `vault/studio/tracks/**` → converted to `[[<basename-without-.md>]]` form

Validation: the target `.md` file must exist under `vault/studio/tracks/`. Hard error before any upload or copy if it does not. `--track` is ignored in YouTube mode; emit a warning if passed.

## Frontmatter Spec

```yaml
title: "..."
type: background-mix | music-video | single-track | live-performance
source: youtube | file
source_url: https://...          # present only when source=youtube
source_file: relative/path.mp3   # present only when source=file (repo-relative path to copied audio)
track: "[[track-filename]]"      # present only when source=file AND user provided --track; omit otherwise
channel: "..."                   # youtube only
sample: "5:00–10:00"              # youtube only — omit in file mode
duration: 5m00s
created: 2026-04-09
tags: []
suno_has_vocal: true              # all suno_* fields present ONLY if readback succeeded
suno_duration_seconds: 298
suno_display_tags: "jazz, shamisen, lo-fi"
suno_extracted_at: 2026-04-09T21:55:45.123Z
```

Rules:
- The old single-field `source: <url>` shape is being replaced. Clean slate, no migration.
- When `sunoResult` is null (failed or `--no-suno`): emit NO `suno_*` fields. Do not write empty strings, do not write `null`, do not write placeholder values. Just omit.
- In YouTube mode, omit `source_file` and `track`.
- In file mode, omit `source_url`, `channel`, and `sample`.

## Card Body Additions

Two new sections inserted between the existing `## MiniMax Translation` and `## Notes` sections. Both are skipped entirely (zero bytes emitted) when `sunoResult` is null.

### `## Suno Readback`

```markdown
## Suno Readback

**Display tags:** jazz, shamisen, lo-fi
**Has vocal:** yes | **Duration:** 4:58

### Style (Suno's prose)
```
<result.style_prose verbatim>
```

### Lyrics (Suno's transcription)
```
<result.lyrics verbatim>
```
```

Wrap `style_prose` and `lyrics` in fenced code blocks (triple backtick, no language tag). Reasons:
- Prevents Obsidian from treating `[Verse]` / `[Chorus]` / `[Bridge]` bracket markers as broken wikilinks
- Preserves exact whitespace and line breaks
- Enables clean copy-paste back into Suno or another tool

Render "Has vocal" as `yes` / `no` (not `true` / `false`). Render duration from `suno_duration_seconds` as `M:SS` (integer seconds → `Math.floor(sec/60) + ":" + pad(sec%60)`).

### `## Suno Raw Response`

```markdown
## Suno Raw Response

<details>
<summary>Full SunoExtractResult JSON (click to expand)</summary>

```json
{ ...full SunoExtractResult, JSON.stringify with 2-space indent... }
```

</details>
```

`<details>` / `<summary>` renders natively in Obsidian live preview and reading view. Users can expand when they want the raw data, ignore otherwise.

## File-by-File Spec

### 1. `src/libs/suno-api/index.ts` (MODIFY)

Read the file first. Relevant line ranges:
- `72-84`: `SunoExtractResult` interface — no changes needed, but re-export via this file is fine (it already is)
- `158-188`: `validateInputs()` — the mp3-only gate is here
- `209-244`: `step1CreateUploadSession()` — hardcoded `{ extension: "mp3" }` on line 213
- `248-283`: `step2...` (S3 upload) — format-agnostic, DO NOT touch
- `287-303`: `step3...` (upload-finish) — format-agnostic, DO NOT touch

Changes:

**`validateInputs(filePath: string)`**
- Change the extension check from `ext !== ".mp3"` to `![".mp3", ".wav"].includes(ext)`.
- Update the error message to list both accepted formats, e.g. `Unsupported audio format: ${ext}. Expected .mp3 or .wav.`
- Keep the `ext = extname(filePath).toLowerCase()` line as-is.
- All other validation (file exists, file is readable, file size bounds if any) stays untouched.

**`step1CreateUploadSession(extension: "mp3" | "wav")`**
- Add `extension` as the first (and only) parameter. Type it literally as `"mp3" | "wav"` — do not widen to `string`.
- Replace the hardcoded body `JSON.stringify({ extension: "mp3" })` with `JSON.stringify({ extension })`.
- Everything else in step1 (URL, headers, fetch call, response parsing, error handling) stays identical.
- If step1 currently has a JSDoc block, update it to mention the parameter.

**`extractMeta(filePath: string)`**
- After `validateInputs(filePath)` succeeds, derive the extension:
  ```ts
  const ext = extname(filePath).toLowerCase().slice(1) as "mp3" | "wav";
  ```
- Pass `ext` into `step1CreateUploadSession(ext)`.
- No other changes. Steps 2 and 3 keep their existing call signatures.

**Do NOT change:**
- `SunoError` class
- `SunoExtractResult` interface
- Browser fingerprint headers
- The 5s wait before initialize-clip (known gotcha from FAI-0002 memory)
- Trailing-slash handling in URLs (known gotcha)
- Authorization header shape

### 2. `src/tools/__tests__/suno-meta.test.ts` (MODIFY)

Read first. Keep all existing mp3 tests green. Add new test cases:

- **`validateInputs accepts .wav`** — mock `existsSync` / file stat to return true, call `validateInputs("some/file.wav")`, assert no throw.
- **`validateInputs rejects .flac`** — same mocks, call `validateInputs("some/file.flac")`, assert throws with message mentioning both `.mp3` and `.wav`.
- **`validateInputs rejects .m4a`** — same pattern, different extension, same assertion shape.
- **`step1 payload reflects wav extension`** — mock the `fetch` call, invoke `step1CreateUploadSession("wav")`, assert the mock was called with a body whose parsed JSON equals `{ extension: "wav" }`. Also add the mirror assertion for `"mp3"` if not already covered.
- **`extractMeta threads extension through to step1 for wav`** — mock all three step functions, call `extractMeta("path/to/audio.wav")`, assert `step1` spy was called with `"wav"`.

Use mocks for all HTTP calls. Do NOT hit the real Suno API. If the existing test file uses `mock.module` or a manual fetch stub, follow the same pattern for consistency.

Note: the user flagged in `project_bun_test_broken.md` that `bun test` has a Windows segfault issue on imports from modified modules. If tests cannot be executed via `bun test` on Windows, document that in the Ryan handoff notes and run via `bun run` against a small harness script instead — but still write the test cases so they run in CI / on other platforms.

### 3. `src/tools/musiccard/index.ts` (MODIFY)

Read first to understand current `MusicCardArgs` shape and `createMusicCard()` flow.

**Type + schema changes:**
- Extend `MusicCardArgs` type with: `file?: string`, `track?: string`, `noSuno?: boolean`.
- Extend `toolDef.inputSchema` (zod) with matching optional fields. `file` is a string path, `track` is a string, `noSuno` is a boolean.
- Keep `url` optional now (it used to be effectively required — now the mutual exclusivity check handles that).

**CLI flag parsing:**
- Add `--file <path>` → `args.file`
- Add `--track <wikilink-or-path>` → `args.track`
- Add `--no-suno` → `args.noSuno = true`
- Keep all existing flags (`--start`, `--duration`, `--type`, `--slug`, `--title`, `--genre`, `--mood`, etc.) unchanged.

**At the top of `createMusicCard()`:**
- Validate mutual exclusivity: exactly one of `args.url` or `args.file` must be truthy. Both set → hard error. Neither set → hard error with usage hint.
- If `args.file` is set and `args.title` is missing → hard error (`--title is required in file mode`).
- If `args.file` is set and `args.start` or `args.duration` is set → emit a warning to the `out` array (`Warning: --start/--duration ignored in file mode`), do not error.
- If `args.url` is set and `args.track` is set → emit a warning (`Warning: --track ignored in YouTube mode`), do not error.

**Branch on mode:**

YouTube mode (`args.url`): existing behavior through the extract phase. No changes.

File mode (`args.file`):
- Call a new helper `extractLocalFile(args.file, audioDir, audioSlug)` from `extract.ts` (see file 4 below). It returns `{ audioFile, fileSizeMB, actualDuration }` — same shape as the existing `downloadAndTrim()` return value, so the rest of the pipeline is interchange-compatible.
- Build a synthetic `YtMeta`-shaped object:
  ```ts
  const meta: YtMeta = {
    title: args.title!,   // already validated non-null above
    channel: "",
    duration: extract.actualDuration,
    tags: [],
    description: "",
  };
  ```
- If `args.track` is set, resolve it via `resolveTrackWikilink(args.track)` (new helper, defined below in this file or a small util) before any Suno call. This way a bad `--track` hard-errors before we spend 40-50s uploading.

**After librosa analyze (both modes), add the Suno readback block:**

```ts
import { extractMeta, SunoError, type SunoExtractResult } from "../../libs/suno-api/index.js";

// ...

let sunoResult: SunoExtractResult | null = null;
if (!args.noSuno) {
  out.push("Extracting Suno readback...");
  try {
    sunoResult = await extractMeta(extract.audioFile);
    out.push(`  Suno: has_vocal=${sunoResult.has_vocal}, tags=${sunoResult.display_tags}`);
  } catch (err) {
    const msg = err instanceof SunoError
      ? (err.exitCode === 2
          ? "Suno auth expired. Refresh tokens via `suno-meta --setup-help`."
          : err.message)
      : (err as Error).message;
    out.push(`  Suno readback skipped — ${msg}`);
  }
}
```

Place this AFTER librosa analysis and BEFORE `buildCard()`. Never let an exception escape this block.

**Pass new fields to `buildCard()`:**
- `source: args.file ? "file" : "youtube"`
- `sourceUrl: args.url` (undefined in file mode)
- `sourceFile: args.file ? <repo-relative-path-to-copied-audio> : undefined`
- `track: resolvedTrackWikilink` (undefined in YouTube mode or when `--track` not provided)
- `sunoResult`

For `sourceFile`, use the path of the copied audio relative to the repo root (use forward slashes for Obsidian compatibility), e.g. `vault/studio/references/shared/music-cards/audio/2026-04-09-215545-my-slug.wav`.

**New helper `resolveTrackWikilink(input: string): string`:**

Place in this file (or a small util module if there's already a util file in `src/tools/musiccard/`). Behavior:
- If `input.startsWith("[[") && input.endsWith("]]")` → return `input` verbatim.
- Otherwise treat `input` as a repo-relative path:
  - Resolve against repo root
  - Verify the file exists AND is under `vault/studio/tracks/` (use `path.relative` and check it does not start with `..`)
  - If not found or outside tracks dir → throw `Error("--track target not found under vault/studio/tracks/: " + input)`
  - Extract basename without `.md` extension
  - Return `"[[" + basename + "]]"`

Call this BEFORE the Suno extraction step so a bad track path fails fast.

### 4. `src/tools/musiccard/extract.ts` (MODIFY)

Read first to understand current `downloadAndTrim()` signature and return shape.

**Add `probeDurationSeconds(filePath: string): Promise<number>` (or sync — match existing ffmpeg helper style):**
- Shell out to ffprobe: `ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 "<file>"`
- Parse stdout as float (seconds).
- If ffprobe is not on PATH → throw a clear error: `ffprobe not found. Install ffmpeg to enable local file music cards.`
- If the file is unreadable or ffprobe returns non-zero → throw with the stderr included.
- Use the same subprocess primitive the existing ffmpeg trim code uses (likely `Bun.spawn` or `spawnSync`) for consistency.

**Add `extractLocalFile(srcPath: string, audioDir: string, audioSlug: string): Promise<{ audioFile: string; fileSizeMB: number; actualDuration: number }>`:**

Mirrors `downloadAndTrim()`'s return shape so callers can interchange them.

Steps:
1. Verify `srcPath` exists. Throw clear error if not.
2. Get extension via `extname(srcPath).toLowerCase()`. If not in `[".mp3", ".wav"]` → throw `Unsupported local file format: ${ext}. Expected .mp3 or .wav.`
3. Build destination path: `join(audioDir, audioSlug + ext)`.
4. Ensure `audioDir` exists (mkdir -p).
5. Copy file from `srcPath` to destination. Use `Bun.write(dest, Bun.file(src))` or `fs.copyFile`. Do not re-encode.
6. Probe duration via `probeDurationSeconds(dest)`.
7. Stat destination for size, convert bytes → MB with 1 decimal place (`(bytes / 1024 / 1024).toFixed(1)` → parseFloat).
8. Return `{ audioFile: dest, fileSizeMB, actualDuration }`.

Do not alter the existing `downloadAndTrim()` function. Add `extractLocalFile` and `probeDurationSeconds` alongside it.

### 5. `src/tools/musiccard/card.ts` (MODIFY)

Read first to understand the current `BuildCardInput` type and the section order in the rendered template.

**Extend `BuildCardInput`:**
- `source: "youtube" | "file"`
- `sourceUrl?: string`
- `sourceFile?: string`
- `track?: string`
- `sunoResult?: SunoExtractResult | null`

Import `SunoExtractResult` type from `../../libs/suno-api/index.js` (type-only import).

**Frontmatter emission:**

Branch on `input.source`:

YouTube branch:
- `source: youtube`
- `source_url: <input.sourceUrl>`
- `channel: <existing>`
- `sample: <existing>`
- (no `source_file`, no `track`)

File branch:
- `source: file`
- `source_file: <input.sourceFile>`
- `track: <input.track>` ONLY if `input.track` is set (omit the line entirely otherwise)
- (no `source_url`, no `channel`, no `sample`)

Suno fields (both branches):
- If `input.sunoResult != null`, emit:
  ```yaml
  suno_has_vocal: <bool>
  suno_duration_seconds: <number>
  suno_display_tags: "<string>"
  suno_extracted_at: <ISO string>
  ```
  Use `new Date().toISOString()` for `suno_extracted_at` if the result does not carry its own timestamp. If `SunoExtractResult` already has an extracted-at field, use that instead — check the interface.
- If `input.sunoResult == null` → emit NONE of the `suno_*` fields.

**New render helpers:**

`renderSunoReadback(result: SunoExtractResult | null): string`
- Returns empty string if `result` is null.
- Returns the full `## Suno Readback` markdown block as specified in the Card Body Additions section above.
- Format "has vocal" as `yes` / `no`.
- Format duration as `M:SS`.
- Wrap `style_prose` and `lyrics` each in their own fenced code block (triple backtick, no language tag).

`renderSunoRawResponse(result: SunoExtractResult | null): string`
- Returns empty string if `result` is null.
- Returns the full `## Suno Raw Response` block with a `<details>` / `<summary>` collapsible containing the result JSON formatted via `JSON.stringify(result, null, 2)` inside a ` ```json ... ``` ` fenced block.

**Insert into card template:**

Between the existing `## MiniMax Translation` and `## Notes` sections, in this order:
1. `## Suno Readback` (from `renderSunoReadback`)
2. `## Suno Raw Response` (from `renderSunoRawResponse`)

If both helpers return empty strings, there should be zero extra blank lines left behind in the output (conditionally join with `\n\n` only when non-empty).

### 6. `src/tools/musiccard/refine.ts` (DO NOT MODIFY)

Do not open this file in an editor with intent to change it. Do not add a parameter. Do not add `sunoResult` to the Gemini prompt. Do not adjust the signature. Do not import anything from `suno-api`.

Reason: see Non-Negotiable Ground Rule #1. This is load-bearing for the drift-study corpus.

You may read it if you need to understand the Gemini call shape for debugging — but write changes are out of scope for this task.

### 7. `.claude/skills/create-music-card/SKILL.md` (MODIFY)

Read first.

**Description:** Update to mention local file input and Suno readback. Stay under the 250-char limit for skill descriptions.

**Input collection:**
- Question 1: change from "YouTube URL" to "Source: YouTube URL or local file path (.mp3 or .wav)"
- Questions about `--start` / `--duration` (likely Q5 and Q6): annotate "YouTube mode only — ignored for file input"
- Add Question 7: "Track (file mode only): wikilink like `[[track-name]]` or relative path to a vault track `.md` file, or empty to skip"

**Phase 2 description:** Mention that Suno readback now runs inside the tool (+40-50s).

**Output section:** List the new frontmatter fields (`source`, `source_url` / `source_file`, `track`, `suno_*`) and the two new card body sections (`## Suno Readback`, `## Suno Raw Response`).

**Examples:** Show both a YouTube-mode invocation and a file-mode invocation. File-mode example should include `--title` and optionally `--track`.

### 8. `.claude/skills/create-music-card/workflow.md` (MODIFY)

Read first.

**P1-001 (Fetch YouTube metadata):** Add an upstream branch — if input 1 is a local file path, skip yt-dlp entirely. Validate file exists and extension is `.mp3` or `.wav`. Probe duration via ffprobe. Synthesize the metadata object. Consider renaming the node to reflect dual-mode (e.g., "Resolve source metadata").

**P1-002 (Resolve inputs):** Add file-mode branch:
- `--title` must be explicit (not auto-derived)
- `--start` / `--duration` are N/A
- `--track` is collected and validated (target `.md` must exist under `vault/studio/tracks/`)
- Present a different summary table for file mode (no channel, no sample range, shows file path and optional track wikilink)

**P2-001 (Download + trim + analyze):** Document dual-mode tool invocation:

YouTube mode:
```
bun run tool musiccard "<url>" --start <s> --duration <d> --type <t> --slug <slug> [--genre "<g>"] [--mood "<m>"]
```

File mode:
```
bun run tool musiccard --file "<path>" --title "<title>" --type <t> --slug <slug> [--track "<wikilink-or-path>"] [--genre "<g>"] [--mood "<m>"]
```

Mention that Suno readback runs inside the tool and adds ~40-50s to execution time. Mention `--no-suno` as an opt-out for fast iteration.

**P3-001 (Present draft card):** Add a Suno readback preview block to the presentation template. Show:
- Display tags
- `Has vocal` line
- Style prose truncated to first 300 chars, append `...` if longer
- Lyrics truncated to first 200 chars, append `...` if longer
- Note that the full values live in the card file (`## Suno Readback` and `## Suno Raw Response` sections)

If `--no-suno` was used or the Suno step failed, show a single line like "Suno readback: skipped ({reason})" instead.

**Mermaid graph:** If the workflow has a mermaid diagram, update it to reflect the source-type branching at the top (YouTube vs file input → unified extract → librosa → Suno → card → refine).

## Scope Exclusions

These are explicitly OUT OF SCOPE for FAI-0003. Do not implement any of them, even if they seem "while we're here" easy:

- **No backfill.** Existing cards are being wiped. No migration scripts, no "upgrade old cards" tool.
- **No format conversion / transcoding.** Hard error on unsupported formats.
- **No formats beyond `.mp3` and `.wav`.** Not flac, not m4a, not ogg, not opus.
- **No changes to `refine.ts`.** Gemini refinement is untouched.
- **No changes to the Gemini prompt.** Do not sneak `sunoResult` into the prompt.
- **No streaming / progress bar for the Suno step.** A single "Extracting Suno readback..." line and a result/error line is sufficient.
- **No retry logic for Suno failures.** A 401 or network error skips the step — we do not retry with backoff. If the user wants a retry, they rerun the tool.
- **No changes to brain.db schema or sync logic.** Music cards will continue to be indexed the same way; the new frontmatter fields just flow through the existing sync.
- **No Suno API quota / rate limit handling beyond what FAI-0002 already provides.**

## Acceptance Tests

Ryan runs these manually after implementation. All must pass before this task is marked done.

1. **YouTube URL → full card.** Run with a known-good YouTube URL. Expect:
   - `source: youtube` in frontmatter
   - `source_url`, `channel`, `sample` present
   - `suno_*` fields present (assuming Suno tokens are valid)
   - `## Suno Readback` and `## Suno Raw Response` sections present in body
   - `## MiniMax Translation` appears BEFORE the Suno sections
   - `## Notes` appears AFTER the Suno sections

2. **Local `.mp3` file → full card.** Run with `--file path/to/sample.mp3 --title "Test Track" --type single-track --slug test-mp3`. Expect:
   - `source: file` in frontmatter
   - `source_file` present and points to the copied audio under `vault/studio/references/shared/music-cards/audio/`
   - No `channel`, no `source_url`, no `sample`
   - `suno_*` fields present
   - `## Suno Readback` and `## Suno Raw Response` sections present
   - Audio file actually exists at the `source_file` path

3. **Local `.wav` file → full card.** Same as test 2 but with a `.wav` file. Verify:
   - `source_file` ends in `.wav`
   - `suno-api` step1 payload was `{ extension: "wav" }` — check via debug log or test harness
   - Readback still succeeds

4. **Local file with `--track "[[some-existing-track]]"`** (where `some-existing-track.md` exists under `vault/studio/tracks/`) → `track: "[[some-existing-track]]"` appears in frontmatter.

5. **Local file with `--track path/to/nonexistent.md`** → hard error BEFORE any file copy or Suno upload. Error message mentions the missing path and `vault/studio/tracks/`.

6. **`SUNO_AUTHORIZATION` unset** → card generates successfully without any `suno_*` fields and without Suno sections in the body. Tool output contains the line `Suno readback skipped — ...`. Exit code 0.

7. **`--no-suno` flag** → same as test 6 (no suno fields, no sections) but NO "skipped" warning line. Exit code 0.

8. **Both `--file` and a URL passed** → hard error about mutual exclusivity. No file copy, no API call, no card written.

9. **`--file foo.flac`** → hard error about unsupported format. No file copy, no API call.

10. **Existing `bun test` suite still green** for `suno-meta.test.ts` — all mp3 tests continue to pass, new wav tests pass. (See file 2 note about Windows bun test segfault — if that blocks locally, document the workaround.)

11. **`refine.ts` is byte-identical** to before this task (quick `git diff` check). If any bytes changed in that file, the task is rejected.

## Related Context

- FAI-0002 task doc: `vault/notes/tasks/2026-04-09-190554-FAI-0002-suno-extract-meta.md`
- FAI-0002 memory (gotchas): `C:\Users\odiem\.claude-personal\projects\d--PROJECTS-freddie-ai\memory\project_suno_meta.md`
  - Trailing slash asymmetry between endpoints
  - Authorization header format
  - Browser fingerprint headers required
  - 5-second wait before initialize-clip
  - Token expiry = 401 = exitCode 2
- `src/libs/suno-api/index.ts:72-84` — `SunoExtractResult` interface
- `src/libs/suno-api/index.ts:158-188` — current `validateInputs` (mp3-only gate)
- `src/libs/suno-api/index.ts:209-244` — `step1CreateUploadSession` (hardcoded `{ extension: "mp3" }` on line 213)
- `src/libs/suno-api/index.ts:248-283` — step2 S3 upload (format-agnostic, do not touch)
- `src/libs/suno-api/index.ts:287-303` — step3 upload-finish (format-agnostic, do not touch)
- `src/tools/suno-meta.ts` — CLI entry, exit code conventions
- `src/tools/musiccard/index.ts` — current orchestrator
- `src/tools/musiccard/extract.ts` — yt-dlp + ffmpeg wrapper (add local file + ffprobe helpers here)
- `src/tools/musiccard/card.ts` — card renderer (read to understand current section ordering)
- `src/tools/musiccard/refine.ts` — Gemini refinement — **DO NOT MODIFY**
- `.claude/skills/create-music-card/SKILL.md` + `workflow.md` — the skill that calls musiccard
- Existing cards in `vault/studio/references/shared/music-cards/` — legacy frontmatter shape reference; being wiped, so do not preserve compatibility

## Notes for Ryan

- Read every file listed in "File-by-File Spec" before touching any of them.
- Landmines from FAI-0002 are documented in the memory file above. If the Suno step mysteriously stops working during your testing, check tokens first (exitCode 2) before assuming you broke something in `suno-api`.
- The `refine.ts` prohibition is not me being precious — it is the whole reason this corpus has any research value. If you find a reason you think it MUST change, post a `[TECH BLOCKER]` and I will review before any edit lands.
- Test order: file 1 (suno-api) → file 2 (tests for file 1) → file 4 (extract.ts helpers) → file 5 (card.ts renderer changes) → file 3 (musiccard index.ts orchestration) → files 7+8 (skill docs). This order lets you verify each layer independently.
- Use the tool runner for any command invocation. Relative paths only. No `cd &&`.

---

## Amendment 1 — Librosa Raw Data + Comparison (2026-04-09)

### FAI-0003 AMENDMENT — Librosa Raw Data + Comparison Sections

FAI-0003 shipped in commits `3ac2c21` (initial) and `a147b8c` (Suno/Gemini contamination fix). Cards now include Suno readback and raw JSON. First real test on a Dwarven Songs YouTube video confirmed drift integrity — Gemini's output was not contaminated by Suno data.

One drift finding surfaced: librosa measured **95.7 BPM**, Suno heard **72 BPM** (half-time feel). This is exactly the kind of data the corpus is meant to capture, but right now it's invisible — you have to read the Suno style prose and compare against the Profile table manually. This amendment makes drift visible at a glance by adding two new card sections.

### Goals

1. **`## Librosa Raw Data`** — structured table + collapsible full `AudioAnalysis` JSON. Mirror of the existing `## Suno Raw Response` section.
2. **`## Comparison`** — side-by-side librosa vs Suno table covering BPM, Key, Duration, Has vocal, Display tags.

### Architecture Decisions (locked)

#### Decision A1 — Placement: grouped raw data at the bottom

Card body order after amendment:

```
## Audio
## Profile
## Groove
## Palette
## Suno Translation
## MiniMax Translation
## Comparison          ← NEW (human-readable drift summary)
## Suno Readback       ← existing (unchanged position)
## Librosa Raw Data    ← NEW (collapsible JSON + table)
## Suno Raw Response   ← existing (unchanged position)
## Notes
```

Rationale: keeps narrative sections (Profile through Translations) at the top, groups all machine-raw output at the bottom near the existing Suno Raw Response. Comparison goes right before Suno Readback so the reader sees the drift summary before the detailed Suno artifacts.

#### Decision A2 — Suno BPM/Key extraction via dedicated Gemini structured call

Suno's BPM and Key are embedded in `style_prose` as free text (e.g., "Epic orchestral choral piece in D minor at 72 BPM"). To build a structured Comparison table, we need to extract these into fields.

**Approach:** A dedicated Gemini call that reads Suno's `style_prose` (and ONLY the style_prose, no other card data) and returns a minimal JSON object:

```ts
export interface SunoStructured {
  bpm: number | null;          // numeric BPM if present, null if not extractable
  key: string | null;          // e.g., "D minor", "C# major", null if not extractable
  tempo_feel: string | null;   // e.g., "half-time", "marching", "processional" — free text hint, optional
}
```

**Drift integrity:** This Gemini call only sees `style_prose`. It does NOT see librosa data, the card content, the user's genre/mood hints, or anything else that could contaminate its extraction. Its sole job is to parse one prose paragraph into three fields. The `refine.ts` contamination rule from the original task still holds — this is a separate Gemini call with a separate prompt, not an enrichment of refine.

**Cost:** ~300 input tokens, ~50 output tokens per card. Negligible.

**Timing:** Runs AFTER the main Gemini refinement and AFTER the Suno readback, before the Comparison section is rendered. If the extraction call fails (timeout, bad response, no JSON), fall back to `{ bpm: null, key: null, tempo_feel: null }` and the Comparison table shows "—" for those fields. **Never fails the card.**

**New file:** `src/tools/musiccard/suno-extract.ts`
- Export `extractSunoStructured(styleProse: string): Promise<SunoStructured>`
- Define and export the `SunoStructured` interface
- Implementation: `generateText(prompt, { model: "gemini-2.5-flash", temperature: 0.1, timeout: 30_000, caller: "musiccard-suno-extract" })` via the existing helper from `src/libs/gemini.ts`
- Parse response, return null-filled default on any failure
- **Do NOT throw** — callers should always get a default object

**Prompt template** (keep terse to minimize token cost):

```
Extract BPM, key, and tempo feel from this music description. Respond with ONLY a JSON object, no prose, no code fences.

Description:
<style_prose>

Required JSON shape:
{"bpm": <number or null>, "key": "<string or null>", "tempo_feel": "<string or null>"}

Rules:
- bpm: integer if clearly stated, null otherwise
- key: format like "C major", "D minor", "F# minor" — null if not stated
- tempo_feel: short phrase like "half-time", "marching", "driving" if clearly implied by the description, null otherwise
- Output must be valid JSON parseable by JSON.parse
```

Error handling: wrap `JSON.parse` in try/catch. On any failure → return the null-filled default.

#### Decision A3 — Librosa Raw Data section shape

**Structured table:**

```markdown
## Librosa Raw Data

| Metric     | Value                                 |
|------------|---------------------------------------|
| BPM        | 95.7                                  |
| Key        | D minor (confidence: 0.698)           |
| Energy     | medium (RMS: 0.142, dynamics: 0.089)  |
| Brightness | dark (centroid: 1420 Hz)              |
| Width      | moderate                              |
| Rhythm     | 3.2 onsets/sec (moderate density)     |
```

Values come directly from the `AudioAnalysis` object returned by `analyzeAudio()` in `src/tools/musiccard/analyze.ts`. Read that file to confirm the exact shape before implementing the renderer.

**Collapsible full JSON** (use HTML details block — Obsidian renders it):

```markdown
<details>
<summary>Full librosa AudioAnalysis JSON (click to expand)</summary>

`` `json
{ ...full AudioAnalysis serialized with 2-space indent... }
` ``

</details>
```

(Fence shown with a space inside to avoid nesting in this doc — the real renderer emits a normal triple-backtick `json` fence.)

**Gating:** If `audioAnalysis` is null (e.g., `--no-analyze` or librosa failed), **skip this section entirely**. Never emit with placeholder "—".

#### Decision A4 — Comparison section shape

```markdown
## Comparison

| Metric       | Librosa       | Suno                                                | Notes                  |
|--------------|---------------|-----------------------------------------------------|------------------------|
| BPM          | 95.7          | 72                                                  | Δ 23.7 (half-time)     |
| Key          | D minor       | D minor                                             | match                  |
| Duration     | 219s          | 219s                                                | match                  |
| Has vocal    | —             | yes                                                 | librosa doesn't compute|
| Display tags | —             | cinematic choral, dark fantasy, male vocal ensemble | —                      |
```

**Rendering rules:**

- **BPM row:** librosa BPM from `audioAnalysis.bpm`. Suno BPM from `SunoStructured.bpm`. If both present, compute `Δ <delta>` in notes column; if `suno.tempo_feel` present, append the feel label, e.g. `Δ 23.7 (half-time)`. If Suno BPM null, show "—" in Suno column and "see Style prose" in Notes.
- **Key row:** librosa key from `audioAnalysis.key.label`. Suno key from `SunoStructured.key`. If both present and equal (case-insensitive normalized), "match"; else show "differs". If Suno key null, "—" in Suno column.
- **Duration row:** librosa from `audioAnalysis` if available, otherwise from the trim duration. Suno from `sunoResult.duration_seconds`. If equal (±1s tolerance), "match".
- **Has vocal row:** librosa column always "—" (we don't compute it). Suno column shows yes/no from `sunoResult.has_vocal`.
- **Display tags row:** librosa column always "—". Suno column from `sunoResult.display_tags`.

**Gating:**

- **Skip Comparison section entirely if `sunoResult` is null** (same gating as Suno Readback). If librosa succeeded but Suno failed, there's nothing to compare.
- If `audioAnalysis` is null but `sunoResult` is present, still show the Suno-only rows with "—" in the Librosa column. (Rare — librosa would have had to fail while Suno succeeded.)
- If both are null, no section.

### File Changes

#### 1. NEW: `src/tools/musiccard/suno-extract.ts`

- Export `extractSunoStructured(styleProse: string): Promise<SunoStructured>`
- Define and export the `SunoStructured` interface
- Uses `generateText` from `src/libs/gemini.ts`, model `gemini-2.5-flash`, temperature 0.1, 30s timeout, caller `musiccard-suno-extract`
- Null-filled default on any failure, never throws

#### 2. `src/tools/musiccard/card.ts` (modify)

- Import `AudioAnalysis` type and `SunoStructured` type
- Add helpers:
  - `renderComparison(audioAnalysis, sunoResult, sunoStructured): string` — returns the `## Comparison` markdown block. Empty string if `sunoResult` is null.
  - `renderLibrosaRawData(audioAnalysis): string` — returns the `## Librosa Raw Data` markdown block with table + collapsible JSON. Empty string if `audioAnalysis` is null.
- **Splice strategy:** Recommended — one splice helper `appendRawDataAndComparison(cardContent, audioAnalysis, sunoResult, sunoStructured): string` that handles all post-Gemini sections in order. Alternatively extend the existing `appendSunoToCard` helper. Your call on shape — the hard constraint is that all these sections get spliced **AFTER** Gemini refinement, never before.
- Section ordering in the splice:
  - `## Comparison` inserts between `## MiniMax Translation` and `## Suno Readback`
  - `## Librosa Raw Data` inserts between `## Suno Readback` and `## Suno Raw Response`

#### 3. `src/tools/musiccard/index.ts` (modify)

- After `refineWithGemini` returns (Phase E), add Phase F: structured Suno extraction:

```ts
let sunoStructured: SunoStructured = { bpm: null, key: null, tempo_feel: null };
if (sunoResult) {
  out.push("Extracting structured fields from Suno prose...");
  sunoStructured = await extractSunoStructured(sunoResult.style_prose);
  out.push(`  Suno structured: bpm=${sunoStructured.bpm ?? "—"}, key=${sunoStructured.key ?? "—"}, feel=${sunoStructured.tempo_feel ?? "—"}`);
}
```

- Pass `sunoStructured` and `audioAnalysis` to the splice helper alongside `sunoResult`
- Import `extractSunoStructured` and `SunoStructured` from `./suno-extract.js`

#### 4. `src/tools/musiccard/refine.ts` — **DO NOT MODIFY**

Still byte-identical. The original non-negotiable ground rule from this task still applies: Suno data must never feed Gemini refinement. The new `suno-extract.ts` is a separate Gemini call with a separate prompt; it is NOT an extension of `refine.ts`. Reviewer will check `refine.ts` with `git diff` and fail the review if it changed.

#### 5. `.claude/skills/create-music-card/SKILL.md` (optional, small doc update)

- Update the Output description to mention the new Comparison and Librosa Raw Data sections

#### 6. `.claude/skills/create-music-card/workflow.md` (optional)

- P3-001 presentation template can mention the Comparison section exists for the user to spot drift at a glance

### Scope Exclusions

- **No regex BPM extraction fallback.** If Gemini extraction fails, fields are null. Keep it simple.
- **No historical backfill.** User is clean-slating.
- **No automated drift metric computation** beyond the Comparison table. The human reads the table.
- **No changes to `refine.ts`.** Still load-bearing rule.
- **No new CLI flags.** This is purely additive rendering on top of the existing pipeline.

### Acceptance Tests (additive to the original 11)

12. **Happy path.** Run on a YouTube URL with valid Suno tokens. Verify:
    - `## Comparison` section appears between `## MiniMax Translation` and `## Suno Readback`
    - BPM row shows both librosa and Suno values with a delta in Notes if they differ
    - `## Librosa Raw Data` section appears between `## Suno Readback` and `## Suno Raw Response`
    - Librosa Raw Data has BOTH the structured table AND the collapsible JSON block

13. **Suno-off path.** Run with `--no-suno`. Verify:
    - `## Comparison` section is ABSENT (no Suno data to compare against)
    - `## Librosa Raw Data` IS present (librosa data still exists)
    - Existing Suno Readback and Suno Raw Response sections remain absent (as before)

14. **Extraction failure path.** Mock or simulate `extractSunoStructured` failure (invalid JSON response / timeout / thrown error). Verify:
    - Card still builds successfully
    - Comparison table shows "—" for Suno BPM and Key columns
    - BPM row Notes column shows "see Style prose"
    - No exception propagates to the user

15. **Refine integrity check.** After implementation, run `git diff src/tools/musiccard/refine.ts` — expect empty diff. If anything changed in that file, the amendment is rejected.

### Notes for Ryan (amendment)

- The `refine.ts` prohibition still stands. `suno-extract.ts` is a **separate** Gemini call for a **separate** purpose (field extraction, not refinement). Do not be tempted to merge them.
- The 95.7 vs 72 BPM finding is the motivating example — test against that Dwarven Songs video if you still have the URL, and confirm the Comparison row reads `Δ 23.7 (half-time)` or equivalent.
- `generateText` signature: confirm the option names (`model`, `temperature`, `timeout`, `caller`) match the current `src/libs/gemini.ts` before writing `suno-extract.ts`. If any name has drifted, adapt.
- Splice helper shape is your call — one helper or two — but the constraint is post-Gemini only, and `refine.ts` stays byte-identical.
