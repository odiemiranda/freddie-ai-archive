---
title: "Songwriting and Lyrics Composition Improvement"
type: prd
project: songwriting-lyrics-improvement
status: approved
created: 2026-04-03
author: holmes
brief: "vault/studio/projects/songwriting-lyrics-improvement/BRIEF.md"
---

# Songwriting and Lyrics Composition Improvement

## Overview

This project builds a comprehensive lyrics analysis toolkit that replaces LLM judgment with algorithmic precision across 9 dimensions, upgrades Rune's craft vocabulary with techniques from 5 elite lyricist archetypes, and tightens the lyrics workflow with automated tool checkpoints. The current toolchain covers approximately 15% of what is algorithmically checkable (LY2 meter/stress, LY5 consonant texture, LY6 homographs). This project expands that to approximately 80%, closing the verification gap that causes vocal artifacts -- rushed lines, mis-stressed words, forced rhymes -- in Suno AI and MiniMax Music output. All analysis supports English, Tagalog, and Taglish (code-switched) lyrics.

**Hypothesis:** We believe building 9 deterministic analysis tools, integrating 5 craft layers, and wiring tool checkpoints into the lyrics workflow will measurably reduce vocal artifacts in generated tracks for the user, because the current ~15% deterministic coverage leaves 85% of verifiable lyric dimensions to LLM self-assessment, and research confirms LLMs are provably unreliable at syllable counting, stress detection, rhyme scoring, and section balance analysis.

## User Stories

### US-1: Accurate syllable counting across languages

**As a** user writing lyrics in English, Tagalog, or Taglish, **I want** a tool that returns exact syllable counts per word and per line, **so that** I can verify lines match genre density targets and avoid overstuffed/understuffed lines that cause the vocal engine to rush or stretch.

**Acceptance Criteria:**
- [ ] AC-1.1: `bun run tool lyrics-analyze syllable --text "Walking through the morning light"` returns per-word and per-line syllable counts
- [ ] AC-1.2: English words resolved via `phonemize`/`syllables` npm package (CMU-backed). Coverage: 90%+ of words in standard English lyrics return dictionary-confirmed counts
- [ ] AC-1.3: Words not in dictionary fall back to G2P estimation. Fallback words flagged with `[approx]` marker in output
- [ ] AC-1.4: Tagalog words resolved via vowel-group heuristic (Tagalog has 5 vowels, no silent letters, no ambiguous diphthongs). Accuracy: within +/-1 syllable for 90%+ of Tagalog lines
- [ ] AC-1.5: Taglish lines use language detection per word -- CMU-dict-present words processed as English, dict-miss words processed as Tagalog. Detection accuracy: 80%+ against manually labeled test set of 50+ lines
- [ ] AC-1.6: Contractions preserved as single tokens (never expanded). `"don't"` = 1 syllable, not `"do not"` = 2 syllables
- [ ] AC-1.7: Existing `syllable-heuristic.ts` replaced by this tool. No orphaned code
- [ ] AC-1.8: Both CLI (`bun run tool lyrics-analyze syllable`) and programmatic mode (registry `toolDef`) operational

**Priority:** Must Have

---

### US-2: Stress pattern mapping for rhythmic alignment

**As a** user, **I want** to see the stress pattern (strong/weak) of every line in my lyrics, **so that** I can identify rhythmic dead zones (3+ consecutive unstressed syllables) and ensure stressed syllables carry meaning words, not function words.

**Acceptance Criteria:**
- [ ] AC-2.1: Each line outputs an S/W (strong/weak) pattern string with metrical foot classification (iambic, trochaic, anapestic, dactylic, or mixed)
- [ ] AC-2.2: English stress derived from CMU stress markers (0/1/2). Primary stress (1) and secondary stress (2) both count as strong
- [ ] AC-2.3: Tagalog stress defaults to penultimate syllable (malumay rule). Known exceptions from an exception list override the default
- [ ] AC-2.4: Lines with 3+ consecutive unstressed syllables flagged as "rhythmic dead zone" with severity high (4+) or medium (3)
- [ ] AC-2.5: Lines where function words (articles, prepositions, pronouns) occupy positions that would map to strong beats flagged as advisory warnings
- [ ] AC-2.6: This tool BLOCKS at node 008 -- lyrics with high-severity stress flags must be revised before proceeding to critics

**Priority:** Must Have

---

### US-3: Rhyme detection and quality scoring

**As a** user, **I want** to know exactly where rhymes occur in my lyrics (end rhyme and internal rhyme), what type they are (perfect, slant, assonance, consonance, mosaic), and how strong they are, **so that** I can assess whether rhymes sound natural rather than forced, and catch rhyme tax (lines warped to fit a rhyme).

