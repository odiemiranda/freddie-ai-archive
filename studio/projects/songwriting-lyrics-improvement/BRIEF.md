---
title: "Songwriting and Lyrics Composition Improvement"
type: brief
project: songwriting-lyrics-improvement
status: approved
created: 2026-04-03
author: holmes
research: db4502a2
---

# Songwriting and Lyrics Composition Improvement

## Problem

**Hypothesis:** Building a comprehensive lyrics analysis toolkit, upgrading Rune's craft DNA with techniques from elite lyricists, and tightening the lyrics workflow will produce measurably better vocal output from Suno AI and MiniMax Music, because our current tooling covers approximately 15% of what is algorithmically verifiable, and LLMs are provably unreliable at the remaining 85% (syllable counting, stress pattern detection, rhyme quality scoring, section balance analysis).

**Five Whys:**

1. **Why do AI-generated tracks sometimes sound "off" vocally?** Because the lyrics contain stress misalignments, overstuffed lines, and forced rhymes that cause the vocal engine to rush, mumble, or mis-emphasize words.
2. **Why do those problems reach the vocal engine?** Because the LLM drafting the lyrics cannot reliably count syllables, detect stress patterns, or assess rhyme quality -- these are fundamentally non-linguistic tasks that require phonetic computation, not token prediction.
3. **Why doesn't the existing toolchain catch them?** Because `phonetics.ts` only checks two dimensions (LY2 meter/stress deviations and LY5 consonant texture clashes, plus LY6 homographs). There is no syllable counter, no rhyme detector, no section balance analyzer, no singability scorer, no prosody alignment check.
4. **Why were those tools never built?** Because the initial phonetics implementation was scoped as a proof-of-concept to validate the "deterministic check in the lyrics pipeline" pattern. It proved the pattern works -- the structural and intentionality critics now reference phonetic output -- but the scope was never expanded.
5. **Why does this matter now?** Because Rune's creative philosophy and workflow architecture are mature (tri-critic pipeline, Four Gates, 9 lyrical approaches, genre-aware density targets, typed brackets), meaning the bottleneck has shifted from "can we write good lyrics" to "can we verify they're good before they reach the vocal engine."

The user writes lyrics in three language modes: **English, Tagalog, and Taglish** (Tagalog-English code-switching within a single line or verse). The current phonetic analysis pipeline (CMU dict + LY2/LY5) is English-only -- Tagalog and Taglish lyrics silently skip all phonetic checks. This means roughly a third of the user's creative output receives no meter, stress, or texture validation. Rune already has a knowledge rule ("Non-English vocab never fabricated, verify realness") but no tooling enforces phonetic quality for non-English words.

**Who feels the pain:** The user, who hears vocal artifacts (rushed lines, mis-stressed words, weak rhymes) in generated tracks and must either accept them, manually rewrite, or burn generation credits re-rolling. Rune, whose Four Gates self-critic relies on the LLM's own judgment for syllable counting and rhyme assessment -- the exact tasks where LLMs fail hardest.

**Evidence sources:**
- NotebookLM research (db4502a2, ~80 sources, 12 queries): Confirms CMU dict + ARPAbet as standard foundation for computational prosody. Documents 8 LLM failure mode categories with algorithmic countermeasures. Validates topline writing techniques as directly applicable to Suno.
- brain.db decisions: Past decisions confirm phonetics tool pattern works within workflow. Performance Notation Tool and Phase 3 Vocal Lyrics Builder decisions show architectural precedent.
- Codebase review: `phonetics.ts` (438 lines) implements LY2/LY5/LY6 against CMU dict. Heuristic syllable counter uses vowel-group counting, known inaccurate for ~15-20% of English words.

## Goals

| # | Goal | Measure |
|---|------|---------|
| 1 | Expand deterministic coverage from ~15% to ~80% of algorithmically checkable lyric dimensions | 9 analysis tools operational. Phonetic analysis covers 90%+ of words in English lyrics, 85%+ in Tagalog, 80%+ in Taglish. |
| 2 | Reduce vocal artifacts in generated tracks | Fewer re-rolls. Measurable via reflect workflow outcomes logged to brain.db. |
| 3 | Upgrade Rune's craft vocabulary without replacing persona | 5 craft layers (Architect, Flow Master, Cinematographer, Painter, Compressor) integrated as reference material -- not persona overrides. |
| 4 | Wire tools into workflow automatically | Analysis at defined checkpoints (post-draft, pre-critic, pre-presentation) without manual invocation. Reduces critic API cost on fundamentally flawed lyrics. |
| 5 | Fix known workflow gaps | Iteration tracking, discover integration, rewrite context, self-critic loop with tool-validated data. |

