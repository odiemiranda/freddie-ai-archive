---
project-code: FAI
task-id: FAI-0004
status: todo
agent: wick
created: 2026-04-10
title: musiccard-mine analysis CLI
---

# FAI-0004 — musiccard-mine analysis CLI

## Summary

Build a local, deterministic CLI tool `bun run tool musiccard-mine` with four analysis modes that mine the music-cards corpus (`vault/studio/references/shared/music-cards/*.md`) to reverse-engineer Suno's native vocabulary, bracket conventions, and vocabulary drift vs Tala's reference Style blocks. Pure text processing — no Gemini calls, no brain.db coupling, no writes back to vault references. Output is markdown tables (default) or JSON (`--json`), to stdout or `--out <path>`.

## Context

FAI-0002 (Suno upload-and-extract) and FAI-0003 (music-card integration) shipped the drift-study corpus infrastructure. Music cards now carry Suno's own `style_prose`, `lyrics` (typed-bracket), `display_tags`, and raw JSON alongside librosa data and (optionally) a `track:` wikilink back to the source Tala track.

The real goal, clarified by the user: **improve the system's ability to write Suno Style blocks and lyrics that match how Suno actually speaks.** We want to reverse-engineer Suno's native vocabulary from its own outputs, then update references so Tala/Rune write prompts in Suno's own tongue. This applies to BOTH BGM (instrumental) and non-BGM (vocal) tracks.

The user is about to process ~30 tracks (15 BGM + 15 non-BGM) as Phase 1. Before they batch-process, they need a mining CLI that can actually extract patterns from the corpus — otherwise they'll generate cards they never read. This tool is that reader.

Why no Gemini:
1. Deterministic and reproducible
2. Zero token cost — the user re-runs after every update
3. Patterns we're mining are surface-level lexical, not semantic
4. Fast iteration (<5s on 100 cards)

If later phases want semantic clustering, that's FAI-0005+, not this tool.

## Architecture Decisions

### 1. Input source — files on disk, not brain.db

Scan `vault/studio/references/shared/music-cards/*.md` directly. Parse frontmatter for metadata, then parse the **Suno Raw Response** collapsible JSON block at the bottom of each card. That block is the cleanest structured source — less fragile than scraping the rendered `## Suno Readback` section.

Extract per card:
- `suno_style_prose` — the full prose paragraph
- `suno_lyrics` — the typed-bracket lyrics string
- `suno_display_tags` — comma-separated tag list
- `suno_has_vocal` — boolean (for BGM vs non-BGM split)
- `track` — frontmatter field, wikilink to source Tala track (optional)
- `title`, `source`, `source_url`, `tags` — frontmatter

**Rationale:** brain.db is a read-optimized index, not the primary source. Cards are already sitting on disk in a stable format. File-based parsing is simpler, doesn't require brain.db to be synced, and doesn't couple the tool to the brain.db schema.

**Graceful degradation:** if a card lacks a Suno Raw Response block (readback was skipped, older card, etc.), log a warning to stderr and skip that card. Continue with the rest. Never throw.

### 2. Four analysis modes

All modes scan all cards by default. Shared flags:

- `--filter vocal|bgm|all` (default `all`) — splits by `suno_has_vocal`
- `--since YYYY-MM-DD` — scans only cards created after the given date (from frontmatter `created` or file mtime fallback)

#### Mode A: `--suno-vocabulary`

Aggregate every unique word from every card's `suno_style_prose`.
- Lowercase, strip punctuation, filter standard English stopwords, filter single-char tokens.
- Output: frequency-ranked markdown tables, one per category.
- Categorization (heuristic, via lexicons):
  - **instruments** — match against INSTRUMENT_LEXICON
  - **production** — reverb, compression, mix, mastering, EQ, panning, stereo, mono, lo-fi, warm, dry, wet, etc.
  - **mood** — epic, melancholic, tense, serene, etc.
  - **tempo/energy** — driving, marching, slow, fast, frenetic, sparse, etc.
  - **other** — everything else