**Acceptance Criteria:**
- [ ] AC-3.1: Detects end rhymes (last stressed vowel + tail) and internal rhymes (mid-line phonetic matches) per stanza
- [ ] AC-3.2: Classifies each rhyme pair: perfect, slant, assonance, consonance, mosaic/multisyllabic, identity (same word)
- [ ] AC-3.3: Scores each rhyme pair 0.0-1.0 using weighted phoneme comparison: stressed vowel match (65%), tail consonant similarity (25%), prefix similarity (15%). Research-backed weighting from RHYME-CTRL
- [ ] AC-3.4: Identity rhymes (same word or homophone) flagged as "lazy rhyme" with severity medium
- [ ] AC-3.5: Rhyme pairs scoring below 0.3 flagged as "weak rhyme" with severity low
- [ ] AC-3.6: Internal rhyme density reported per stanza (count of internal rhyme pairs). Research shows internal rhymes doubled 1990s-2010s and correlate with critical acclaim
- [ ] AC-3.7: Functions on English lyrics. Tagalog end rhyme detection supported via vowel-group comparison (Tagalog rhyme is primarily vowel-based). Taglish analyzed per-word with appropriate method

**Priority:** Must Have

---

### US-4: Section balance analysis

**As a** user, **I want** to see structural metrics per section (word count, line count, TTR, syllable density), **so that** I can detect imbalanced verses, overlong choruses, or sections that break genre conventions.

**Acceptance Criteria:**
- [ ] AC-4.1: Parses section headers from bracket tags (`[Verse 1]`, `[Chorus]`, `[Bridge]`, etc.) including aliased names
- [ ] AC-4.2: Per section reports: word count, line count, unique word count, TTR (type-token ratio), average syllables per line, total syllables
- [ ] AC-4.3: Cross-section comparison flags sections that deviate >50% from the section-type average (e.g., Verse 2 is twice as long as Verse 1)
- [ ] AC-4.4: Genre targets from `rhythm-and-arcs.md` loaded from brain.db and compared against actual metrics. Out-of-range sections flagged
- [ ] AC-4.5: Chorus compression check: choruses exceeding 6 lines flagged (research: 2-4 lines optimal for Suno melodic consistency)
- [ ] AC-4.6: Language-agnostic -- works identically for English, Tagalog, and Taglish (operates on tokenized text, not phonetics)

**Priority:** Must Have

---

### US-5: Rhyme scheme identification

**As a** user, **I want** the rhyme scheme of each section identified and labeled (AABB, ABAB, ABCB, etc.), **so that** I can verify the scheme is intentional and consistent, and detect accidental breaks.

**Acceptance Criteria:**
- [ ] AC-5.1: Outputs alphabetic scheme notation per section (e.g., "Verse 1: ABAB", "Chorus: AABB")
- [ ] AC-5.2: Based on rhyme detector output (US-3). Two lines rhyme if end-rhyme score >= configurable threshold (default 0.5)
- [ ] AC-5.3: Reports consistency: all instances of a section type (e.g., all verses) compared for scheme consistency. Mismatches flagged
- [ ] AC-5.4: Supports flexible schemes (ABAA, ABAX where X = no rhyme) without treating them as errors. Advisory only
- [ ] AC-5.5: Free verse / no-scheme sections labeled "FREE" rather than flagged

**Priority:** Should Have

---

### US-6: Vowel/consonant density analysis

**As a** user, **I want** to see the phonetic texture profile of each section (ratio of open vowels to plosives to liquids to fricatives), **so that** I can match the sound texture to the section's energy and emotion -- open vowels for sustained emotional moments, percussive consonants for driving rhythmic sections.

**Acceptance Criteria:**
- [ ] AC-6.1: Per section, reports counts and ratios: open vowels, closed vowels, plosives (p/t/k/b/d/g), fricatives (f/v/s/z/sh/zh), nasals (m/n/ng), liquids (l/r), glides (w/y)
- [ ] AC-6.2: Absorbs and replaces the existing LY5 consonant texture check from `phonetics.ts`. All current harsh/soft cluster detection preserved as a subset of this richer analysis
- [ ] AC-6.3: Texture-energy alignment check: flags open-vowel-dominated lines in `[Energy: High]` sections (may lack impact) and plosive-dominated lines in `[Energy: Low]` sections (may feel harsh)
- [ ] AC-6.4: Tagalog: "ng" digraph (/N/) classified as nasal, not as separate n+g. This is the known R11 risk from the brief -- bounded fix
- [ ] AC-6.5: Reports as advisory -- texture is a stylistic choice. No blocking

**Priority:** Should Have

---

### US-7: Repetition analysis

