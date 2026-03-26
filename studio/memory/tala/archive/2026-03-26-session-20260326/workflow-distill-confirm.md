---
name: workflow-distill-confirm
description: Mandatory interaction cycle for Suno track generation to ensure prompt quality.
type: knowledge
agent: tala
tags: [workflow, confirmation, distill-confirm]
---

# Distill → Confirm Cycle

## Overview
Never build a track without explicit user confirmation. Every track generation must complete the full Phase 1 (Distill & Adjust) before entering Phase 2 (Build).

## Why
Skipping the interactive distill step leads to over-stuffed prompts and muddled output. Suno requires high-signal, low-noise inputs. The "Distill" step allows the user to prune ingredients, fix genre collisions, and ensure harmony before files are created.

## Success Metrics
- **Validated by "First Sip" (2026-03-25):** Full workflow produced a production-quality track on the first generation by catching a genre mismatch and BPM error during the Distill phase.

## How to Apply
1. **Phase 1 (Distill):** Present the ingredient breakdown (genres, instruments, mood, BPM/Key).
2. **Phase 1 (Adjust):** Ask the user for confirmation or adjustments.
3. **Phase 2 (Build):** Only once explicit approval is received, generate the Style and Lyrics blocks and initiate the build.

**Consolidated from:** 2026-03-21-011500-no-build-without-confirmation.md, 2026-03-25-validated-formula-confirmed.md
