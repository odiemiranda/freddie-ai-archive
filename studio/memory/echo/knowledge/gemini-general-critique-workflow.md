---
name: Gemini General Critique Workflow
description: A multi-stage workflow leveraging Gemini for comprehensive critique of documents, including agent design files.
type: knowledge
agent: echo
tags: [gemini, critique, workflow, multi-pass, document-analysis, agent-design, tool-capability]
---

## Overview
A multi-stage critique workflow, utilizing an initial 'flash' pass for high-level structural issues (e.g., redundancy) followed by a 'pro-level' pass for deeper content and qualitative analysis (e.g., tactical depth, voice, opinion quality), is an effective pattern for comprehensive document critique. Gemini is a capable tool for this, demonstrating proficiency in identifying structural bloat, tactical deficiencies, and qualitative aspects like opinion strength and voice consistency across various document types, extending its validated critique capabilities beyond music generation.

## Workflow Stages

### Stage 1: Flash Pass (Intentionality & Redundancy)
An initial rapid pass focuses on high-level structural integrity, identifying filler, redundancy, and overall intentionality of the content. This stage aims to clean up surface-level bloat.

### Stage 2: Pro-Level Pass (Tactical Depth & Qualitative Analysis)
A subsequent, more in-depth pass delves into the substantive content. This includes assessing tactical depth, evaluating the quality and consistency of opinions, and analyzing the voice and personal texture of the document.

## Why
This multi-stage approach ensures both structural cleanliness and deep content analysis. The initial pass quickly addresses common issues like verbosity, allowing the second pass to focus on more nuanced and critical aspects without being muddled by superficial problems. Gemini's broad knowledge base makes it effective across diverse domains, identifying specific issues in complex documents like agent design files.

## How to Apply
1.  **First Pass**: Prompt Gemini for a 'flash' review focused on identifying redundancy, filler, and overall intentionality.
2.  **Second Pass**: After incorporating initial fixes, prompt Gemini for a 'pro-level' review, focusing on tactical depth, opinion quality, and voice consistency. Specify areas like API design, migration strategies, debt triage, build-vs-buy decisions, data modeling, and debugging for technical documents.

## Evidence
In critiquing an agent design document ('soul file'):
-   The intentionality critic (flash pass) found zero slop but flagged over 20 filler/redundancy items.
-   The pro-level review confirmed tactical gaps in API design, migration, debt triage, build-vs-buy, data modeling, and debugging.
-   Opinion grades identified included 2 strong (4-5), 2 solid (3-4), and 2 weak (2-3) opinions.
-   No internal contradictions were found, and the agent's voice was coherent but thin on personal detail.

**Consolidated From**:
- `20260402-200615-a-multi-stage-critique-workflow-utilizin-2.md`
- `20260402-200615-gemini-is-a-capable-tool-for-critiquing--1.md`
