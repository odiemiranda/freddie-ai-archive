---
id: ASL-0026
project-code: ASL
title: Add tri-critic to /minimax-music + apply T4 pattern + --lyrics-only mode for validate-envelope
status: planned
priority: P2
created: 2026-04-09
depends-on: [ASL-0025]
related: [ASL-0025]
tags: [asl, skill, sol, minimax, critic, validate-envelope]
author: freddie
---

# ASL-0026 — Add tri-critic to /minimax-music + apply T4 pattern + lyrics-only envelope mode

## Why this exists

Two follow-ups that got deferred out of ASL-0025:

### 1. `/minimax-music` has no tri-critic

During ASL-0025 T5 (extending the authority reversal pattern to sibling skills), McCall's arch pass discovered that `.claude/skills/minimax-music/workflow.md` has 13 nodes and **zero critic nodes**. Sol (the MiniMax music agent) never runs critics at all.

This was a surprise — we assumed all four generation skills (`/create-track`, `/create-track-variant`, `/minimax-music`, `/lyrics`) had tri-critic, and planned to apply the T4 authority reversal pattern to all four. But there's nothing to reverse authority on in Sol's skill, because Sol doesn't defer to critics in the first place.

The user chose to DROP `/minimax-music` from ASL-0025 T5 scope and defer to this follow-up task. ASL-0025 shipped for `/create-track`, `/create-track-variant`, and `/lyrics` — three of the four.

### 2. `validate-envelope` can't validate lyrics without a Style block

McCall's T5 review caught a functional bug in the `/lyrics` skill's envelope gate integration (`ASL-0025 d37570f` lyrics/workflow.md node 008):

`validate-envelope.ts` hard-requires `--style` or `--style-file`. But `/lyrics` per node 001 accepts "Style block, emotional arc, genre direction, or existing track path" as input. Emotional arc and genre direction inputs produce **lyrics WITHOUT a Style block**. The envelope gate would have hard-failed every such run.

Ryan's T5 fix was a conditional skip in `lyrics/workflow.md` node 008: "if Style block exists → run gate; else → skip with logged note." This is correct short-term behavior but leaves a gap: **lyrics-only builds lose the canonical section markers check**, which is the one envelope-level check that would still apply (canonical `[Verse]`/`[Chorus]`/`[Bridge]` vs `[Section N:]` custom markers).

The fix is to add a `--lyrics-only` mode to `validate-envelope` that runs only the lyrics-dependent checks (canonical section markers, end marker, lyric content for vocal). Then `/lyrics` node 008 becomes unconditional: always run the gate, with `--lyrics-only` for no-Style-block builds.

## Goals

### Part A — Tri-critic for /minimax-music

1. **Decide whether Sol should have tri-critic at all.** This is not mechanical — it's a design question. MiniMax music generation may have different quality signals than Suno. Do Gemini/MiniMax/Grok critics add value here, or is Sol's pipeline different enough that tri-critic would be noise?
2. **If yes, which critic prompts?** Reuse existing (`production-musical.md`, `production-effectiveness.md`)? Write MiniMax-specific ones? A mix?
3. **Where in Sol's workflow do critics slot in?** Phase 3 generation has 13 nodes already — find the right injection point (likely after prompt construction, before finalization).
4. **Once tri-critic is in place, apply the T4 authority reversal pattern.** Same mechanics as /create-track and /lyrics: `reconcile-critics` CLI, structured output parsing, auto-apply compliance + apply quality with conflict detection + resolve from refs + return ORIGINAL+MODIFIED to user. New Phase 4 presentation with the 5-option menu.
5. **Update `.claude/souls/sol.md`** with the authority boundary section (same pattern as tala.md and rune.md got in ASL-0025).
6. **Update `.claude/agents/minimax.md`** (or whatever sol's agent file is) with the reconciliation protocol section.

### Part B — `--lyrics-only` mode for validate-envelope

1. **Add `--lyrics-only` flag** to `src/tools/validate-envelope.ts`. When set, the tool:
   - Does NOT require `--style` or `--style-file`
   - Runs only lyrics-dependent checks: `lyrics.canonical_section_markers`, `lyrics.end_marker`, `lyrics.has_lyric_content` (vocal only)
   - Skips all style.* checks without marking them as failures
2. **Update `/lyrics` node 008** in `lyrics/workflow.md` to be unconditional: always run the gate, use `--lyrics-only` for no-Style-block builds, use the full gate (with `--style`) for builds that produced a Style block.
3. **Add tests** to `src/tools/__tests__/validate-envelope.test.ts` covering:
   - `--lyrics-only` with canonical markers → pass
   - `--lyrics-only` with `[Section N:]` → fail on markers check
   - `--lyrics-only` without `--style` → no "style required" error (the whole point)
   - `--lyrics-only` with `--style` provided → either ignore the style or run full gate (pick one, document choice)
4. **Remove the "ASL-0026 TODO" comment** from `lyrics/workflow.md` node 008 once this ships.

## Investigation path

### Part A investigation

1. Read `.claude/skills/minimax-music/workflow.md` — understand Sol's current 13 nodes
2. Read `.claude/souls/sol.md` — understand Sol's current philosophy on quality (does Sol believe in critics?)
3. Read `.claude/agents/minimax.md` — understand Sol's current tool use
4. Compare with `.claude/skills/create-track/workflow.md` — what's the equivalent of P3-002 (build Style block) in Sol's flow?
5. Research: does MiniMax music have its own quality signals that tri-critic would add to, or duplicate?

### Part B investigation

1. Read `src/tools/validate-envelope.ts` — understand the current check structure
2. Find where style.* checks are defined and how to conditionally skip them
3. Confirm the existing check IDs (`lyrics.canonical_section_markers`, `lyrics.end_marker`, `lyrics.has_lyric_content`)
4. Check if there's any shared logic between `--type bgm` vs `--type vocal` vs `--lyrics-only` that would benefit from extraction

## Not in scope

- Adding critics to /minimax-art (Iris) or /minimax-video (Veo). Those skills have their own quality questions.
- Rewriting Sol's existing 13 nodes — only adding critic nodes (if the design decision in Part A step 1 says yes).
- Changing `validate-envelope`'s existing `--type bgm|vocal` modes. `--lyrics-only` is additive, not replacing.

## Dependency order

1. **Part B first (lyrics-only mode).** Small, bounded, directly fixes a known functional gap from ASL-0025 T5. 30-60 min.
2. **Part A second (minimax tri-critic).** Bigger scope, requires design decisions. 2-3 hours minimum, depending on what Sol gets.

Can ship independently but Part B is higher-leverage (closes an existing gap) and lower-risk (additive feature).

## Context

- Deferred from ASL-0025 (commits `428ab08`, `bdbd2f6`, `6ae6ea5`, `d37570f`)
- Lyrics gate workaround is in `lyrics/workflow.md` node 008 as conditional skip
- ASL-0025 task doc references this as a follow-up in the T5 Part B and Part C sections
