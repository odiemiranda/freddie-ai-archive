---
id: ASL-0020
title: Investigate MiniMax judge stability — frequent transport failures and degraded mode
project-code: ASL
status: planned
priority: P1
created: 2026-04-08
author: freddie
depends-on: []
blocks: []
related: [ASL-0001, ASL-0012]
tags: [task, asl, judge-panel, minimax, stability]
---

# ASL-0020 — MiniMax judge stability investigation

## Observed problem

Across all 39 staged proposals reviewed on 2026-04-08, **MiniMax was effectively carrying zero judge load**:

- **All 12 Penny proposals** ran in degraded mode — only gemini + grok alive, minimax dead from the start.
- **All 3 Sol proposals** ran in degraded mode — same pattern.
- **All 5 remaining non-degraded proposals** (echo, rune, tala) that reached a full 3-judge panel had minimax either returning `api_error` (transport failure, errorType: `api`) or returning a substantive verdict that was wrong enough to be overridden (see judge-override precedent at `.claude-personal/.../feedback_judge_override_precedent.md`).

In effect, the judge panel is operating as a **2-judge panel** (gemini + grok) today, with minimax contributing either noise or nothing. ASL-0001 (the defensive MiniMax parser fix from 2026-04-07) is working in the sense that failures no longer crash the pipeline, but it's only converting crashes into silent unavailability.

## Why this matters

The 3-judge panel's whole value proposition is that **unanimous approval is the bar** — any single judge can veto. When one judge is consistently absent, the bar effectively drops to "2 alive judges must both approve." Combined with the judge-override precedent from 2026-04-08 (lone dissent on refinements can be overridden by the user), the signal-to-noise of the panel is degrading.

Worse: because degraded mode still stages proposals that 2/2 alive judges approved, we may be accepting changes that a healthy minimax would have rejected. We can't know what we're missing.

## Goals

1. **Diagnose why MiniMax is failing.** Is it:
   - API rate-limited (hitting plan quotas)?
   - Model-level JSON parse failures (prompt length too long, response format drifting)?
   - Network transport failures (DNS, TLS, regional)?
   - Authentication (token expired)?
   - Prompt-length related (some proposals have bigger context than others)?

2. **Decide the stability strategy.** Options:
   - Fix the parse/API errors directly (if the root is known-solvable).
   - Add retry with backoff for transient failures.
   - Replace MiniMax with a more stable third model (Claude via API, DeepSeek, Qwen, etc.).
   - Keep minimax but adjust the panel logic to skip it on repeated failures and note the degradation clearly.

3. **Track minimax availability as a first-class metric.** Per-day uptime percentage, error type distribution, mean latency. Expose via `agent-state` or a new CLI.

## Investigation path

1. Read recent judge panel logs — check how many minimax calls succeeded vs failed in the last 24h. (Is there even a log? If not, that's the first gap.)
2. Read `src/libs/minimax.ts` — what error handling exists? Does it log structured errors or just return the parse-error string?
3. Read `src/libs/judge-panel.ts` — how does it handle minimax failures? Does it record them separately from substantive rejections?
4. Test minimax directly via its API with a minimal payload — confirm the baseline is even working today.
5. Compare success rate of minimax across workflows outside the judge panel (Sol music generation, etc.) — is this a judge-prompt-specific issue or a platform-wide issue?
6. Check if the failure pattern correlates with prompt length / input size. Judge prompts are typically longer than music generation prompts.

## Why P1

Every proposal going through the pipeline until this is fixed is being judged by an effectively 2-model panel. The longer this goes on, the more the drift. P1 because it affects the trust model of the whole self-learning loop, not because it's an outage.

## Not in scope

- Redesigning the judge panel architecture.
- Removing minimax permanently without data to justify.
- Adding a fourth judge.

## Context

- ASL-0001 shipped the defensive parser fix on 2026-04-07 — the fix prevented crashes but didn't fix root cause.
- ASL-0012 added the errorType column that made this problem visible. Without that column, minimax transport failures would have looked like substantive rejections.
- The 2 Echo proposals that went through the full panel were approved by user after this review — one was a pure api_error case, one was an "override minimax on refinement" case. Both recorded in `vault/studio/memory/echo/approved/`.
