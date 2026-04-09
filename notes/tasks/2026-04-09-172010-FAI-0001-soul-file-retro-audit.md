---
title: "Soul file retro-audit — remove TECHNIQUE contamination from pre-archive ASL approvals"
type: task
task-id: FAI-0001
status: open
priority: low
created: 2026-04-09
assignee: freddie
effort: 30-60 minutes
tags: [soul-files, cleanup, asl-archive-followup]
related-commit: 5b59473
---

# FAI-0001 — Soul file retro-audit

## Context

During the 2026-04-09 session that led to archiving ASL (commit `5b59473`), McCall hand-classified 8 recent approved soul changes against `.claude/rules/memory-boundaries.md`. Only **3 of 8** were unambiguous IDENTITY. The other 5 were borderline UNCERTAIN or outright TECHNIQUE.

This means the ASL pipeline — running for weeks before archival — quietly promoted TECHNIQUE-class content (generation rules, workflow mechanics, platform quirks) into soul files where it does not belong per `memory-boundaries.md`. The soul files need a cleanup pass.

## The concrete finding (from ASL-0027 task doc, Notes section)

| # | Change | Hand-label |
|---|--------|------------|
| 1 | Tala "core task goals always override critic suggestions / intent paramount" | IDENTITY ✅ |
| 2 | Rune "authenticity over rhyme fabrication, restructure rather than distort" | IDENTITY ✅ |
| 3 | Sol "mastering the room" | IDENTITY ✅ |
| 4 | Tala "3-5 rerolls are typical, prompt works if worst generation is recognizable" | UNCERTAIN / borderline |
| 5 | Tala "Subtle Intros `[Short Fade In]` trick" | UNCERTAIN → TECHNIQUE |
| 6 | Echo "Expanded Suno Critique Filter Rules" | IDENTITY-leaning → UNCERTAIN |
| 7 | Echo "Prioritized Suno Structural Critique" (seven-factor) | UNCERTAIN → TECHNIQUE |
| 8 | Rune "Build with positive constraints, max three instruments per section" | **TECHNIQUE** ❌ |

Case #8 is the clearest violation — a literal generation rule sitting in Rune's soul. The others need judgment calls.

## What to do

1. **Read each of the five soul files** in `.claude/souls/` that received updates from the ASL pipeline: `tala.md`, `rune.md`, `sol.md`, `echo.md`. (Also check `penny.md` and `mccall.md` — they had auto-applies in recent morning briefings too.)
2. **For each section or bullet added by the distill pipeline**, ask the memory-boundaries test:
   - **IDENTITY** — is this WHO the agent is (reflexes, judgment, beliefs, pet peeves, character)? → keep.
   - **TECHNIQUE** — is this WHAT the agent knows (generation rules, structural templates, platform mechanics, workflow recipes)? → move it to `vault/studio/memory/{agent}/knowledge/` as a new entry if it's not already there, then delete from soul.
   - **UNCERTAIN** — borderline? → leave it but tag it in a review comment, decide case-by-case.
3. **For the cases above specifically**:
   - Definitely remove #8 (Rune max-three-instruments) → move to `vault/studio/memory/rune/knowledge/instrument-density.md` or similar
   - Consider removing #5 (Tala Short Fade In trick) → move to Tala's knowledge as a signature-move technique
   - Consider removing #7 (Echo seven-factor critique) → move to Echo's knowledge as a methodology
   - #4 and #6 — judgment call; leave them unless a clean case emerges
4. **Git-commit each soul file change independently** with the rationale in the message, so future audits can see exactly what was removed and why.

## Acceptance criteria

- [ ] All 5 soul files inspected
- [ ] Rune #8 removed from soul, relocated to knowledge (or documented decision to keep)
- [ ] Each removal documented in its commit message with the memory-boundaries reasoning
- [ ] No mass edits — one soul file per commit, so rollback is clean

## Out of scope

- Automated classification (the ASL-0027 classifier that would have done this is cancelled)
- Cleaning up `memory/{agent}/knowledge/` files — those are supposed to contain TECHNIQUE content
- Rewriting `memory-boundaries.md` — separate concern if needed
- Recovering blacklisted rejected proposals from brain.db

## Notes

- This is low priority. The cleanup can happen in small chunks — one soul file at a time, whenever you're already touching that file for another reason.
- `git log -p .claude/souls/rune.md` and similar will show the history of what ASL added, making it easy to identify candidates.
- If a removal decision is hard, it's probably UNCERTAIN — leave it.
