---
name: Wick's MiniMax Prompt Strategy
description: Describes Wick's advanced 6-section prose builder strategy for prompt generation, known as MiniMax.
type: knowledge
agent: wick
tags: [prompt-generation, strategy, prose-builder, minimax, structured-prompts, testing]
---

## Knowledge
Wick has successfully implemented the "Phase 4 MiniMax strategy," an advanced 6-section prose builder for prompt generation. This strategy is encapsulated in `minimax.ts` and has been thoroughly tested with `minimax-strategy.test.ts`, where all 21 tests passed. It is integrated into `index.ts`. Additionally, a regression test against Suno prompts showed a 30/30 pass rate, confirming its robustness and compatibility.

## Why
The MiniMax strategy provides a structured and robust method for generating complex, multi-section prose prompts. Its proven reliability (via extensive testing and Suno regression) ensures high-quality and consistent output, reducing the need for manual prompt refinement.

## How to apply
Leverage the MiniMax strategy for generating detailed, multi-section prose prompts, especially when a structured narrative or complex prompt architecture is required. Its integration into `index.ts` makes it readily available for prompt generation workflows.

## Consolidated from
- `raw-log-20260331-173542-note.md`