- Table columns: `| Word | Count | Cards |` (Cards = distinct cards that used the word)
- Flags:
  - `--top N` — limits each table to top N (default 50)
  - `--min-count N` — skips words with fewer than N occurrences (default 2)

#### Mode B: `--suno-brackets`

Extract every `[...]` tag from every card's `suno_lyrics` via regex `/\[([^\]]+)\]/g`. Classify each tag:

- **Section tags:** match the SECTION_TAGS regex case-insensitively (`^(intro|outro|verse\s*\d*|pre-chorus|chorus|bridge|hook|refrain|drop|build|breakdown|interlude|instrumental\s*break|instrumental)$`)
- **Direction tags:** contain any of — a known instrument name from INSTRUMENT_LEXICON, a vocal descriptor (`soft`, `loud`, `whispered`, `shouted`, `belted`, `chanted`, `hummed`, `male`, `female`, `choir`, `solo`), a verb form (`-s`, `-ing`, `-ed` word ending), or dynamic descriptors (`building`, `fading`, `sparse`, `full`)
- **Other:** everything that doesn't match above

Output: three frequency-ranked tables (sections, directions, other).
Columns: `| Tag | Count | Cards | Example (first occurrence card) |`

Summary footer:
> X unique section tags, Y unique direction tags, Z other. Most common section: `[Chorus]` (28 cards). Most common direction: `[full choir enters]` (4 cards).

When `--filter all` (default), split output into a BGM section and a vocal section — bracket conventions differ between them.

#### Mode C: `--compare-vocabulary`

For each card that has a `track: "[[wikilink]]"` frontmatter field:
1. Resolve the wikilink to a file under `vault/studio/tracks/**/*.md` (search by basename without `.md`)
2. Read the track file. Extract the `### Style` section content (fenced code block after the `### Style` heading — content between opening ` ``` ` after the heading and the next closing ` ``` `)
3. Tokenize both Tala's Style block and Suno's `style_prose` into lowercase word sets (stopwords filtered)
4. Compute three sets: **intersection** (both used), **tala-only**, **suno-only**

Output: per-card report:

```
## <card title> → <track title>

| Category | Words |
|----------|-------|
| Both | celtic, folk, baritone, harp, ... |
| Tala only | bodhran, gravelly, measured, ... |
| Suno only | orchestral, cinematic, processional, ... |
```

