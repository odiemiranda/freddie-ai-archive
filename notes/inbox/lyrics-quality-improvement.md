---
title: Lyrics Quality Improvement — Intentionality Over Rhyme
created: 2026-03-25
tags: [idea, lyrics, rune, quality, minimax, gemini, critique, craft]
status: in-progress
---

## Problem

Lyrics sometimes use words that exist to serve the rhyme or meter rather than because the scene demands them. The result technically scans but feels hollow — the listener can sense that a word was chosen for sound, not meaning. Rune's self-critic (step 8b) already checks for "rhyme tax" but writer and critic are the same brain — blind spots persist.

## Goal

Every word in a lyric should be **intentional** — there because the scene, emotion, or character needs it. Rhyme and meter serve the meaning, not the other way around. When they conflict, meaning wins.

## Three Improvements (in priority order)

### 1. MiniMax as Second Lyrics Critic

**Why:** Different models have different blind spots. Gemini catches structural issues (volta, density, cliché). MiniMax may catch intentionality issues — words that exist only for rhyme, lines that say nothing specific, vocabulary mismatches.

**How:**
- New Gemini-style critic prompt tuned for MiniMax M2.7
- Runs **after** Gemini's critique as a parallel second opinion
- Focused specifically on **intentionality**: purpose of every word, rhyme tax detection, register consistency
- Cheap: $0.30/$1.20 per 1M tokens, one call per variation

**Tiebreaker rule:** If both critics flag the same line → must rewrite. If only one flags → Rune decides. Neither critic can wholesale rewrite — they suggest, Rune writes.

**Implementation:**
- Create `.minimax/agents/lyrics_critic.md` — MiniMax-specific lyrics critic prompt
- Add step 8c2 to `lyrics.md` — MiniMax critique pass after Gemini pass
- Use `libs/minimax.ts` `chatCompletion()` via `tools/minimax/chat.ts`

### 2. Intentionality Pass (Stronger Self-Critic)

**Why:** Even with two external critics, Rune's own internal check should be sharper. Currently step 8b asks "was this line warped to fit a rhyme?" but doesn't systematically test every word.

**How:** Add a focused sub-step to the self-critic (8b):
- **Word-level audit:** For every noun and adjective in a stanza, ask: "Is this word here because the scene needs it, or because the meter needed a syllable?" If removing the word doesn't break the scene, it's filler.
- **Rhyme removal test:** For every rhyming pair, mentally remove the rhyme constraint. Would the line say the same thing? If not, the rhyme is writing the lyric.
- **The Replacement Test:** For any flagged word, find the word the scene actually wants. If the replacement breaks the rhyme, keep the replacement and fix the rhyme scheme around it. Meaning > sound.

**Implementation:**
- Add "Intentionality Audit" sub-bullets to step 8b in `lyrics.md`
- Update Rune's soul with the principle (after it's proven via reflection)

### 3. Craft Studies Reference (Future)

**Why:** Example lyrics can help — not as vocabulary sources, but as **architectural blueprints** showing how great songwriters handle rhyme-vs-meaning tradeoffs.

**How:**
- Create `vault/studio/references/lyrics/craft-studies/` folder
- Annotated examples from different genres
- Each study focuses on a specific craft question:
  - How to handle a chorus that needs to rhyme but also say something specific
  - How to land the bridge turn without cliché
  - Where to break rhyme scheme for a more important word
  - How to use slant rhyme when perfect rhyme costs too much meaning
- Discoverable via `discover.ts` — agents can reference during generation

**Deferred:** Most labor-intensive. Build after improvements #1 and #2 prove what the critics actually catch, so we know which craft questions to study.

## Success Criteria

- Lyrics where every word passes the "why is this word here?" test
- Rhyme schemes that serve meaning, not the reverse
- Measurable: fewer user adjustments for word choice in post-generation review
- Two-critic consensus catches more issues than single-critic alone

## Open Questions

- [ ] Should the MiniMax critic run on every variation or only on request?
- [ ] How do we prevent critique fatigue — too many passes making the workflow slow?
- [ ] Should craft studies be genre-specific or technique-specific?
- [ ] Could we A/B test: same prompt through Gemini-only vs Gemini+MiniMax pipeline?
