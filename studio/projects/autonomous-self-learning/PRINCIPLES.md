---
id: asl-principles
title: ASL Pipeline — Architectural Principles
project-code: ASL
status: active
created: 2026-04-08
author: freddie
tags: [principles, architecture, asl, self-learning]
---

# ASL Pipeline — Architectural Principles

Principles discovered during the 2026-04-08 post-ASL-0017 review cleanup. These are not rules for code style — they are load-bearing architectural invariants that, when violated, produce the specific failure modes this project has already seen (proposal spam, silent auto-apply, rejected content regeneration, stacked soul duplicates).

## Principle 1 — The pipeline is a funnel of reductions

The self-learning pipeline is **a chain of reduction stages**, each producing less content than the previous one:

```
save (raw workflow output)
  ↓ reflect        [LLM: extract learnings from a session]
inbox items
  ↓ consolidate    [LLM: dedupe, merge, resolve contradictions]
knowledge entries
  ↓ distill        [LLM: identify identity-worthy patterns]
soul proposals
  ↓ gate + judge   [automated pass/fail + human review]
soul changes
```

Each arrow is an LLM-backed reduction. Each arrow's output is the next arrow's input. This is a funnel: the total volume of content decreases monotonically from top to bottom.

## Principle 2 — Consolidation is a mini-distill

The consolidate step and the distill step do the **same conceptual work at different levels of abstraction**:

- **Consolidate:** inbox items → knowledge entries. Dedupe, merge, resolve contradictions. The input is a list of observations from recent sessions; the output is cleaned up working memory.
- **Distill:** knowledge entries → soul proposals. Identify identity-worthy patterns, propose rule changes. The input is the agent's accumulated clean working memory; the output is proposed identity updates.

Both steps are distillations. Both use an LLM to synthesize content. Both have dedupe and contradiction-resolution concerns. The difference is only the input domain and the output schema.

**Implication:** Any future work that improves consolidation quality is equivalent to improving a first-stage mini-distill. Lessons learned about prompt design, evidence weighting, contradiction handling in distill apply to consolidate and vice-versa. If the two steps start diverging in quality of output, that's a signal something is wrong at the shared conceptual layer.

## Principle 3 — Each reduction stage fires ONLY on meaningful upstream delta

**Every stage must only fire when its immediate upstream produced real new output.** "Real new output" means content has materially changed — not file mtimes, not cache keys, not pipeline hash nonsense.

Concretely:
- **Reflect fires only on save events** (raw workflow output was written). If no save happened, nothing to reflect on.
- **Consolidate fires only when inbox has new or changed items.** If inbox is empty or has no unprocessed delta since last consolidation, skip.
- **Distill fires only when consolidation produced new/changed knowledge content.** If consolidation ran on a dupe inbox item and merged it into existing knowledge without changing the knowledge content, distill should skip.
- **Gate + judge fires only when distill proposed something.** Already correct today.

The original design violated this at the distill stage: distill's trigger was "knowledge hash changed or cache miss," which fires on any touch anywhere in the pipeline — including touches that don't change content substance. That's how Sol and Tala generated 14 and 2 proposals respectively from **empty inboxes** during the 2026-04-08 sync: something touched their knowledge files, hash changed, distill fired, re-synthesized existing content.

The fix: **stage-delta content hashes stored in `agent_lifecycle` (brain.db table from ASL-0002).** Compare against the previous stored hash. If identical, skip.

## Principle 4 — Degraded modes must fail CLOSED, not OPEN

When the system operates below its designed redundancy (e.g., a judge is dead, a gate can't run, an embedding model is unavailable), the default behavior must be **more cautious**, not less.

The original design violated this in the judge panel's degraded quorum path: when minimax is dead, the code auto-applies on "all alive judges approve" (2/2) instead of escalating to human review. This is exactly the opposite of the intended safety net.

**The rule:** if you can't tell whether a change is safe because part of the check machinery is broken, don't apply the change — stage it and tell a human. Silent permissive fallbacks are a root-cause of the 2026-04-08 Penny auto-apply incident.

## Principle 5 — Every automated write to identity files MUST leave an audit trail

Soul files are identity. Any automated modification to `.claude/souls/*.md` must be accompanied by a durable, queryable record of:
- What changed (the diff)
- Why (the proposal, evidence, gate results, judge votes)
- When (timestamp)
- Under what authority (auto-applied vs human-approved vs degraded-approved)

The auto-applied Penny proposals on 2026-04-08 left **no audit trail** — no entry in any `approved/` directory, no record in brain.db visible to `review-staged list`, no way to reconstruct from the filesystem what had just happened. The only evidence was a modified working tree in git.

**The rule:** no soul modification without a matching persistent record. If the record can't be written (disk full, permissions, whatever), the modification must fail and log loudly — not proceed silently.

## Principle 6 — Rejection is information that must flow upstream

When a proposal is rejected, that rejection is information about what the distill/gate/judge pipeline should NOT produce in the future. Today, rejection is a one-shot block: the proposal goes to `rejected/`, and nothing about the rejection feeds back into distill's next invocation. The same content gets re-proposed on the next run.

This is a leak in the funnel — the reduction becomes reversible. Content we decided to exclude can re-enter the pipeline without anyone noticing.

**The rule:** distill must be aware of what has been rejected in the past and skip content that would be substantively similar to a prior rejection. Implementation options range from "simple keyword blacklist from rejected titles" to "vector similarity check against rejected proposal embeddings." The lightweight version is enough for now; the full version is a future refinement.

## How these principles were discovered

All six principles surfaced during the review queue cleanup on 2026-04-08, when the ASL pipeline had been running autonomously for one day post-ASL-0017 shipment. Observing 39+ proposals in the queue, most of them structural rejections or content regenerated from rejected inputs, exposed the funnel violation (Principle 3). Seeing Penny's soul modified with no audit trail exposed Principle 5. Seeing minimax dead but proposals still auto-applying exposed Principle 4. The "consolidation is a mini-distill" insight (Principle 2) came from the user noticing that the pipeline had two synthesis stages doing similar work.

These are not hypothetical principles. They are tombstones for specific bugs that the pipeline produced in the first 24 hours of autonomous operation. Any future change to the ASL codebase should be evaluated against all six.