## Three Pillars

### Pillar 1 -- Lyrics Analysis Toolkit (tools)

Build 9 deterministic analysis tools to replace LLM judgment with algorithmic precision:

| # | Tool | Function | Data Source | Build Status |
|---|------|----------|------------|--------------|
| 1 | Syllable Counter | Exact count per word/line | CMU dict via `syllables`/`phonemize` npm; Tagalog rule-based | Ready (npm) + custom (Tagalog) |
| 2 | Stress Mapper | S/W pattern per line, metrical feet | CMU stress markers + POS tagger; Tagalog penultimate-default | Custom build |
| 3 | Rhyme Detector & Scorer | Identify rhymes, score quality (perfect/slant/assonance), internal + end rhymes | ARPAbet phonemes, cosine similarity | Custom build |
| 4 | Rhyme Scheme Identifier | Output AABB/ABAB/etc pattern | Rhyme detector output | Custom build |
| 5 | Vowel/Consonant Density | Phonetic texture ratio (open vowels vs plosives vs liquids) | Phoneme feature mapping | Custom build |
| 6 | Section Balance | TTR, word count per section, structural complexity | Tokenized text | Custom build |
| 7 | Repetition Analyzer | Detect hooks, refrains, similarity ratios | SequenceMatcher port / Levenshtein | Custom build |
| 8 | Singability Score | Composite: line length + vowel placement + breath points + density | Syllable + density + phonetics | Custom build |
| 9 | Prosody Alignment | Stress-to-beat mapping, constraint violations | Stress map + POS + target meter | Custom build |

**Foundation:** `phonemize` (pure JS, 125k+ words, G2P, actively maintained) + `syllables` (CMU-backed, actively maintained). Architecture: `src/libs/lyrics-analyzer.ts` library with modular functions. Each tool also CLI via `src/tools/`. Existing `phonetics.ts` absorbed/replaced.

**Tiering:**
- **T1 (must ship):** Syllable counter, stress mapper, rhyme detector, section balance
- **T2 (should ship):** Vowel/consonant density, repetition analyzer, rhyme scheme
- **T3 (could ship):** Singability score, prosody alignment

**Block vs advise (resolved via research):**
- **BLOCK:** Syllable counter (syllables must match notes -- mechanical necessity), stress mapper (unstressed words on downbeats = unintelligible)
- **ADVISE:** Everything else (rhyme quality, density, balance, scheme, repetition, singability, prosody) -- stylistic choices where subversion is valid

### Pillar 2 -- Agent Craft Upgrade (soul + references)

Integrate 5 craft layers from research into Rune's references. NOT a persona replacement -- technique expansion. Rune absorbs them like studying the masters.

| Layer | Masters | Techniques |
|-------|---------|------------|
| The Architect | Max Martin | Syllable symmetry ("melodic math"), hook placement (50-second rule), melodic economy (3-4 motifs), glue hooks (melodic previews) |
| The Flow Master | Paul Simon, Lin-Manuel Miranda, Kendrick Lamar | Enjambment, multisyllabic/mosaic rhymes, internal rhyme density, rhythmic driving, flexible schemes (ABAA) |
| The Cinematographer | Bob Dylan | Slant rhymes over perfect, perspective/tense shifts, cinematic narrative arc, dynamic storytelling |
| The Painter | Joni Mitchell, Leonard Cohen | Internal assonance, compound images (multiple meanings per phrase), extended metaphor, subtext over statement |
| The Compressor | Tracy Chapman, Johnny Cash | Word economy, scaffolding removal ("just"/"really"/"well"), monosyllabic power, one-image verses |

### Pillar 3 -- Workflow Tightening (skill + workflow)

Wire new tools into lyrics workflow and fix identified gaps:

**New tool checkpoints:**
- Post-draft (node 008): Syllable symmetry check + stress mapping -- BLOCKING
- Pre-critic (node 009): Rhyme analysis (scheme + quality + internal) + section balance -- advisory data fed to critics
- Pre-presentation (node 013): Singability gate + prosody alignment -- advisory warnings shown to user

