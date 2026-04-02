---
title: "Contradiction Detection — Prompts & Algorithms"
type: research
project: shirozen
topic: storage-retention
date: 2026-04-02
sources:
  - https://arxiv.org/html/2504.00180v1
  - https://github.com/datarootsio/knowledgebase_guardian
  - https://aclanthology.org/2024.emnlp-main.486.pdf
  - https://openreview.net/forum?id=EmQSOi1X2f
tags: [contradiction, detection, prompt, knowledge-base, fact-checking]
---

# Contradiction Detection — Prompts & Algorithms

## The Problem

During memory consolidation, new knowledge may contradict existing entries. Current system silently merges (averaging). Shirozen needs to detect and surface contradictions.

## Approach 1: Retrieve-Then-Compare (knowledgebase_guardian pattern)

1. When new entry arrives, embed it
2. Retrieve top-5 similar existing entries (cosine > 0.8)
3. For each pair, ask LLM to check for contradiction
4. Flag contradictions, don't merge

### Prompt Template
```
You are a knowledge base consistency checker. Compare these two knowledge entries and determine if they contradict each other.

Entry A (existing, from {date_a}):
{content_a}

Entry B (new, from {date_b}):
{content_b}

Analyze for these contradiction types:
1. VALUE CONFLICT: Different facts about the same thing
2. TEMPORAL CONFLICT: Something that was true but isn't anymore
3. LOGICAL CONFLICT: Two entries that can't both be true

Respond in JSON:
{
  "contradicts": true/false,
  "type": "value" | "temporal" | "logical" | "none",
  "explanation": "Brief explanation of the contradiction",
  "confidence": 0.0-1.0,
  "resolution_hint": "Which entry is likely more accurate and why"
}
```

## Approach 2: Self-Contradiction Detection (80% F1)

Research shows **chain-of-thought prompting** achieves ~80% F1 on contradiction detection:

```
Given the following two statements from our knowledge base, think step-by-step about whether they are consistent or contradictory.

Statement 1: "{content_a}"
Statement 2: "{content_b}"

Step 1: Identify the key claims in each statement.
Step 2: Check if any claims directly conflict.
Step 3: Check if any claims are implicitly inconsistent.
Step 4: Consider if both could be true in different contexts or time periods.

Conclusion: Are these statements contradictory? Explain your reasoning.
```

## Approach 3: Embedding-Based Pre-Filter

Before expensive LLM calls, use embedding similarity to narrow candidates:

```typescript
async function findContradictionCandidates(newEntry: string): Promise<string[]> {
  const embedding = await embed(newEntry);
  
  // High similarity = likely same topic = potential contradiction
  const similar = await db.query(`
    SELECT id, content, distance
    FROM vec_memories
    WHERE embedding MATCH ?
    AND distance < 0.3  -- very similar = same topic
    ORDER BY distance
    LIMIT 5
  `).all(embedding.buffer);
  
  return similar.map(s => s.id);
}
```

## Approach 4: Sentence-BERT + NLI Model

Use Natural Language Inference (NLI) model for fast, no-LLM contradiction detection:
- Cross-encode entry pairs with NLI model
- Output: entailment / contradiction / neutral
- Fast (sub-second) but less nuanced than LLM
- Good as pre-filter before LLM verification

## Implementation Strategy for Shirozen

### Phase 1: During Consolidation
```typescript
async function consolidateWithContradictionCheck(agent: string) {
  const inbox = loadInbox(agent);
  const knowledge = loadKnowledge(agent);
  
  for (const newEntry of inbox) {
    // 1. Find similar existing entries
    const candidates = await findContradictionCandidates(newEntry.content);
    
    // 2. Check each candidate for contradiction
    for (const candidateId of candidates) {
      const existing = await getEntry(candidateId);
      const result = await checkContradiction(newEntry, existing);
      
      if (result.contradicts) {
        // 3. Log contradiction instead of merging
        await logContradiction({
          entry_a_id: candidateId,
          entry_b_id: newEntry.id,
          type: result.type,
          description: result.explanation,
          confidence: result.confidence,
          resolution_hint: result.resolution_hint
        });
      }
    }
  }
}
```

### Phase 2: Periodic Background Scan
- Batch-compare knowledge entries within same topic clusters
- Run weekly or when pattern_log shows knowledge was used unsuccessfully

### Phase 3: Morning Briefing Integration
```
Contradictions detected:
  ⚠ Entry "Suno v5.5 handles pipes differently" vs "Pipe syntax unchanged since v5.0"
    Type: temporal | Confidence: 0.85
    Hint: The newer entry (Apr 1) is likely correct, older (Mar 26) outdated
```

## Key Metric
Chain-of-thought prompting: **~80% F1** on contradiction detection. Acceptable for knowledge base use case where false positives (flagged non-contradictions) are cheap to dismiss and false negatives (missed contradictions) are the real cost.
