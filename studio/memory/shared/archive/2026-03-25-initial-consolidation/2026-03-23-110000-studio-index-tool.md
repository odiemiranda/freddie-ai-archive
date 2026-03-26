---
id: mem-shared-20260323-110000-01
title: Studio Index Tool Design
created: 2026-03-23 11:00:00
tags: [tooling, tracks, lyrics, search, index, performance]
type: decision
agent: shared
connected: []
---

Design for a studio search tool (`tools/studio.ts`) that indexes tracks and lyrics for fast agent lookup. Agents currently traverse folders file-by-file — an index makes it instant.

**Index schema per entry:**
- file, type (track-index/track-variation/lyrics-index/lyrics-variation), title, parent
- created, platform, genre[], mood[], instruments[], bpm, key, tags[]
- summary (from variation table or first line)
- body_keywords (extracted signal words from Style block, production notes — not full content)

**Three modes:**
1. Search: keyword match against tags + summary + body_keywords, returns file paths + summaries
2. List: catalog by type, recent, genre filter
3. Detail: return full frontmatter as JSON for one file

**Save flow:** `bun tools/studio.ts --save {path}` after writing track/lyrics files (same pattern as remember.ts --save). Bootstrap rebuilds all.

**Output files:** `vault/studio/tracks-index.json` + `vault/studio/lyrics-index.json`

**Why:** Tala and Rune traverse folders during workflows (linking lyrics to tracks, checking for duplicates, referencing past work). Index lookup is instant vs file-by-file reads.

**How to apply:** Build as next-session task. Follow remember.ts patterns. Add --save call to agent write steps.