**Gap fixes:**
- Iteration/version tracking beyond 3 variations
- Discover.ts integration into active workflow (not supplementary)
- Rewrite context specification (what gets passed on user adjustment)
- Self-critic loop (re-run analysis tools after fixes, not just one pass)
- Sensory anchor automated enforcement

## Constraints

| Constraint | Source | Negotiable? |
|------------|--------|-------------|
| Bun + TypeScript stack | Project-wide | No |
| Tool runner pattern (`bun run tool <name>`) | Architecture rule | No |
| CLI-first, never MCP from local | Architecture rule | No |
| All tools dual-mode: CLI + programmatic via registry | Architecture rule | No |
| Existing phonetics.ts patterns (CMU dict, ARPAbet) | Codebase precedent | No -- extend, don't replace |
| Tri-critic pipeline preserved | Working system | No |
| Four Gates persona preserved | Working system | No |
| Genre density targets preserved | Working system | No |
| Phonetic analysis supports English, Tagalog, and Taglish | User requirement | No |
| No Tagalog pronunciation dictionary at CMU-dict scale | Filipino NLP resources are sparse | Design constraint -- use rule-based analysis |
| `phonemize` and `syllables` npm packages | Research-confirmed Bun-compatible | Verify at implementation |
| Rune's soul file is identity, not instructions | Agent architecture | No -- craft layers go in references + agent, NOT soul |
| Craft references queried from brain.db on demand | Research recommendation -- context window efficiency | No -- don't load at startup |

## Assumptions

