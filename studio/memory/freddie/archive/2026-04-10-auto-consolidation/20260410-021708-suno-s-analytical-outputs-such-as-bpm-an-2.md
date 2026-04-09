---
source: reflection
agent: freddie
project: freddie-ai
projectPath: "D:\\PROJECTS\\freddie-ai"
task: "FAI-0003 + FAI-0004 shipped"
workflow: dev-start
outcome: success
scope: shared
confidence: high
memoryType: entity
importance: 10
tags: [suno, platform_behavior, non_determinism, data_integrity, evaluation, drift_study, audio_processing, reliability]
session: 2026-04-10T02:17:08
---

## Observation
Suno's API exhibited non-deterministic behavior, returning different BPM readings (72 vs 85) and display_tags for the same audio file across two separate uploads.

## Learning
Suno's analytical outputs (such as BPM and display tags) are non-deterministic, even for identical audio inputs. This implies that single-reading drift studies or direct comparisons based on these outputs will be inherently noisy and unreliable. Future analyses relying on these metrics should account for this variability, potentially by averaging multiple readings.

## Evidence
Suno is non-deterministic - same audio file returned different BPM readings (72 vs 85) and different display_tags across two uploads, so single-reading drift studies will be noisy.
