---
title: Consolidation tool token limit caused silent knowledge loss
scope: shared
source: session-20260326
confidence: high
---

## Discovery: maxOutputTokens Too Low Caused Silent Data Loss

The consolidation tool's Gemini call had maxOutputTokens set to 8192. For Tala's consolidation (17 input files → 4 planned output files), the plan section consumed most of the budget. Only 1 of 4 planned knowledge files was actually written. The other 3 were silently lost.

### Fix applied:
1. maxOutputTokens increased to 32768 (4x headroom)
2. Post-parse check: compares planned file count to parsed file count, warns if mismatch
3. Post-write check: verifies knowledge dir matches plan

### Lesson:
When using LLMs for structured multi-file output, always verify the output count matches the plan. Token limits cause silent truncation — the model doesn't error, it just stops mid-output.
