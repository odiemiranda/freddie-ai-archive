# Create Track Redesign — Technical Specification

## Overview

Replace `/suno` (25-node, 3-variation, folder output) with two simpler phase-gated skills:

| Skill | Purpose |
|-------|---------|
| `/create-track` | Build one Suno track from a user prompt |
| `/create-track-variant` | Build a variant from an existing track file |

**Key changes from `/suno`:**
- Single variation per run (not 3)
- Single `.md` file output (not folder with index + variations)
- Freddie orchestrates user interaction; Tala handles analysis + generation as sub-agent
- Explicit approve/adjust/retry/cancel gates at each phase
- Critic digest presented as summary tables, not raw output

---

## Architecture

### Execution Model

```
User prompt
    |
    v
[Freddie: main agent, orchestrator]
    |
    |-- Phase 1: fork to Tala sub-agent --> analysis
    |-- Phase 2: present analysis, user review gate
    |-- Phase 3: fork to Tala sub-agent --> generation + tri-critic
    |-- Phase 4: present generation, user review gate
    |-- Phase 5: save to disk
```

Freddie owns all user interaction. Tala is invoked via `Agent` tool with `subagent_type: "tala"` for bounded work in Phase 1 and Phase 3. No context fork — skills run inline per user preference.

### Agent Roles

| Agent | Responsibility |
|-------|---------------|
| **Freddie** (main) | Parse user intent, call tools for discovery/brain, delegate to Tala, present review tables, handle user choices, save output |
| **Tala** (sub-agent) | Analyze prompt ingredients, build Style/Exclude/W-SI/Lyrics, run charcount, run tri-critic, generate track names |

---

## Phase Flow: `/create-track`

### Phase 1 — Analysis

**Executor:** Freddie delegates to Tala sub-agent

**Steps:**
1. Query `brain.db` for past decisions on similar genres/instruments
2. Run `discover --skill suno` with user prompt keywords
3. Parse user prompt — extract explicit constraints (genre, mood, instruments, BPM, scene)
4. Normalize genres (max 2, check genre gravity)
5. Identify music card matches (if any)
6. Identify archetype matches (if any)
7. Distill ingredients + health checks (frequency map, instrument count <= 3)
8. Build vibe map translation

**Returns to Freddie:** Structured analysis object with all dimensions.

### Phase 2 — Analysis Review

**Executor:** Freddie (main agent)

**Presents:**

```
| Dimension   | Value              | Source        | Explanation                |
|-------------|--------------------|---------------|----------------------------|
| Genre       | Jazzhop            | User prompt   | ...                        |
| Mood        | warm, expectant    | User prompt   | ...                        |
| Instruments | shamisen, harp     | User prompt   | Added subtle percussion    |
| BPM         | 78                 | Tala + ref    | Matches morning vibe       |
| Key         | A Minor            | Tala + ref    | Warm minor for coffee mood |
| Music Card  | none               | discover      | No match found             |
| Archetype   | none               | discover      | No archetype applied       |
```

**User choices:**
- `[1] Approve` — proceed to Phase 3
- `[2] Adjust` — user provides modifications, re-run Phase 1 with adjustments appended
- `[3] Retry` — re-run Phase 1 from scratch (fresh analysis)
- `[4] Cancel` — abort

### Phase 3 — Generation

**Executor:** Freddie delegates to Tala sub-agent

**Input:** Locked analysis from Phase 2

**Steps:**
1. Load matched references (full sections from discover results)
2. Build Style block (~75-120 chars, 11-slot formula)
3. Validate charcount via `charcount` tool
4. Build Exclude Style (focused, not kitchen-sink)
5. Write Lyrics block (typed brackets, piping, layer lines)
6. Vocal performance pass (if vocal track)
7. Set W/SI values with justification
8. Run tri-critic:
   - Gemini: `production-musical` critic
   - MiniMax: `production-effectiveness` critic
   - Grok: wild-card critic
9. Reconcile critic feedback — accept/reject each flag with reasoning
10. Generate 18 track names via `tracknames` tool

**Returns to Freddie:** Style, Exclude Style, W/SI, Lyrics, all critic outputs (raw + digest), track names, reconciliation decisions.

### Phase 4 — Generation Review

**Executor:** Freddie (main agent)

**Presents (in order):**

**1. Critic Digest Table:**
```
| Critic  | Flags | Accepted | Key Changes                          |
|---------|-------|----------|--------------------------------------|
| Gemini  | 3     | 2/3      | Swapped register for lead shamisen   |
| MiniMax | 1     | 1/1      | Added behavior verb to harp tag      |
| Grok    | 2     | 1/2      | Rejected: wholesale mood rewrite     |
```

**2. Per-critic narrative** (2-3 sentences each — what was flagged, accepted, rejected)

**3. Track Summary Table** (same format as Phase 2, now with final values post-critic)

**4. Generated Output:**
- Style block
- Exclude Style
- W/SI with justification
- Lyrics (full)

**User choices:**
- `[1] Approve` — proceed to Phase 5
- `[2] Adjust` — user provides modifications, re-run Phase 3 with adjustments
- `[3] Retry` — re-run Phase 3 from scratch (new generation)
- `[4] Cancel` — abort

User can manually test in Suno before deciding.

### Phase 5 — Save

**Executor:** Freddie (main agent)

