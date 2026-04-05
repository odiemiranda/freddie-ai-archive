---
title: "Tech Approach: NotebookLM Research Agent"
date: 2026-03-29
status: planning
tags: [research, notebooklm, agent, architecture]
---

# Tech Approach: NotebookLM Research Agent

## Goal

Build a researcher agent/skill that uses Google NotebookLM (via `notebooklm-py`) as a **source-grounded research and distillation layer**. Instead of raw document lookup, we get synthesized, citation-backed answers from curated source collections.

## Why

- Current workflow: discover.ts searches reference files by keyword — returns raw matches, no synthesis
- NotebookLM adds: upload sources once, query many times, get **distilled answers grounded in those sources**
- Use case: "What makes shamisen jazz fusion work?" returns a synthesized answer from genre guides, production docs, and music theory — not a list of file matches

## Library: notebooklm-py

- **Repo:** https://github.com/teng-lin/notebooklm-py
- **PyPI:** `pip install notebooklm-py` (v0.3.4+)
- **Browser auth:** `pip install "notebooklm-py[browser]"` + `playwright install chromium`
- **How it works:** Undocumented Google APIs (not official — can break)
- **Has:** CLAUDE.md, built-in Claude Code skill support

### Capabilities

| Feature | Supported | Notes |
|---------|-----------|-------|
| Create notebooks | Yes | Programmatic notebook creation |
| Upload sources | Yes | Text, URLs, PDFs, docs |
| Query notebook | Yes | Source-grounded Q&A with citations |
| Audio overviews | Yes | Podcast-style summaries |
| List notebooks | Yes | Library management |
| Delete/manage | Yes | Full CRUD |

### Risk

**Undocumented APIs.** Google can break this without notice. Acceptable for personal toolkit, not for production. Mitigation: treat as enrichment layer, not core dependency. All outputs get saved to vault as reference files — if NotebookLM breaks, we still have the distilled knowledge.

## Architecture

### Notebook Strategy

Organize NotebookLM notebooks by research domain:

| Notebook | Sources | Purpose |
|----------|---------|---------|
| `suno-production` | Suno guides, platform docs, style references | Suno-specific production queries |
| `genre-fusion` | Genre guides, music theory, fusion technique docs | Genre blending research |
| `instrument-techniques` | Instrument reference cards, playing techniques | Instrument-specific knowledge |
| `music-theory` | Theory docs, chord progressions, scale guides | Theory grounding |

### Integration Points

```
User question
    |
    v
/research skill
    |
    ├── 1. Select notebook (by domain)
    ├── 2. Query via notebooklm-py
    ├── 3. Get source-grounded answer + citations
    ├── 4. Present to user
    └── 5. Optionally save as reference card in vault
```

### Where it fits in existing workflows

| Workflow | Integration |
|----------|-------------|
| `/create-music-card` | Research genre context before/after card creation |
| `/create-track` | Deep research on fusion techniques during P1 analysis |
| `/lyrics` | Genre-specific lyrical conventions and structures |
| Manual research | Ad-hoc "what works for X?" questions |

### Proposed Skill: `/research`

**Input fields:**
1. **Question** — what to research (required)
2. **Notebook** — which domain notebook to query (auto-select or user choice)
3. **Save** — save result as vault reference card? (yes/no, default: no)

**Workflow:**
1. Parse question, select notebook
2. Query NotebookLM via `notebooklm-py`
3. Present distilled answer with source citations
4. User reviews — adjust query, save to vault, or done

**Output:** Distilled answer in chat. Optionally saved as `vault/studio/references/shared/research/<slug>.md` with frontmatter, citations, and source notebook reference.

## Setup Requirements

1. Install: `pip install "notebooklm-py[browser]"` + `playwright install chromium`
2. Authenticate: browser-based Google login (one-time, persistent)
3. Create initial notebooks and upload source collections
4. Build skill file + workflow
5. Wire CLI wrapper in `src/tools/`

## Open Questions

- **Source management:** How to keep notebook sources in sync with vault references? Manual upload or automated?
- **Caching:** Should we cache NotebookLM responses locally to avoid repeated queries?
- **Notebook granularity:** Fewer large notebooks (broad context) vs. many small ones (precise but requires routing)?
- **Audio overviews:** Worth generating for learning? Or just text Q&A?
- **Fallback:** If notebooklm-py breaks, fall back to Gemini direct with same sources?

## Next Steps

1. Install notebooklm-py, test auth flow on Windows
2. Create a test notebook, upload 2-3 reference files
3. Query programmatically, evaluate answer quality
4. If quality is good: build `/research` skill
5. If quality is mediocre: stick with Gemini direct + discover.ts