Aggregate summary at the end:
> Across N cards: X% average vocabulary overlap.
> Top Tala-only words (we use, Suno doesn't): ...
> Top Suno-only words (Suno's preferred vocabulary we're not using): ...

Skip cards without a `track` field with a brief stderr notice. Flag:
- `--min-overlap N` — filters out low-overlap cards that aren't useful

#### Mode D: `--instrument-drift`

For each instrument in INSTRUMENT_LEXICON, count:
- How many Tala Style blocks mention it (across all linked tracks resolved via the `track:` field)
- How many Suno style_prose paragraphs mention it (across all cards)
- For each card where Tala mentions it but Suno doesn't: grab the nearest noun phrase in Suno's prose as the heuristic "drift target"

Output table:

```
| Instrument | Tala uses | Suno uses | Tala→Suno drift |
|------------|-----------|-----------|-----------------|
| bodhran | 8 | 0 | "frame drum" (4), "tribal drum" (2), "hand drum" (1), (none) (1) |
| shamisen | 12 | 3 | "plucked string" (5), "lute" (2), "nylon guitar" (1), (present) (3) |
```

Flag:
- `--instruments <comma-list>` — overrides the canonical list with a custom set

### 3. Output format

- **Default:** markdown to stdout
- `--out <path>` — writes to file, stdout only shows a short progress line
- `--json` — structured JSON instead of markdown, valid `JSON.parse()` output
- `--out report.json --json` is valid; `--out report.md --json` is valid (writes JSON to `.md` — no validation of extension)

### 4. Tool file location

Registry convention (auto-discovered by the tool runner):

- `src/tools/musiccard-mine/index.ts` — CLI entry, exports `toolDef`, argument parsing, mode dispatch
- `src/tools/musiccard-mine/parse-card.ts` — shared card file parser
- `src/tools/musiccard-mine/lexicons.ts` — shared stopword/classifier constants
- `src/tools/musiccard-mine/vocabulary.ts` — Mode A
- `src/tools/musiccard-mine/brackets.ts` — Mode B
- `src/tools/musiccard-mine/compare.ts` — Mode C
- `src/tools/musiccard-mine/instruments.ts` — Mode D

### 5. No Gemini calls

Pure local text processing. No API calls, no LLM classification, no external dependencies beyond file I/O and markdown parsing. Runs in under 5 seconds on a corpus of 100 cards.

### 6. Stopword handling

Standard English stopwords:
`the, a, an, and, or, but, of, in, on, at, with, by, for, to, from, is, are, was, were, be, been, being, have, has, had, do, does, did, will, would, could, should, may, might, can, this, that, these, those, it, its, there, their, they, them, as, if, than, then, when, where, which, who, whom, whose, what, how, why`

Plus music-specific stopwords (too generic to be useful):
`music, track, piece, song, sound, sounds, featuring, features, featured, etc`

Export as `STOPWORDS` in `lexicons.ts` so Ryan (or McCall in review) can tune it later.

### 7. Bracket classification heuristics

Order of operations in `brackets.ts`:

1. Normalize: lowercase, trim whitespace
2. Check against SECTION_TAGS regex (case-insensitive). Accept `verse 1` or `verse1`.
3. If not a section: check for direction markers — presence of a known instrument name, a vocal descriptor, a verb form (`-s`, `-ing`, `-ed`), or dynamic descriptors (`soft`, `loud`, `whispered`, `building`, `fading`).
4. If neither: "other" bucket.

Heuristic and imperfect by design. The tables are meant to be reviewable by eye — miscategorization is tolerable, completeness matters.

## File-by-File Spec

### 1. `src/tools/musiccard-mine/index.ts`

CLI entry and registry export.

```ts
export const toolDef = {
  name: "musiccard-mine",
  description: "Mine the music-cards corpus for Suno vocabulary, bracket conventions, and drift vs Tala references",
  // ... standard toolDef shape (match existing tools)
};

type Mode = "vocabulary" | "brackets" | "compare" | "instruments";
type Filter = "vocal" | "bgm" | "all";

interface CliOptions {
  mode: Mode | null;
  filter: Filter;
  since: string | null;
  top: number;
  minCount: number;
  minOverlap: number | null;
  instruments: string[] | null;
  out: string | null;
  json: boolean;
  help: boolean;
}

async function main(argv: string[]): Promise<number> { /* ... */ }
```

Responsibilities:
- Parse argv. Map `--suno-vocabulary`, `--suno-brackets`, `--compare-vocabulary`, `--instrument-drift` to the `mode` field. Only one mode at a time — if none or multiple, print help and exit 0.
- Load all cards via `parseAllCards()` from `parse-card.ts`.
- Apply `--filter` and `--since` globally before dispatching.
- Dispatch to the selected mode handler.
- Emit via stdout or `--out`. If `--json`, stringify with `JSON.stringify(result, null, 2)`.
- Exit 0 on success. Exit 0 on empty corpus with a helpful "no cards found" message. Exit 1 only on unexpected exceptions.

### 2. `src/tools/musiccard-mine/parse-card.ts`

```ts
export interface ParsedCard {
  path: string;
  title: string;
  source: string | null;
  sourceUrl: string | null;
  track: string | null;           // wikilink target basename (no .md, no brackets)
  createdDate: string | null;     // YYYY-MM-DD if present
  tags: string[];
  sunoStyleProse: string;
  sunoLyrics: string;
  sunoDisplayTags: string[];
  sunoHasVocal: boolean;
}

export function parseCardFile(path: string): ParsedCard | null;
export function parseAllCards(dir: string): ParsedCard[];
```

Responsibilities:
- Read the file. Parse YAML frontmatter (use existing project helper if available; otherwise a minimal parser is fine — frontmatter is controlled).
- Extract the `<details>...<summary>Suno Raw Response</summary>` block and its fenced JSON. If missing or malformed, log a warning to stderr and return `null`.
- From the raw JSON, extract `style_prose` (or `prompt`/`style` depending on the shape — check a real card), `lyrics`, `display_tags`, and `has_vocal` (or infer from presence of lyrics).
- Resolve `track` wikilink: strip `[[` and `]]`, take the final segment after `|` if an alias is present, drop any `.md`.
- `parseAllCards` uses `Bun.glob` or `fs.readdirSync` on the cards directory, calls `parseCardFile` on each, filters nulls.

### 3. `src/tools/musiccard-mine/lexicons.ts`

```ts
export const STOPWORDS: ReadonlySet<string> = new Set([ /* ... */ ]);

export const INSTRUMENT_LEXICON: readonly string[] = [
  "bodhran", "celtic harp", "shamisen", "taiko", "frame drum", "bass drum",
  "cello", "violin", "viola", "flute", "low whistle", "penny whistle",
  "uilleann pipes", "saxophone", "trumpet", "trombone", "tuba",
  "piano", "rhodes", "wurlitzer", "organ", "synth",
  "upright bass", "double bass", "electric guitar", "acoustic guitar", "nylon guitar",
  "banjo", "mandolin", "ukulele",
  "drum kit", "snare", "kick", "hi-hat", "ride", "crash",
  "hand drum", "tambourine", "shaker", "rainstick", "bells", "chimes",
  "choir", "choral", "baritone", "tenor", "alto", "soprano", "mezzo",
  "falsetto", "whistle register", "spoken word",
];

export const SECTION_TAGS_REGEX: RegExp =
  /^(intro|outro|verse\s*\d*|pre-?chorus|chorus|bridge|hook|refrain|drop|build|breakdown|interlude|instrumental\s*break|instrumental)$/i;

export const VOCAL_DESCRIPTORS: readonly string[] = [
  "soft", "loud", "whispered", "shouted", "belted", "chanted", "hummed",
  "male", "female", "choir", "solo",
];

export const PRODUCTION_WORDS: readonly string[] = [
  "reverb", "compression", "mix", "mastering", "eq", "panning", "stereo", "mono",
  "lo-fi", "warm", "dry", "wet", "saturated", "crisp", "muddy", "tight", "loose",
];

export const MOOD_WORDS: readonly string[] = [
  "epic", "melancholic", "tense", "serene", "bright", "dark", "hopeful",
  "menacing", "playful", "somber", "uplifting", "nostalgic",
];

export const TEMPO_WORDS: readonly string[] = [
  "driving", "marching", "slow", "fast", "frenetic", "sparse", "pulsing",
  "steady", "relentless", "lazy", "urgent",
];
```

Responsibilities:
- Pure data module. No functions. Exports are `const` arrays/sets/regexes.
- Include a brief header comment noting that these are tunable — Ryan and McCall can add entries.

### 4. `src/tools/musiccard-mine/vocabulary.ts`

```ts
export interface VocabularyReport {
  totalCards: number;
  filter: "vocal" | "bgm" | "all";
  categories: {
    instruments: WordRow[];
    production: WordRow[];
    mood: WordRow[];
    tempoEnergy: WordRow[];
    other: WordRow[];
  };
}

export interface WordRow {
  word: string;
  count: number;
  cards: number;
}

export interface VocabularyOpts {
  top: number;
  minCount: number;
  filter: "vocal" | "bgm" | "all";
}

export function runVocabularyMode(cards: ParsedCard[], opts: VocabularyOpts): VocabularyReport;
export function renderVocabularyMarkdown(report: VocabularyReport): string;
```

Responsibilities:
- Tokenize `suno_style_prose` (lowercase, strip punctuation except hyphens inside words, split on whitespace).
- Drop single-char tokens and STOPWORDS.
- Build count + distinct-card maps.
- Categorize using lexicons. A word belongs to the first matching category (instruments → production → mood → tempoEnergy → other).
- Apply `top` and `minCount`.
- Return both the structured report and a markdown renderer. `index.ts` chooses which to emit based on `--json`.

### 5. `src/tools/musiccard-mine/brackets.ts`

```ts
export interface BracketsReport {
  totalCards: number;
  filter: "vocal" | "bgm" | "all";
  bgm?: BracketBuckets;
  vocal?: BracketBuckets;
  combined?: BracketBuckets;
  summary: string;
}

export interface BracketBuckets {
  sections: BracketRow[];
  directions: BracketRow[];
  other: BracketRow[];
}

export interface BracketRow {
  tag: string;
  count: number;
  cards: number;
  exampleCardTitle: string;
}

export function runBracketsMode(cards: ParsedCard[], opts: BracketsOpts): BracketsReport;
export function renderBracketsMarkdown(report: BracketsReport): string;
export function classifyBracket(tag: string): "section" | "direction" | "other";
```

Responsibilities:
- Regex-extract `[...]` from `suno_lyrics`.
- Normalize: trim, lowercase for classification but preserve original casing for output.
- When `--filter all`, build both BGM and vocal bucket sets and render them in separate subsections.
- Summary footer with counts and top examples.

### 6. `src/tools/musiccard-mine/compare.ts`

```ts
export interface CompareReport {
  totalCards: number;
  cardsWithTrack: number;
  cardsCompared: number;
  perCard: CardComparison[];
  aggregate: {
    averageOverlapPct: number;
    topTalaOnly: WordRow[];
    topSunoOnly: WordRow[];
  };
}

export interface CardComparison {
  cardTitle: string;
  trackTitle: string;
  intersection: string[];
  talaOnly: string[];
  sunoOnly: string[];
  overlapPct: number;
}

export function runCompareMode(cards: ParsedCard[], opts: CompareOpts): CompareReport;
export function renderCompareMarkdown(report: CompareReport): string;
export function parseTalaStyleBlock(trackFilePath: string): string | null;
```

Responsibilities:
- Resolve each card's `track` field by searching `vault/studio/tracks/**/*.md` for a file whose basename matches. Cache track file lookups across a single run.
- `parseTalaStyleBlock` reads the track file and extracts the content between the `### Style` heading and the next ` ``` ` closing fence. Return null if no `### Style` section or no fenced block.
- Tokenize both texts with the same pipeline as `vocabulary.ts` (reuse a shared tokenizer — put it in `parse-card.ts` or a new `tokenize.ts` if cleaner).
- Compute intersection and differences.
- Apply `--min-overlap` filter.
- Aggregate: mean overlap, top-N Tala-only and Suno-only word rows.

### 7. `src/tools/musiccard-mine/instruments.ts`

```ts
export interface InstrumentsReport {
  totalCards: number;
  cardsAnalyzed: number;
  rows: InstrumentRow[];
}

export interface InstrumentRow {
  instrument: string;
  talaUses: number;
  sunoUses: number;
  driftTargets: { phrase: string; count: number }[];
}

export function runInstrumentsMode(cards: ParsedCard[], opts: InstrumentsOpts): InstrumentsReport;
export function renderInstrumentsMarkdown(report: InstrumentsReport): string;
```

Responsibilities:
- For each instrument in the canonical or `--instruments` override list:
  - Count Tala Style blocks containing it (case-insensitive substring match)
  - Count Suno `style_prose` paragraphs containing it
  - For each card where Tala mentions it but Suno doesn't, heuristically grab the nearest noun phrase in Suno's prose and count it as a drift target. "Nearest noun phrase" can be approximated by: scan the Suno prose for tokens in INSTRUMENT_LEXICON or a small set of fallback nouns (`drum`, `string`, `horn`, `voice`, `lead`, `pad`) and pick the first hit. If none, record `(none)`.
- Sort rows by `talaUses` descending.
- Render as a markdown table with a compact drift column (top 4 drift targets joined by commas).

## Acceptance Tests

1. With 5+ real music cards present, `bun run tool musiccard-mine --suno-vocabulary` outputs a markdown vocabulary table with category sub-tables, at least 20 words total, no stopwords present.
2. `bun run tool musiccard-mine --suno-brackets` outputs three tables (sections, directions, other). `[Intro]` and `[Verse 1]` (if present) are classified as sections. `[frame drum enters]` (if present) is classified as direction.
3. `bun run tool musiccard-mine --compare-vocabulary` outputs per-card intersection/Tala-only/Suno-only tables for cards with a `track` field. Cards without a `track` field are silently skipped with a brief stderr notice.
4. `bun run tool musiccard-mine --instrument-drift` outputs the canonical instrument table with Tala/Suno counts and drift targets.
5. Running any mode with `--filter vocal` and `--filter bgm` produces different results that correctly split cards by `suno_has_vocal`.
6. `--json` flag emits valid JSON parseable by `JSON.parse()`.
7. `--out report.md` writes to the given file. Stdout is empty or shows only a progress line.
8. Running on an empty cards directory exits 0 with a helpful "no cards found" message — no exception thrown.
9. Running on a card whose Suno Raw Response block is missing skips that card with a stderr warning; other cards still process.
10. `bun run tool musiccard-mine` with no mode flag prints usage help documenting all four modes and exits 0.
11. `bun run tool musiccard-mine --help` behaves identically to #10.

## Scope Exclusions

- No Gemini-assisted clustering (FAI-0005+ material)
- No semantic similarity — pure lexical matching
- No writes to reference files — user applies findings manually via McCall/Ryan loop
- No visualization (charts, graphs) — markdown tables only
- No brain.db integration — file-based only
- No comparison against Rune's lyrics files — only Tala's Style blocks via the card's `track:` frontmatter field
- No analysis of music-video or single-track card types beyond the BGM/vocal split

## Related Context

- **Music card location:** `vault/studio/references/shared/music-cards/*.md`
- **Card format reference:** see any recent card, e.g., `vault/studio/references/shared/music-cards/2026-04-09-235604-nights-in-the-mines-v3.md`. Frontmatter carries `title`, `source`, `source_url`, `track?`, `suno_*` fields. Body has `## Suno Readback` (rendered) and `## Suno Raw Response` (collapsible `<details>` block with a fenced JSON payload).
- **Tala track format:** see `vault/studio/tracks/2026-03-22/070039-celtic-tavern-ballad/before-the-light.md`. Frontmatter with title + tags. Body has `### Style` and `### Lyrics` as fenced code blocks (triple-backtick).
- **Extracting Tala's Style block:** use a simple regex or markdown parser. The `### Style` block's content is everything between the opening ` ``` ` immediately after the `### Style` heading and the next ` ``` ` closing fence.
- **Parent projects:** FAI-0002 (suno-extract-meta CLI), FAI-0003 (/create-music-card Suno integration)
- **Downstream consumers:** Tala (`/create-track`) and Rune (`/lyrics`) — they will consume the human-curated findings this tool surfaces, applied manually to `vault/studio/references/suno/*` via McCall/Ryan loops.

## Notes for Ryan

- Tool runner auto-discovers via `toolDef` export from `index.ts`. Match the shape of an existing multi-file tool (look at `src/tools/musiccard/` or `src/tools/brain.ts` for reference).
- Never use `cd <path> && ...`. Relative paths from project root. Bun + TypeScript only.
- If the card JSON shape is slightly different from what I described (e.g., `style_prose` vs `prompt` vs `style`), inspect a real card first and adapt `parse-card.ts`. Flag it as a `[TECH BLOCKER]` if the shape varies wildly across cards — that's a data-quality issue I need to see.
- The tokenizer is shared across vocabulary, compare, and instruments modes. Put it in one place and import it — don't copy-paste.
- Prefer small pure functions. Each mode's `run*` should take parsed cards in and return a typed report out. The markdown renderer is a separate function. This makes `--json` trivial and unit testing possible.
- No tests required for this task (bun test is broken on Windows per known-issues memory), but keep the code shaped so tests could be added later.
