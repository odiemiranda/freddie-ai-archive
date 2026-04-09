---
source: reflection
agent: freddie
project: unknown
projectPath: ""
task: "FAI-0003 + FAI-0004 shipped"
workflow: dev-start
outcome: success
scope: shared
confidence: high
memoryType: semantic
importance: 9
tags: [data_integrity, bug_fix, defensive_coding, pipeline_design, external_service, gemini, suno, McCall, robustness]
session: 2026-04-10T02:17:08
---

## Observation
McCall identified a critical drift-study contamination bug where Suno data (via cardContent) was leaking into Gemini refinement, which was fixed by implementing a two-pass build strategy.

## Learning
When integrating data from multiple external services, especially for refinement or comparison, a two-pass build strategy (processing one service's output, then splicing in data from another) is an effective pattern to prevent data contamination and ensure integrity.

## Evidence
McCall caught a critical drift-study contamination bug where Suno data was leaking into Gemini refinement via cardContent - fixed by two-pass build that splices Suno data AFTER Gemini returns.
