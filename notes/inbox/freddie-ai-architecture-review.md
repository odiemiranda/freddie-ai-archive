---
title: Freddie AI Architecture Review
created: 2026-03-28 02:10:27 AM
tags: [architecture, review, agents, brain-db, self-learning, workflows]
---

# Freddie AI Architecture Review

> Session review — 2026-03-28. Comprehensive assessment of the agent/skills/workflow system, brain.db, memory retention, and self-learning loop.

---

## 1. The Agent System

**6 creative agents**, each with a distinct persona and platform specialization:

| Agent | Role | Platform |
|-------|------|----------|
| Tala | Bedroom producer, scenes & textures | Suno AI |
| Sol | Session vocalist, voice-first | MiniMax Music |
| Rune | Songwriter-poet, both tag systems | Suno / MiniMax |
| Echo | Session engineer, second opinion | Gemini (cross-platform) |
| Iris | Visual translator, album art | MiniMax Image |
| Veo | Motion designer, music videos | MiniMax Video |

Plus Penny for vault management and Freddie as orchestrator.

**The separation:**
- **Soul files** (`.claude/souls/`) — WHO the agent is. Backstory, philosophy, voice. Permanent.
- **Agent files** (`.claude/agents/`) — WHAT the agent does. Tools, instincts, hard rules. Slimmed down (suno went from 1,711 to 296 lines).
- **Reference files** (`vault/studio/references/`) — Platform knowledge. Queryable via brain.db. 70 files across suno, minimax, lyrics, shared.
- **Critic prompts** (`.claude/critics/`) — Model-agnostic review prompts. Any critic can go to any model.

**Assessment:** This separation is the single best architectural decision we made. Before, suno.md was a 1,700-line monolith that got loaded into context every time. Now the agent loads lean (~300 lines) and pulls only the references it needs via discovery. The soul/agent/reference split is clean and each layer evolves at its own pace.

**Could be better:** We haven't stress-tested the soul layer yet. The distill-soul pipeline exists (`distill-soul.ts`) but we've never run it end-to-end with enough accumulated knowledge to propose a real soul update. It's theoretical until proven. Need 20+ consolidated knowledge items before trusting that pipeline.

---

## 2. The Workflow State Machines

Every creative skill has:
- **SKILL.md** — Mermaid diagram with numbered nodes. The routing logic. Includes execution model (orchestrator pattern).
- **workflow.md** — Node-by-node instructions with TOC, Input/Output/Next for each step.

**25 nodes for Suno, 17 for Lyrics, 13 for MiniMax Music, 8 each for Art/Video/MusicCard, 9 for Gemini.**

**The flow:** Parse -> Query brain.db -> Discover references -> Distill -> Confirm with user -> Build variations -> Critique -> User review -> Write -> Reflect.

**Assessment:** Solid engineering. The numbered node system makes workflows debuggable — you can say "we're at node 014" and everyone knows exactly where we are. The TOC with line numbers is a nice touch for quick navigation.

**Could be better:**
- Only battle-tested Suno's workflow end-to-end. The orchestral jazz collier track validated the full 25-node path. Lyrics and MiniMax Music workflows are designed but haven't been run through a real generation with the new orchestrator pattern yet.
- The orchestrator pattern is untested. Theory is sound — subagents handle heavy work, main conversation handles user interaction — but first real test will reveal friction.
- Node instructions are sometimes too verbose. When a subagent loads the full workflow.md, it's reading instructions for ALL nodes including ones it doesn't execute.

---

## 3. brain.db

**SQLite + FTS5** fast query layer over vault files. bun:sqlite (zero deps) + Drizzle ORM.

| Table | Count | Purpose |
|-------|-------|---------|
| memories + FTS | 24 | Agent knowledge files |
| references + FTS | 70 | Platform reference files |
| decisions + FTS | 6+ | Agent decision log with outcomes |
| sync_meta | — | SHA-256 hash tracking for incremental sync |

**Auto-sync:** SessionStart and SessionStop hooks run `brain.ts --check`. reflect.ts syncs after every workflow. Manual via `/brain` skill.