**Steps:**
1. Get date/time via `getdate` tool
2. Use track name #1 (from tracknames output) as filename slug
3. Write single `.md` file to `vault/studio/tracks/YYYY-MM-DD/HHMMSS-<track-name-slug>.md`
4. Run `reflect` for learning capture + brain.db decision logging

---

## Phase Flow: `/create-track-variant`

### Phase 0 — Load Source

**Executor:** Freddie (main agent)

1. Read source track `.md` file
2. Extract: Analysis table, Style, Exclude Style, W/SI, Lyrics, frontmatter
3. Validate file structure (has expected sections)

### Phases 1-5 — Same as `/create-track` with modifications:

- **Phase 1:** Tala receives existing analysis + Style + Lyrics as context, plus modifier prompt. Identifies what changes vs what carries forward. Returns diff-style analysis ("inherited" vs "modified").
- **Phase 2:** Summary table includes "Source" indicator (inherited/modified per dimension).
- **Phases 3-5:** Same as create-track. Variant saves as new file, never overwrites source.

**Save path:** `vault/studio/tracks/YYYY-MM-DD/HHMMSS-<variant-name-slug>.md`

---

## Output File Format

Single `.md` file containing everything. No folder, no index, no separate variation files.

```markdown
---
id: track-{YYYYMMDD}-{HHMMSS}
title: "{Track Name #1}"
created: YYYY-MM-DD HH:MM:SS
tags: [genre, instrument, mood tags]
platform: suno
genre: "{genres}"
mood: "{moods}"
bpm: {number}
key: "{key}"
type: "{instrumental|vocal}"
variant_of: "{path to source track, only for variants}"
---

# {Track Name #1}

> Prompt: {original user prompt}
> Created: YYYY-MM-DD HH:MM:SS AM/PM UTC+N

---

## Analysis

| Dimension   | Value              | Source        | Explanation                |
|-------------|--------------------|---------------|----------------------------|
| Genre       | ...                | ...           | ...                        |
| Mood        | ...                | ...           | ...                        |
| Instruments | ...                | ...           | ...                        |
| BPM         | ...                | ...           | ...                        |
| Key         | ...                | ...           | ...                        |
| Music Card  | ...                | ...           | ...                        |
| Archetype   | ...                | ...           | ...                        |

---

## Style

```
{style block — ready to paste into Suno}
```

## Exclude Style

```
{exclude style}
```

## Advanced Settings

- **Weirdness**: X% — {justification}
- **Style Influence**: X% — {justification}

## Lyrics

```
{full lyrics with typed brackets — ready to paste into Suno}
```

---

## Critic Digest

| Critic  | Flags | Accepted | Key Changes                     |
|---------|-------|----------|---------------------------------|
| Gemini  | N     | N/N      | ...                             |
| MiniMax | N     | N/N      | ...                             |
| Grok    | N     | N/N      | ...                             |

### Gemini (Production-Musical)
{2-3 sentence summary}

### MiniMax (Production-Effectiveness)
{2-3 sentence summary}

### Grok (Wild-Card)
{2-3 sentence summary}

---

## Track Names

| # | Name | Model | Strategy |
|---|------|-------|----------|
| 1 | {name} | {gemini/minimax/grok} | {strategy label} |
| ... | ... | ... | ... |
| 18 | {name} | {gemini/minimax/grok} | {strategy label} |

---

## Production Notes

- **Char count**: {N} chars ({pass/fail})
- **Element count**: {N} elements
- **Frequency map**: {frequency allocation}
- **BPM rationale**: {why this BPM}
- **Exclude rationale**: {why these exclusions}
- **W/SI justification**: {why these values}
```

---

## File Changes Summary

### New Files

| File | Description |
|------|-------------|
| `.claude/skills/create-track/SKILL.md` | Main skill definition — 5-phase orchestration instructions |
| `.claude/skills/create-track/workflow.md` | Node-by-node workflow for Tala sub-agent |
| `.claude/skills/create-track/assets/track-output-template.md` | Single-file output template |
| `.claude/skills/create-track-variant/SKILL.md` | Variant skill definition |
| `.claude/skills/create-track-variant/workflow.md` | Variant-specific workflow |

### Modified Files

| File | Change |
|------|--------|
| `.claude/agents/suno.md` | Decouple from `/suno` workflow reference, make generic for both new skills |
| `.claude/skills/suno/SKILL.md` | Add deprecation notice, set `user-invocable: false` |

### Unchanged Files

| File | Reason |
|------|--------|
| `.claude/souls/tala.md` | Soul file unchanged — Tala's identity doesn't change |
| `.claude/critics/*` | All four critics unchanged — same system prompts |
| `src/tools/*` | No tool changes — same CLI tools used |

---

## Tool Usage by Phase

| Phase | Tools Used |
|-------|-----------|
| P1 Analysis | `brain.ts --query-refs`, `discover --skill suno`, `charcount` (if needed) |
| P2 Review | None (Freddie presents) |
| P3 Generation | `charcount`, `gemini` (critic), `minimax_chat` (critic), `grok` (critic), `tracknames` |
| P4 Review | None (Freddie presents) |
| P5 Save | `getdate`, `reflect` |

---

## Migration

1. Build and test `/create-track` and `/create-track-variant`
2. Deprecate `/suno` (keep files, mark non-invocable)
3. Update CLAUDE.md team table to reference new skills
4. Existing tracks in old folder format remain valid — no migration needed for old output