**As a** user, **I want** to detect intentional repetition patterns (hooks, refrains, callbacks, anaphora) and unintentional repetition (overused words, phrases appearing too frequently), **so that** I can distinguish deliberate hook reinforcement from lazy writing.

**Acceptance Criteria:**
- [ ] AC-7.1: Identifies exact-match repeated phrases (2+ words) across and within sections
- [ ] AC-7.2: Identifies near-match phrases (Levenshtein similarity > 0.7) as potential callbacks/variations
- [ ] AC-7.3: Calculates repetition ratio per section: (repeated words / total words). Flags sections with ratio > 0.4 that are not choruses/hooks
- [ ] AC-7.4: Distinguishes structural repetition (chorus/hook by definition repeats) from within-section repetition
- [ ] AC-7.5: Identifies anaphora (repeated line-opening patterns) and labels as intentional technique
- [ ] AC-7.6: Advisory output -- repetition can be a deliberate craft choice (Rune's "repetition is a feature" instinct)

**Priority:** Should Have

---

### US-8: Singability composite score

**As a** user, **I want** a composite score (0-100) that predicts how well my lyrics will translate to natural-sounding vocals, **so that** I can catch singability issues before spending generation credits.

**Acceptance Criteria:**
- [ ] AC-8.1: Composite of: syllable line length (vs genre target), open vowel placement at line endings, breath point presence (line breaks, punctuation), consonant density, stress regularity
- [ ] AC-8.2: Sub-scores reported individually so the user can see which dimension drags the composite down
- [ ] AC-8.3: Weighting initially based on research coexistence probabilities: Dur-str (long notes + stressed syllables), Peak-str (pitch peaks + stress), Line-len (syllable = note alignment). Weights configurable
- [ ] AC-8.4: Ships as ADVISORY ONLY. Research assumption A4 flags this as high-risk -- weighting may not correlate with actual Suno vocal quality until empirically calibrated
- [ ] AC-8.5: Calibration mechanism: reflect workflow outcomes logged to brain.db with singability scores. Over time, correlation analysis adjusts weights
- [ ] AC-8.6: Advisory warning shown at node 013 (pre-presentation). User sees the score but is never blocked

**Priority:** Could Have

---

### US-9: Prosody alignment check

**As a** user, **I want** to see which syllables in my lyrics would naturally align with strong vs weak beats assuming a standard metrical grid, **so that** I can identify lines where meaning-heavy words might land on weak beats and vice versa.

**Acceptance Criteria:**
- [ ] AC-9.1: Maps each syllable to an alternating S/W metrical grid (configurable: iambic default, trochaic option)
- [ ] AC-9.2: Uses POS tagging to identify content words (nouns, verbs, adjectives) vs function words (articles, prepositions)
- [ ] AC-9.3: Flags constraint violations: content word on weak position, function word on strong position. Severity based on violation count per line
- [ ] AC-9.4: Ships as ADVISORY ONLY. Research assumption A5 flags this as high-risk -- prosody alignment without knowing the actual melody produces false positives
- [ ] AC-9.5: Configurable strictness threshold. Default: flag only lines with 2+ violations
- [ ] AC-9.6: Advisory warning shown at node 013. Never blocks

**Priority:** Could Have

---

### US-10: Unified analysis CLI and report

**As a** user or as the Rune agent, **I want** a single command that runs all available analysis tools and produces a unified report with per-tool scores and a composite summary, **so that** I get a complete picture without running 9 separate commands.

**Acceptance Criteria:**
- [ ] AC-10.1: `bun run tool lyrics-analyze --file <path>` runs all enabled tools (T1+T2+T3) against the lyrics file
- [ ] AC-10.2: `bun run tool lyrics-analyze --file <path> --tools syllable,stress,rhyme` runs only specified tools
- [ ] AC-10.3: Report format: per-tool section (tool name, pass/warn/fail status, key findings, flag list), then composite summary
- [ ] AC-10.4: JSON output via `--json` flag for programmatic consumption by critics and workflow automation
- [ ] AC-10.5: Per-tool severity rollup: any tool with high-severity flags = overall status "FAIL". Medium = "WARN". All low or none = "PASS"
- [ ] AC-10.6: Report includes language detection summary: per-word language assignment and confidence. Useful for Taglish debugging
- [ ] AC-10.7: Tool registered via `toolDef` in registry. Both CLI and programmatic execution paths functional
- [ ] AC-10.8: Execution time reported. Total should stay under 500ms for a typical song (assumption A7)

**Priority:** Must Have

---

### US-11: Rune's self-critic loop uses tool-validated data

**As** the Rune agent, **I want** the self-critic pass (node 008) to run analysis tools and receive structured output, **so that** my self-assessment is grounded in algorithmic data rather than my own judgment on tasks where LLMs fail (syllable counting, stress detection, rhyme scoring).

**Acceptance Criteria:**
- [ ] AC-11.1: Node 008 runs `bun run tool lyrics-analyze --file out/rune-draft.md --json` and receives structured output
- [ ] AC-11.2: Syllable counter and stress mapper results checked against genre density targets. High-severity flags trigger mandatory rewrite before proceeding
- [ ] AC-11.3: Rhyme and balance results included as context for self-critic reasoning but do not block
- [ ] AC-11.4: Self-critic loop: if Rune rewrites to fix flags, tools re-run on the revised draft. Maximum 2 tool-check iterations before proceeding (prevents infinite loops)
- [ ] AC-11.5: Tool output summary included in the draft context passed to tri-critics at nodes 009-011
- [ ] AC-11.6: Existing phonetic check instructions in `workflow.md` node 008 replaced with the new unified tool call

**Priority:** Must Have

---

### US-12: Structural critic consumes tool output

**As** the Gemini structural critic (node 009), **I want** to receive the lyrics analysis report as structured data alongside the draft, **so that** I can reference deterministic tool findings instead of running my own unreliable phonetic checks.

**Acceptance Criteria:**
- [ ] AC-12.1: Critic input includes the JSON report from US-10 appended to or alongside the draft text
- [ ] AC-12.2: `lyrics-structural.md` critic prompt updated to reference tool output sections: "Refer to the PHONETIC ANALYSIS section for meter, stress, and texture data. Do not re-derive these metrics."
- [ ] AC-12.3: Critic prompt's "Phonetic Analysis" section (current section 5) updated to consume tool output rather than instructing the critic to run `bun run tool phonetics`
- [ ] AC-12.4: Non-English lyrics no longer skip phonetic analysis. The tool handles language detection internally. The critic prompt's "When lyrics are non-English: Skip phonetic analysis" removed
- [ ] AC-12.5: Intentionality critic (`lyrics-intentionality.md`) unchanged -- word-level checks (rhyme tax, filler, register, dead metaphors) remain LLM territory

**Priority:** Must Have

---

### US-13: Pre-presentation advisory gate

**As a** user, **I want** to see advisory singability and prosody warnings alongside the presented variation at node 013, **so that** I can make an informed decision about whether to approve, adjust, or investigate before generating audio.

**Acceptance Criteria:**
- [ ] AC-13.1: Node 013 includes a "Technical Notes" section with singability score (if available) and any prosody warnings
- [ ] AC-13.2: Warnings are clearly labeled "ADVISORY" -- they do not block approval
- [ ] AC-13.3: If no T3 tools are shipped yet, this section is omitted rather than showing empty output

**Priority:** Could Have

---

### US-14: Craft reference files for 5 lyricist layers

**As** the Rune agent, **I want** reference material on techniques from 5 elite lyricist archetypes (Architect, Flow Master, Cinematographer, Painter, Compressor), **so that** I can draw on specific craft techniques during generation without those techniques overriding my persona.

**Acceptance Criteria:**
- [ ] AC-14.1: Five reference files created under `vault/studio/references/lyrics/craft/`: `architect.md`, `flow-master.md`, `cinematographer.md`, `painter.md`, `compressor.md`
- [ ] AC-14.2: Each file contains: masters studied, key techniques with examples, when to apply (genre/mood/energy context), anti-patterns to avoid
- [ ] AC-14.3: Files indexed in brain.db via sync. Discoverable via `--get-context` and `discover.ts`
- [ ] AC-14.4: Rune's agent file updated with reference to `vault/studio/references/lyrics/craft/` and instruction to query brain.db for relevant craft layer based on genre/mood
- [ ] AC-14.5: Rune's soul file (`souls/rune.md`) UNCHANGED. Craft layers are technique references, not identity
- [ ] AC-14.6: Loaded on-demand from brain.db, not at startup. Context window efficiency per constraint

**Priority:** Should Have

---

### US-15: Workflow gap fixes

**As a** user running `/lyrics`, **I want** iteration tracking beyond 3 variations, discover.ts integration into the active workflow, rewrite context specification, and self-critic tool-validated loops, **so that** the workflow handles real-world usage patterns without breaking.

**Acceptance Criteria:**
- [ ] AC-15.1: Variation index supports N variations (no hardcoded limit of 3). Variation numbering continues (V4, V5, ...) for rewrites and additions
- [ ] AC-15.2: `discover.ts` output from node 003 feeds into the subagent prompt at node 007 (currently supplementary, should be first-class input)
- [ ] AC-15.3: When user requests adjustment at node 014, the subagent receives: (a) original variation, (b) user's adjustment instructions, (c) tool analysis report from the original, (d) which specific flags the adjustment should address
- [ ] AC-15.4: Self-critic loop at node 008 re-runs analysis tools after rewrites, with a maximum of 2 iterations. Progress logged: "Iteration 1: 3 high flags. Iteration 2: 0 high flags. Proceeding."
- [ ] AC-15.5: Sensory anchor enforcement automated: tool verifies at least one concrete sensory detail per verse/section (see/touch/hear/smell/taste). Advisory flag if missing

**Priority:** Should Have (AC-15.1 through AC-15.4), Could Have (AC-15.5)

---

### US-16: Tagalog/Taglish language detection and routing

**As a** user writing Taglish lyrics, **I want** the analysis tools to automatically detect which words are English and which are Tagalog, routing each to the appropriate analysis pipeline, **so that** I receive accurate syllable counts and texture analysis regardless of language mixing.

**Acceptance Criteria:**
- [ ] AC-16.1: Language detection per word. Primary signal: CMU-dict presence = English, dict-miss = candidate Tagalog
- [ ] AC-16.2: Optional small Tagalog common-word list (top 500-1000 words) to reduce false positives from English slang/proper nouns. Size and necessity determined by feasibility spike (brief UQ3)
- [ ] AC-16.3: Uncertain words (not in CMU dict AND not in Tagalog list) analyzed using vowel-group heuristic as fallback (works for both languages)
- [ ] AC-16.4: Language assignment per word included in JSON output for debugging
- [ ] AC-16.5: No regression in English-only lyrics analysis. All existing phonetics.ts test cases still pass
- [ ] AC-16.6: "ng" digraph in Tagalog correctly handled: treated as single phoneme /N/ for texture analysis, not as n+g cluster

**Priority:** Should Have

---

## Requirements

### Must Have

| ID | Requirement | Traces to |
|----|-------------|-----------|
| M1 | Syllable counter tool (English via `phonemize`/`syllables`, Tagalog via vowel-group heuristic) | US-1 |
| M2 | Stress mapper tool (English via CMU stress markers, Tagalog penultimate-default) | US-2 |
| M3 | Rhyme detector and scorer (end + internal, ARPAbet phoneme comparison, 0-1 scoring) | US-3 |
| M4 | Section balance analyzer (word count, TTR, syllable density, genre target comparison) | US-4 |
| M5 | Unified `lyrics-analyze` CLI and programmatic entry point | US-10 |
| M6 | All tools dual-mode: CLI via tool runner + programmatic via registry `toolDef` | US-1 through US-10 |
| M7 | Existing `phonetics.ts` and `syllable-heuristic.ts` absorbed by new library. No orphaned code | US-1, US-6 |
| M8 | Node 008 self-critic updated to run tools and block on high-severity syllable/stress flags | US-11 |
| M9 | Node 009 structural critic updated to consume tool JSON output | US-12 |
| M10 | Contraction handling: preserve as single tokens, never expand | US-1 (AC-1.6) |
| M11 | Block vs advise enforcement: syllable counter + stress mapper BLOCK. All others ADVISE | US-2, US-3, US-4 |

### Should Have

| ID | Requirement | Traces to |
|----|-------------|-----------|
| S1 | Rhyme scheme identifier (AABB/ABAB/etc per section) | US-5 |
| S2 | Vowel/consonant density analyzer (phoneme feature breakdown per section) | US-6 |
| S3 | Repetition analyzer (exact + near-match phrases, repetition ratio, anaphora detection) | US-7 |
| S4 | Tagalog/Taglish per-word language detection and routing | US-16 |
| S5 | Five craft reference files (Architect, Flow Master, Cinematographer, Painter, Compressor) | US-14 |
| S6 | Agent file update for craft reference loading via brain.db | US-14 |
| S7 | Workflow gap fixes: unlimited variations, discover integration, rewrite context, self-critic loop | US-15 |
| S8 | "ng" digraph handling for Tagalog texture analysis | US-6 (AC-6.4), US-16 (AC-16.6) |

### Could Have

| ID | Requirement | Traces to |
|----|-------------|-----------|
| C1 | Singability composite score (advisory, empirically calibrated over time) | US-8 |
| C2 | Prosody alignment check (advisory, high false-positive risk without melody knowledge) | US-9 |
| C3 | Pre-presentation advisory gate at node 013 | US-13 |
| C4 | Automated sensory anchor enforcement | US-15 (AC-15.5) |

### Won't Have (this version)

| Item | Reason |
|------|--------|
| Languages beyond English, Tagalog, and Taglish | Hard scope boundary. Architecture should not preclude, but we do not build for other languages |
| Full Tagalog pronunciation dictionary | Rule-based analysis sufficient given Tagalog's phonetic regularity |
| Tagalog NLP beyond phonetics | No grammar checking, sentiment, or semantic analysis |
| Melody prediction / audio analysis | We analyze text, not audio |
| Sensory imagery generation | LLM strength -- tools verify presence, LLMs create |
| Emotional arc construction | Rune's arc-first philosophy is sound |
| Metaphor coherence checking | Requires NLP infrastructure beyond current stack |
| Performance notation integration | Separate tool, separate project |
| Persona replacement | Rune's soul file stays unchanged |
| Abstract/concrete language scoring | Requires concreteness lexicon |
| MiniMax-specific lyrics tooling | Universal pipeline improvement |
| Pluggable language framework | EN/TL/Taglish only. Simple if/else, not extensible architecture |

## Technical Requirements

### Architecture

| Requirement | Detail |
|-------------|--------|
| TR-1: Library structure | Single library file `src/libs/lyrics-analyzer.ts` with modular exported functions per tool. Each tool is a pure function that accepts text and returns a typed result object |
| TR-2: Tool files | Each tool gets a thin CLI wrapper in `src/tools/lyrics-analyze.ts` (or `src/tools/lyrics-analyze/` directory with sub-commands). All wrappers follow the `charcount.ts` and `phonetics.ts` patterns: `toolDef` export + `import.meta.main` CLI |
| TR-3: Unified entry point | Single `lyrics-analyze` command with sub-commands (`syllable`, `stress`, `rhyme`, `balance`, `density`, `repetition`, `scheme`, `singability`, `prosody`) or `--all` flag |
| TR-4: phonetics.ts absorption | The new library absorbs all functionality from `src/libs/phonetics.ts`: `checkMeterStress()`, `checkConsonantTexture()`, `checkHomographs()`, `analyzeLyrics()`, `lookupPhonemes()`, `getStresses()`, `syllableCount()`. Callers updated. Old file removed |
| TR-5: syllable-heuristic.ts absorption | `countSyllables()` and `countLineSyllables()` from `src/libs/prompt-data/syllable-heuristic.ts` replaced by new syllable counter. Old file removed. All importers updated |
| TR-6: CMU dict handling | Lazy-loaded singleton (current pattern preserved). Dict loaded once on first access, cached for session |
| TR-7: npm dependencies | `phonemize` (pure JS, G2P fallback for unknown words) and `syllables` (CMU-backed syllable counting). Both must be validated under Bun before committing (brief R1, spike recommended) |
| TR-8: Bun + TypeScript | All code TypeScript. No Python, Node.js, or other runtimes. Bun native APIs where applicable |
| TR-9: Tool runner | All tools callable via `bun run tool lyrics-analyze [subcommand] [args]`. Registered in tool runner auto-discovery |
| TR-10: Output types | Each tool function returns a typed interface (e.g., `SyllableReport`, `StressReport`, `RhymeReport`). Unified report aggregates all sub-reports. JSON serializable |
| TR-11: Performance | All analysis local computation (dictionary lookups + string operations). No API calls. Total execution < 500ms for a typical song (~50 lines) |
| TR-12: Craft references from brain.db | Craft reference files indexed in brain.db. Agent queries `--get-context` with genre/mood context to retrieve relevant craft layer. Never bulk-loaded at startup |

### Data Dependencies

| Dependency | Source | Status |
|------------|--------|--------|
| CMU Pronouncing Dictionary | Already in repo at `src/data/cmudict.json` | Available |
| `phonemize` npm package | Pure JS, 125k+ words, G2P, actively maintained | Requires Bun compatibility spike |
| `syllables` npm package | CMU-backed, actively maintained | Requires Bun compatibility spike |
| Homograph lookup | Already in repo at `src/libs/prompt-data/homographs.ts` | Available, absorbed |
| Tagalog common-word list (500-1000 words) | To be created or sourced. UQ3 from brief | Pending feasibility spike |
| Phoneme feature mapping (for density tool) | ARPAbet phoneme to feature category mapping | Custom build -- bounded data structure |
| Genre density targets | Already in `vault/studio/references/lyrics/rhythm-and-arcs.md`, indexed in brain.db | Available |

## Assumptions

| # | Assumption | Confidence | What breaks if wrong |
|---|-----------|------------|---------------------|
| A1 | `phonemize` runs correctly under Bun | 80% | Fall back to CMU dict + vowel-group heuristic only |
| A2 | `syllables` npm is more accurate than our heuristic | 90% | Keep heuristic as secondary fallback |
| A3 | Rhyme scoring via ARPAbet (65/25/15 weighting) produces meaningful quality distinctions | 85% | Weight tuning iteration |
| A4 | Singability composite correlates with Suno vocal quality | 75% | Ship as advisory only. Collect empirical data via reflect |
| A5 | Prosody alignment without melody knowledge is useful | 70% | High false-positive rate. Advisory only. May be disabled by default |
| A6 | Five craft layers improve output without conflicting with Rune's voice | 85% | Layers are reference material, not persona. Rollback = remove file references |
| A7 | Total tool overhead per workflow run < 500ms | 90% | Profile and optimize. All local computation |
| A8 | Tagalog syllable counting via vowel-group heuristic achieves 95%+ accuracy | 85% | Tagalog is phonetically regular. 15% uncertainty from loanwords |
| A9 | CMU-dict-miss is a viable primary signal for Taglish detection | 70% | False positives on slang/proper nouns. Mitigated by optional Tagalog word list |
| A10 | Tagalog stress defaults to penultimate (malumay) | 75% | Exception patterns need validation. Start with default, build list iteratively |
| A11 | Consonant texture analysis works cross-language | 90% | Only risk is "ng" digraph. Bounded fix |
| A12 | Contractions must be preserved as single tokens | 95% | Research definitively confirms |

## Dependencies

| Dependency | Nature | Blocks |
|-----------|--------|--------|
| Bun compatibility spike for `phonemize` and `syllables` | Technical validation | Phase 1 implementation |
| Tagalog common-word list sourcing/creation | Data dependency | S4 (Taglish language detection) |
| brain.db sync for craft references | Existing infrastructure | S5, S6 |
| Existing lyrics workflow nodes (007, 008, 009, 012, 013) | Integration surface | M8, M9, US-13 |
| Existing critic prompts (`lyrics-structural.md`) | Integration surface | M9, US-12 |
| Tool runner auto-discovery | Infrastructure | Already works |

## Phasing

### Phase 1: Tools Foundation

**Scope:** Library skeleton + 4 T1 tools + unified CLI + Tagalog/Taglish language detection + phonetics.ts absorption

**Delivers:** M1, M2, M3, M4, M5, M6, M7, M10, M11, S4, S8

**User stories:** US-1, US-2, US-3, US-4, US-10, US-16

**Exit criteria:**
- All 4 T1 tools pass against test corpus (10+ good, 10+ bad lyrics)
- `phonetics.ts` and `syllable-heuristic.ts` fully absorbed
- `bun run tool lyrics-analyze --file <path>` produces unified report
- Tagalog syllable counting within +/-1 for 90%+ of lines
- No regression in English-only analysis

### Phase 2: Tools Expansion

**Scope:** 3 T2 tools + 2 T3 tools (advisory)

**Delivers:** S1, S2, S3, C1, C2

**User stories:** US-5, US-6, US-7, US-8, US-9

**Exit criteria:**
- All T2 tools pass against test corpus
- T3 tools operational as advisory (no blocking)
- Singability score calibration data collection mechanism in place

### Phase 3: Craft + Workflow Integration

**Scope:** 5 craft reference files + agent file updates + workflow node modifications + critic prompt updates + workflow gap fixes

**Delivers:** M8, M9, S5, S6, S7, C3, C4

**User stories:** US-11, US-12, US-13, US-14, US-15

**Exit criteria:**
- `/lyrics` skill runs end-to-end with tool checkpoints at nodes 008, 009, 013
- Structural critic consumes tool output (no longer runs its own phonetic checks)
- Craft references indexed in brain.db and discoverable
- Rune's soul file unchanged

## Out of Scope

Fully specified in the Won't Have section above. Key boundaries:

- **No multilingual framework.** EN/TL/Taglish only. Simple conditional routing, not a pluggable architecture.
- **No audio analysis.** Text-only pipeline.
- **No persona changes.** Rune's soul file is identity. Craft layers are additive technique references.
- **No replacement of LLM-appropriate tasks.** Tools verify what LLMs cannot. LLM retains ownership of meaning, emotion, metaphor, narrative, register.

## Unresolved Questions

| # | Question | Needed to resolve | Owner | Blocks |
|---|----------|-------------------|-------|--------|
| UQ1 | Actual distribution of EN vs TL vs Taglish in user's lyrics | Guides language detection priority | Holmes | Nothing (nice-to-know) |
| UQ2 | Does `syllable-heuristic.ts` work for Tagalog or do English rules corrupt it? | Feasibility spike | Ryan | Phase 1 Tagalog accuracy |
| UQ3 | Taglish detection: CMU-dict-miss alone or also Tagalog word list? | Trade-off: zero-maintenance vs precision | Holmes | S4 completeness |
| UQ4 | Should Tagalog stress analysis ship in Phase 1 or defer? | If exception list too large, defer | Holmes + McCall | Phase 1 scope |
| UQ5 | Tagalog-specific clusters beyond "ng"? | Glottal stops may affect analysis | Ryan | S8 completeness |

## Appendix A: Existing Code Interfaces to Absorb

### `src/libs/phonetics.ts` (438 lines)

Exported functions to absorb or replace:
- `lookupPhonemes(word: string): string | null` -- CMU dict lookup. Preserve as internal utility
- `getStresses(word: string): number[] | null` -- stress extraction. Absorb into stress mapper
- `syllableCount(word: string): number | null` -- CMU-based count. Replace with new syllable counter
- `checkMeterStress(lyrics: string): StressFlag[]` -- LY2 check. Absorb into stress mapper
- `checkConsonantTexture(lyrics: string): TextureFlag[]` -- LY5 check. Absorb into density analyzer
- `checkHomographs(lyrics: string): HomographFlag[]` -- LY6 check. Preserve in new library
- `analyzeLyrics(lyrics: string): PhoneticReport` -- orchestrator. Replace with unified analyzer

Exported types to preserve or extend: `StressFlag`, `TextureFlag`, `HomographFlag`, `PhoneticReport`

### `src/tools/phonetics.ts` (127 lines)

CLI wrapper + `toolDef`. Replace with `lyrics-analyze` tool.

### `src/libs/prompt-data/syllable-heuristic.ts` (70 lines)

- `countSyllables(word: string): number` -- vowel-group heuristic. Replace
- `countLineSyllables(line: string): number` -- line-level aggregation. Replace

### `src/tools/charcount.ts` (pattern reference only)

Not absorbed. Referenced as the architectural template for dual-mode tools.

## Appendix B: Workflow Integration Points

### Node 008 (Self-Critic) -- MODIFIED

**Current behavior:** Rune reads draft line-by-line, applies 9 manual checks, then runs `bun run tool phonetics --file out/rune-draft.md` for LY2/LY5.

**New behavior:** Rune reads draft, applies manual checks, then runs `bun run tool lyrics-analyze --file out/rune-draft.md --json`. Receives structured report. Syllable counter and stress mapper flags with severity "high" are BLOCKING -- Rune must rewrite. All other tool flags are advisory context. Re-run tools after rewrite (max 2 iterations).

### Node 009 (Gemini Structural Critic) -- MODIFIED

**Current behavior:** Critic prompt instructs Gemini to run `bun run tool phonetics` itself and check LY2/LY5/singability.

**New behavior:** Tool report from node 008 included in critic input. Critic prompt updated to reference tool output. Non-English skip clause removed.

### Node 013 (Present to User) -- MODIFIED (Phase 3, Could Have)

**Current behavior:** Shows approach, lyrics, style block notes.

**New behavior:** Adds "Technical Notes" section with singability score and prosody warnings. Labeled ADVISORY.

### Nodes 010, 011 (MiniMax + Grok Critics) -- UNCHANGED

Intentionality and wild-card critics remain LLM-only.

### Node 012 (Reconcile) -- MINOR UPDATE

Reconciliation table gains a "Tool" column for flags originating from analysis tools. Tool-originated high-severity flags that survived node 008 carry mandatory-fix weight.

## Appendix C: Risk Register

| # | Risk | L | I | Mitigation | PRD Action |
|---|------|---|---|-----------|------------|
| R1 | npm packages don't work under Bun | Low | Med | Spike: test 100 words | Phase 1 gate |
| R2 | Singability score doesn't correlate | Med | High | Advisory only. Calibrate via reflect | C1 advisory. Weights configurable |
| R3 | Prosody alignment false positives | Med | Med | Advisory only. Configurable thresholds | C2 advisory. Default strict=2 |
| R4 | Scope creep across 3 pillars | Med | High | Phase the work | 3 phases with exit criteria |
| R5 | Craft layers conflict with Rune | Low | High | Reference material only | US-14 AC-14.5: soul UNCHANGED |
| R6 | Tool overhead | VLow | Low | All local computation | TR-11: profile if >100ms |
| R7 | CMU dict coverage gaps | Med | Med | `phonemize` G2P | US-1 AC-1.3: flag approx words |
| R8 | Nine tools at once | Med | Med | Tier the tools | Phased delivery |
| R9 | Taglish detection low accuracy | Med | Med | Vowel-group fallback | US-16 AC-16.3 |
| R10 | Tagalog stress inaccurate | Med | Low | Penultimate-default + exception list | US-2 AC-2.3 |
| R11 | "ng" digraph mishandled | High | Low | Tagalog-aware digraph list | S8 |
| R12 | Tagalog scope creep to multilingual | Low | High | Hard boundary: EN/TL/Taglish | Won't Have |
| R13 | Test corpus quality | Med | Med | Real user lyrics, not just synthetic | Phase 1 exit criteria |
| R14 | Rhyme weight tuning | Med | Low | Initial weights from research. Iterate | US-3 AC-3.3: configurable |
