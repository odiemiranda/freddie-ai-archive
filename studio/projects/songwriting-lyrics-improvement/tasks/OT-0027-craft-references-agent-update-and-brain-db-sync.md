---
title: "Craft References, Agent Update, and brain.db Sync"
type: task
project: songwriting-lyrics-improvement
status: ready
created: 2026-04-03
author: mccall
architect: "vault/studio/projects/songwriting-lyrics-improvement/ARCHITECT.md"
group: 8
parallel: true
depends_on: ["OT-0022"]
---

# OT-0027: Craft References, Agent Update, and brain.db Sync

## What to Build

Create 5 craft reference files for elite lyricist archetypes, update the Rune agent
file to reference them, and sync to brain.db.

### Part A: Craft Reference Files

Create 5 files under `vault/studio/references/lyrics/craft/`:

1. **`architect.md`** -- The Song Architect
   - Masters: Kendrick Lamar, Joni Mitchell, Radiohead
   - Techniques: structural experimentation, section subversion, multi-voice POV, non-linear narrative
   - When to apply: concept tracks, prog, experimental, art-pop
   - Anti-patterns: structure for structure's sake, complexity without emotional payoff

2. **`flow-master.md`** -- The Flow Master
   - Masters: Eminem, Missy Elliott, Andre 3000
   - Techniques: internal rhyme density, enjambment, syllabic runs, rhythmic variation, cross-line flow
   - When to apply: hip-hop, rap, R&B, high-energy pop
   - Anti-patterns: flow over meaning, rhyme tax, filler syllables for meter

3. **`cinematographer.md`** -- The Cinematographer
   - Masters: Leonard Cohen, Nick Cave, Taylor Swift
   - Techniques: scene-setting opening, camera-movement through spaces, sensory anchoring, specific objects as emotional proxies
   - When to apply: ballads, storytelling, country, folk, singer-songwriter
   - Anti-patterns: telling emotion directly, generic imagery ("the stars", "the ocean")

4. **`painter.md`** -- The Sound Painter
   - Masters: Bjork, FKA Twigs, Cocteau Twins
   - Techniques: vowel color selection, consonant texture as rhythm, phonesthetic clusters, synesthetic language
   - When to apply: ambient, dream-pop, shoegaze, electronic, experimental
   - Anti-patterns: prioritizing sound over meaning, "word salad" without anchoring

5. **`compressor.md`** -- The Compressor
   - Masters: Billie Eilish, Frank Ocean, Elliott Smith
   - Techniques: economy of words, white space as instrument, implication over statement, one-image-per-line
   - When to apply: lo-fi, minimal pop, R&B, acoustic, singer-songwriter
   - Anti-patterns: over-explaining, adjective stacking, "more words = more emotion"

**Each file format:**
```markdown
---
title: "{Archetype Name}"
type: reference
category: lyrics-craft
tags: [lyrics, craft, {archetype-slug}]
---

# {Archetype Name}

## Masters Studied
...

## Key Techniques
### Technique 1: {Name}
**What it is:** ...
**Example:** ...
**When to apply:** ...

## Anti-Patterns
...

## Genre/Mood Context
| Genre | Relevance | Notes |
...
```

### Part B: Agent File Update

Update `.claude/agents/lyrics.md` to add craft reference awareness:

Add a section (after "Your instincts" or appropriate location):
```markdown
## Craft Layers

Query brain.db for relevant craft techniques based on genre and mood:
`bun run tool brain --get-context "lyrics craft {genre} {mood}" --agent rune --stores refs --limit 3`

Five craft archetypes available in `vault/studio/references/lyrics/craft/`:
architect, flow-master, cinematographer, painter, compressor.

Loaded on-demand per track. Never bulk-load all five. Pick 1-2 that match the genre/mood.
These are technique references, not identity. Your voice stays yours.
```

### Part C: brain.db Sync

Run brain.db sync to index the new craft reference files:
```bash
bun run tool brain --check
```

The existing sync infrastructure will pick up new files in `vault/studio/references/lyrics/craft/`.
Verify they appear in search results.

## Acceptance Criteria

- [ ] AC-1: Five craft reference files created with proper frontmatter
- [ ] AC-2: Each file contains: masters, techniques with examples, when to apply, anti-patterns
- [ ] AC-3: Files indexed in brain.db after sync
- [ ] AC-4: `bun run tool brain --search-refs "lyrics craft architect"` returns the architect reference
- [ ] AC-5: Agent file updated with craft layer query instructions
- [ ] AC-6: Rune's soul file (`souls/rune.md`) UNCHANGED
- [ ] AC-7: Craft references loaded on-demand, not at startup

Maps to PRD: US-14 (AC-14.1 through AC-14.6)

## Files in Scope

- `vault/studio/references/lyrics/craft/architect.md` -- **create**
- `vault/studio/references/lyrics/craft/flow-master.md` -- **create**
- `vault/studio/references/lyrics/craft/cinematographer.md` -- **create**
- `vault/studio/references/lyrics/craft/painter.md` -- **create**
- `vault/studio/references/lyrics/craft/compressor.md` -- **create**
- `.claude/agents/lyrics.md` -- **modify** (add craft layers section)
- `.claude/souls/rune.md` -- **DO NOT MODIFY** (read-only, constraint C9)

## Architectural Constraints

- Craft references are technique material, NOT identity. They supplement Rune's soul, not replace
- On-demand loading from brain.db per constraint C8 (context window efficiency)
- Frontmatter tags must be consistent for brain.db discovery
- Each file should be 200-400 lines. Detailed enough to be useful, concise enough for context windows

## Out of Scope

- Implementing craft-aware generation logic (that's Rune's responsibility at runtime)
- Modifying Rune's soul file
- Building new brain.db infrastructure (existing sync handles it)

## Test Plan

```bash
# Verify files exist
ls vault/studio/references/lyrics/craft/

# Sync to brain.db
bun run tool brain --check

# Search for craft references
bun run tool brain --search-refs "lyrics craft architect structural"
bun run tool brain --search-refs "lyrics craft flow internal rhyme"
bun run tool brain --search-refs "lyrics craft cinematographer sensory"
```

## Notes

- These are content-authoring tasks, not code. The craft descriptions should draw from real songwriting analysis. Quality of examples matters -- use specific, recognizable technique demonstrations.
- The 5 archetypes map to the PRD's "5 elite lyricist layers" from the research findings. They cover the spectrum from structural (architect) to minimal (compressor).