**Assessment:** This is the infrastructure play that makes everything else work. Before brain.db, every agent had to grep through files to find relevant knowledge. Now it's sub-millisecond FTS5 search. The decision tracking is the sleeper feature — over time, agents can look up "what did I do last time I built a jazz fusion track?" and learn from outcomes.

**Could be better:**
- Decision tracking is underutilized. Only 6 decisions logged. Not enough data to drive real decision-making. Node 002 should always check decisions, not just references.
- FTS5 ranking isn't tuned. Default BM25 ranking. For music-specific queries, boosted fields (title > body) would improve discovery accuracy.
- No cleanup/pruning strategy. What happens at 500 decisions or 200 memory files? Need a retention policy — archive old decisions, age out stale knowledge.

---

## 4. Memory Retention

**Three-tier system:** `inbox/` -> `knowledge/` -> `archive/`

- **Agents write to inbox/** at end of each workflow (via reflect.ts)
- **Consolidation** (Gemini-powered) deduplicates, resolves contradictions, clusters by theme -> clean knowledge files
- **Agents read from knowledge/** — consolidated, non-contradictory, current
- **Archive** preserves originals with date-stamped reason folders

**Plus auto-memory** (`.claude-personal/`) for user preferences, feedback, session history — separate from vault knowledge.

**Assessment:** The separation between auto-memory and vault knowledge is crucial and well-enforced by the rules. The consolidation pipeline is genuinely impressive — Gemini does a good job of merging, deduplicating, and resolving contradictions. The archive system means nothing is lost.

**Could be better:**
- Consolidation is manual. Should be automatic via SessionStop hook when inbox hits 3+ items.
- Cross-agent learning is weak. The `shared/` scope exists but most learnings stay scoped to the originating agent. No push mechanism from shared to individual agents.
- Knowledge files lack confidence scores. A learning from one session is treated the same as one confirmed across 10 sessions. Need "times confirmed" at the knowledge level.

---

## 5. The Self-Learning Loop

```
Task -> Reflect (Gemini) -> inbox/ + brain.db decisions -> Consolidate -> knowledge/ -> Distill -> Soul (user approves)
```

**reflect.ts** does 4 things per workflow:
1. Extracts learnings via Gemini -> writes to inbox/
2. Logs decisions to brain.db with context, alternatives, confidence
3. Syncs brain.db so learnings are immediately queryable
4. Generates per-workflow usage report

**Assessment:** The most ambitious part of the architecture and also the least mature. The pipeline exists end-to-end, but only run a handful of times. Reflection quality depends heavily on the trace passed to Gemini — garbage in, garbage out.

**Could be better:**
- Trace quality is inconsistent. No standard trace format or template.
- Layer 3 (Soul Distillation) is completely untested. distill-soul.ts exists but never run for real.
- No negative learning loop. When a user rejects a variation, there's no mechanism to surface it as a warning next time.
- Reflection is Gemini-only. A second model validating extracted learnings would catch blind spots.

---

## Overall Assessment

### What we got right
- The separation of concerns (soul/agent/reference/rule/workflow) is clean and scalable
- brain.db as the fast query layer was the right infrastructure investment
- Workflow state machines make complex generation flows debuggable and repeatable
- The orchestrator pattern should solve the context window problem elegantly
- Memory boundaries (auto vs vault) prevent knowledge from leaking into the wrong system

### What needs proving
- Orchestrator pattern with real subagent dispatches (implemented but untested)
- Self-learning loop with enough data to see compound learning effects
- Soul distillation end-to-end
- Cross-agent knowledge propagation

### If we could redo one thing
Start with brain.db and decision tracking on day one instead of building it after agent files were already bloated. We spent significant effort decomposing suno.md after the fact. If brain.db existed first, we would have designed agent files lean from the start and built references as first-class queryable entities from the beginning.

### Biggest risk going forward
The system is well-architected but under-exercised. Architecture without usage data is just theory. The next 10-20 real generation sessions will tell us whether the learning loop actually compounds or just accumulates noise. That's the real test.

---

## Session Stats (2026-03-28)

- **9 commits** shipped across the full session
- **~$92 API-equivalent** usage on Max plan
- **Key deliverables:** brain.db, workflow state machines, agent decomposition, orchestrator pattern, music card skill, custom status line