| # | Assumption | Confidence | Risk if wrong |
|---|-----------|------------|---------------|
| A1 | `phonemize` (pure JS, 125k+ words, G2P fallback) runs correctly under Bun | 80% | Medium -- fallback to CMU dict + heuristic only |
| A2 | `syllables` npm package is more accurate than our heuristic | 90% | Low -- keep heuristic as fallback |
| A3 | Rhyme quality can be meaningfully scored via ARPAbet phoneme comparison (weighted: stressed vowel 65%, tail consonant 25%, prefix 15%) | 85% | Medium -- weight tuning may need iteration |
| A4 | Composite singability score correlates with actual Suno vocal quality. Key metrics: Dur-str (long notes + stressed syllables), Peak-str (pitch peaks + stress), Line-len (syllable = note count) | 75% | **High** -- needs empirical calibration via reflect outcomes |
| A5 | Prosody alignment is useful without knowing the actual melody | 70% | **High** -- predicting beat placement, not matching known melody. May produce false positives. |
| A6 | Five craft layers will improve output without conflicting with Rune's existing voice | 85% | Medium -- layers are additive techniques, not personality changes |
| A7 | Tool overhead per workflow run stays under 500ms total | 90% | Low -- all local dict lookups + string analysis |
| A8 | Tagalog syllable counting via vowel-group heuristic will achieve 95%+ accuracy | 85% | Low -- Tagalog is phonetically regular (5 vowels, no silent letters, no diphthong ambiguity). 15% uncertainty from Spanish/English loanwords. |
| A9 | Taglish code-switch detection can use CMU-dict-miss as primary signal (word not in CMU dict = likely Tagalog) | 70% | Medium -- false positives on English slang/proper nouns, false negatives on absorbed loanwords ("computer", "taxi"). May need small Tagalog common-word list. |
| A10 | Tagalog stress defaults to penultimate syllable (malumay) | 75% | Medium -- exception patterns (maragsa, malumi, mabilis) follow morphological rules but need validation against actual lyrics. |
| A11 | Consonant texture analysis (LY5) works across languages without modification | 90% | Low -- only risk is Tagalog "ng" digraph (/N/) being misclassified. Bounded fix. |
| A12 | Contractions (don't, can't, I'm) must be preserved as single tokens, never expanded -- expanding changes syllable count and rhythm | 95% | Low -- research confirms this definitively. |

**Flagged risky assumptions:** A4 (singability weighting) and A5 (prosody without melody) require empirical validation. Ship as advisory until calibrated. A9 (Taglish detection) needs a feasibility spike.

## Success Criteria

### Core Tools
- [ ] 9 analysis tools implemented, each with CLI + programmatic mode + registry toolDef
- [ ] All tools pass against a test corpus of 10+ known-good and 10+ known-bad lyrics (true positive rate >80%, false positive rate <20%)
- [ ] `phonetics.ts` absorbed by the new analyzer library -- no orphaned code
- [ ] `bun run tool lyrics-analyze --file <path>` runs all 9 tools and produces a unified report (per-tool scores + composite)
- [ ] Rhyme detector analyzes both end rhymes and internal rhymes

### Workflow Integration
- [ ] Lyrics workflow updated with tool checkpoints at nodes 008 (blocking), 009 (advisory), 013 (advisory)
- [ ] Structural critic updated to consume tool output instead of running its own phonetic checks
- [ ] Intentionality critic unchanged (word-level checks remain LLM territory)

### Craft Upgrade
- [ ] Five craft layers documented in reference files under `vault/studio/references/lyrics/`
- [ ] Rune's soul file unchanged; agent file updated with new tool invocations and craft reference pointers

### Tagalog/Taglish
- [ ] Tagalog lyrics receive syllable count validation (within +/-1 syllable per line for 90%+ of lines)
- [ ] Taglish lyrics analyzed with correct language detection per word (80%+ accuracy against manually labeled test set of 50+ lines)
- [ ] Tagalog "ng" digraph does not produce false texture flags
- [ ] No regression in English-only phonetic analysis

### End-to-End
- [ ] Run `/lyrics` with upgraded pipeline, confirm tool output appears in self-critic and critic passes
- [ ] Existing tests still pass; new tool tests added

## Risks

| # | Risk | Likelihood | Impact | Mitigation |
|---|------|-----------|--------|------------|
| R1 | npm packages don't work under Bun | Low (20%) | Medium | Spike: test both packages against 100 words before committing |
| R2 | Singability score doesn't correlate with Suno vocal quality | Medium (40%) | High | Ship as advisory only. Collect data from real generations to calibrate via reflect. |
| R3 | Prosody alignment produces excessive false positives | Medium (35%) | Medium | Advisory only. Configurable thresholds. |
| R4 | Scope creep across three pillars | Medium (30%) | High | **Phase the work.** Pillar 1 first. Pillar 3 depends on Pillar 1. |
| R5 | Craft layers conflict with existing Rune behavior | Low (15%) | High | Layers are reference material, not soul overrides. A/B comparison on 3 genres. |
| R6 | Tool overhead slows the workflow | Very Low (10%) | Low | All local computation. Profile if >100ms. |
| R7 | CMU dict coverage gaps for slang/neologisms | Medium (30%) | Medium | `phonemize` G2P handles unknowns. Flag unverified words. |
| R8 | Nine tools at once -- quality suffers | Medium (25%) | Medium | **Tier the tools.** T1 alone covers ~50% of the gap. |
| R9 | Taglish code-switch detection has low accuracy | Medium (30%) | Medium | Fail-safe: when uncertain, use vowel-group heuristic (works for both). Log uncertain words for iterative improvement. |
| R10 | Tagalog stress estimation is inaccurate | Medium (25%) | Low | Start with penultimate-default. Build exception list from user's actual lyrics via self-learning. |
| R11 | "ng" digraph mishandled by consonant cluster detection | High (60%) | Low | Add "ng" to Tagalog-aware digraph list. Bounded fix. |
| R12 | Scope creep from Tagalog into general multilingual infrastructure | Low (15%) | High | Hard boundary: EN + TL + Taglish ONLY. Simple if/else, not pluggable framework. |

## Options

### Option A: Full Three-Pillar Build (Recommended)

Build all 9 tools as `src/libs/lyrics-analyzer.ts`, upgrade craft references with 5 lyricist layers, wire into workflow. Trilingual support (English + Tagalog + Taglish).

**Phasing:**
- **Phase 1** (Tools foundation): Library + 4 T1 tools (syllable, stress, rhyme, balance) + unified CLI + Tagalog/Taglish language detection
- **Phase 2** (Tools expansion): 3 T2 tools (density, repetition, scheme) + 2 T3 tools (singability, prosody as advisory)
- **Phase 3** (Craft + Workflow): 5 craft reference files + agent updates + workflow modifications + critic updates

**Risk:** Largest scope. Highest payoff. Phasing mitigates "build too much at once."

### Option B: Tools Only (Conservative)

Build only the 9 tools. Skip craft upgrade. Solves verification but not generation quality.

### Option C: Incremental (Minimum Viable)

Build only 4 T1 tools. Moves coverage from ~15% to ~50%. Safest but leaves most value on the table.

**Recommendation:** Option A, phased. Ship Phase 1, validate against real generations, then proceed.

## Out of Scope

| Item | Reason |
|------|--------|
| Languages other than English, Tagalog, and Taglish | Hard scope boundary. Design shouldn't preclude future languages but we don't build for them. |
| Full Tagalog pronunciation dictionary | Rule-based analysis sufficient given phonetic regularity. |
| Tagalog NLP beyond phonetics | No grammar checking, sentiment, or semantic analysis. Scope: syllable counting, stress estimation, consonant texture. |
| MiniMax-specific lyrics tooling | Universal pipeline improvement, not platform-specific. |
| Melody prediction / audio analysis | We analyze text, not audio. |
| Sensory imagery generation | LLM strength. Tools verify; LLM creates. |
| Emotional arc construction | Rune's arc-first philosophy is sound. |
| Metaphor coherence checking | Needs NLP infrastructure beyond current stack. Deferred. |
| Performance notation | Separate tool, separate project. |
| Persona replacement | Soul file stays. Craft layers are additive. |
| Abstract/concrete language scoring | Needs concreteness lexicon. Deferred. |

## Resolved Questions

| # | Question | Resolution | Source |
|---|----------|------------|--------|
| RQ1 | Composite score or per-tool? | **Both.** Per-tool scores + weighted composite. Professional tools (REFFLY) use per-pair confidence + overall quality. | NotebookLM |
| RQ2 | Block vs advise? | **Mechanical necessity = block, stylistic choice = advise.** Syllable counter + stress mapper block. Everything else advises. | NotebookLM |
| RQ3 | Startup load vs on-demand? | **On-demand from brain.db.** CMU dict loaded lazily once. Craft references queried, not loaded into context window. | NotebookLM |
| RQ4 | Validate singability empirically? | **Coexistence probabilities:** Dur-str, Peak-str, Line-len. Correlate with reflect generation outcomes. | NotebookLM |
| RQ5 | Contractions + hyphenated words? | **Preprocess but never expand.** Strip punctuation for lookup, keep contractions as single tokens. "don't" != "do not". | NotebookLM |
| RQ6 | Internal rhymes? | **Must include.** Internal rhymes doubled 1990s-2010s, multisyllabic up 232%. Significant correlation with critical acclaim. End-only is "fundamentally unsustainable." | NotebookLM |

## Unresolved Questions

| # | Question | Needed to resolve | Owner |
|---|----------|-------------------|-------|
| UQ1 | What is the actual distribution of English vs Tagalog vs Taglish in the user's lyrics? | Guides priority for language detection investment | Holmes -- analyze past lyrics |
| UQ2 | Does the existing `syllable-heuristic.ts` work for Tagalog, or do English-specific rules (silent-e, consonant-le) corrupt it? | Feasibility spike needed | Ryan -- test against 50 Tagalog words |
| UQ3 | Should Taglish detection use CMU-dict-miss alone, or also a small Tagalog common-word list (top 500-1000)? | Trade-off: zero-maintenance vs precision | Holmes -- evaluate after spike |
| UQ4 | Should Tagalog stress analysis ship in Phase 1 or defer to Phase 2? | If exception list is too large/error-prone, defer and ship syllable + texture first | Holmes + McCall |
| UQ5 | Are there Tagalog-specific clusters beyond "ng" that need dedicated texture rules? | Glottal stops between vowels may affect analysis | Ryan -- investigate during implementation |

## Research Sources

- **NotebookLM notebook:** db4502a2 "Songwriting and Lyrics Composition Improvement" (~80 sources, 12 queries)
- **Saved research:** `vault/studio/research/2026-04-03-songwriting-lyrics-improvement/`
- **Topics covered:** Songwriting craft, pure lyricist analysis (Mitchell/Dylan/Simon/Cohen/Chapman/Cash), computational poetry tools (prosodic/ZeuScansion/CMU/phonemize), Suno-specific practices, LLM lyric failure modes (8 categories), TypeScript/Bun npm ecosystem, toolkit architecture design, topline writing for AI, prosody alignment, phonetic similarity vectors
