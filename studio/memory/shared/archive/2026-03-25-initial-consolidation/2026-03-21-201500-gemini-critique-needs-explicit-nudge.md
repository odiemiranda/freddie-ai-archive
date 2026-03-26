---
id: mem-shared-20260321-201500-01
title: Gemini Critique Needs Explicit Nudge
created: 2026-03-21 20:15:00
tags: [gemini, critique-loop, agent-spawning, workflow]
type: feedback
agent: shared
connected: []
---

When spawning Tala or Rune, explicitly remind them to run step 8b (Gemini critique loop) in the spawn prompt. Both agents have the step documented in their agent files but skip it at runtime unless the spawn prompt specifically says to execute it.

**Why:** First test of the critique loop — both Tala and Rune skipped step 8b entirely during generation. The step had to be run manually after the fact. The agent files define the loop correctly but agents don't reliably self-execute multi-pass external tool calls without explicit instruction.

**How to apply:** When spawning either agent, include a line like: "After building each Style block, run the Gemini critique loop (step 8b)" or "After your critic pass, run the Gemini critique loop (step 8b) for each variation." This applies to any future agent that has a Gemini review step.
