---
id: OT-0003
title: Build character count utility for Style blocks
status: pending
created: 2026-03-26
tags: [tools, suno, validation]
---

# OT-0003: Character Count Utility

## Goal
LLMs cannot accurately count characters. Build a tool that validates Style block character counts before writing to disk.

## Source
nwp/suno-song-creator-plugin `utils/count-prompt.js` / `count-prompt.py`

## Scope
- Add char counting to existing validation or as a standalone tool
- Could integrate into the compliance gate (step 8k)
- Should count: total chars, per-slot chars, warn if over budget
- Consider: CLI tool (`bun src/tools/charcount.ts`) or inline validation in the agent workflow

## Sub-tasks
- [ ] Decide: standalone tool vs inline validation
- [ ] Implement
- [ ] Wire into compliance gate
- [ ] Test with known Style blocks
