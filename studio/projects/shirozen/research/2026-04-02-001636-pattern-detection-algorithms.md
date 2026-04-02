---
title: "Pattern Detection Algorithms for Workflow Fingerprinting"
type: research
project: shirozen
topic: pattern-prediction
date: 2026-04-02
sources:
  - https://github.com/chuanconggao/PrefixSpan-py
  - https://github.com/process-intelligence-solutions/pm4py
  - https://github.com/cortado-tool/cortado
  - https://sequenceanalysis.github.io/slides/analyzing_sequential_user_behavior_part2.pdf
  - https://hanj.cs.illinois.edu/pdf/span01.pdf
  - https://www.philippe-fournier-viger.com/spmf/PrefixSpan.php
tags: [pattern-detection, prefixspan, process-mining, sequential-mining, workflow]
---

# Pattern Detection Algorithms for Workflow Fingerprinting

## Core Problem

Shirozen needs to detect recurring workflow patterns from:
- Raw session transcripts (tool call sequences, topic trajectories)
- User behavior patterns (time-of-day, task clustering)
- Agent interaction sequences (which agents called in what order)

## Sequential Pattern Mining

### PrefixSpan Algorithm
- **Best fit** for Shirozen's pattern detection needs
- Discovers all frequent sequential patterns in a sequence database
- Only grows longer patterns from shorter frequent ones (no candidate generation)
- Python implementation: `pip install prefixspan` ([PrefixSpan-py](https://github.com/chuanconggao/PrefixSpan-py))
- Also includes BIDE (closed patterns) and FEAT (generator patterns)
- Supports: top-k patterns, length limiting, custom key/filter/callback functions

### How PrefixSpan Maps to Shirozen

**Input: Session sequences**
```
Session 1: [discover, ask-brain, build-style, validate, tri-critic, generate]
Session 2: [discover, ask-brain, build-style, validate, tri-critic, generate, reflect]
Session 3: [morning, check-tasks, discover, build-style, validate]
```

**Output: Frequent patterns**
```
[discover, build-style, validate] — support: 3/3 (100%)
[discover, ask-brain, build-style] — support: 2/3 (67%)
[tri-critic, generate] — support: 2/3 (67%)
```

**Application**: When Shirozen sees `discover` → `ask-brain`, it predicts `build-style` → `validate` → `tri-critic` and pre-loads relevant context.

## Process Mining Tools

### PM4Py (Python)
- Full process mining library
- Process discovery, conformance checking, enhancement
- Can generate process models from event logs
- **Relevant**: Extract workflow models from session logs, detect deviations

### Cortado
- Interactive & incremental process discovery
- End-user focused, good for iterative refinement
- Could be used to visually explore session patterns

### ProcessM
- Real-time intelligent process mining
- Published in SoftwareX (2025)
- Real-time aspect relevant for Shirozen's streaming needs

## Implementation Strategy for Shirozen

### Phase 1: Data Extraction
- Parse raw session transcripts into event sequences
- Each event = (timestamp, agent, tool, action, context)
- Store as structured event log in brain.db (new `event_log` table)

### Phase 2: Pattern Mining
- Run PrefixSpan on event sequences with min_support threshold
- Extract workflow fingerprints (frequent subsequences)
- Store as `pattern_log` entries in brain.db with:
  - pattern sequence, support count, confidence, last_seen
  - decay: patterns not seen recently lose confidence

### Phase 3: Prediction
- When a new session starts, match current event prefix against known patterns
- Use prefix matching to predict likely next steps
- Pre-load context relevant to predicted workflow

### Phase 4: Adaptation
- Track prediction accuracy (did the pattern complete?)
- Reinforce accurate patterns, decay inaccurate ones
- New patterns emerge organically from session data

## Language Considerations

- PrefixSpan has Python implementations but no TypeScript
- Options: (1) Port to TypeScript, (2) Call Python from Bun, (3) Use WASM
- PM4Py is Python-only
- **Recommendation**: Build a lightweight TypeScript PrefixSpan for brain.db integration, keep Python tools for analysis/visualization
