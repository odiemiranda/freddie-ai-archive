---
title: "Temporal Event Embeddings for Workflow Pattern Vectors"
type: research
project: shirozen
topic: pattern-prediction
date: 2026-04-02
sources:
  - https://www.kdd.org/kdd2016/papers/files/rpp1081-duA.pdf
  - https://www.researchgate.net/publication/318857021_Event2vec_Learning_Representations_of_Events_on_Temporal_Sequences
  - https://www.nature.com/articles/s41598-020-63221-2
  - https://arxiv.org/html/2407.13053
  - https://hal.science/hal-03464623/document
tags: [event-embedding, temporal, event2vec, workflow-vectors, pattern-representation]
---

# Temporal Event Embeddings for Workflow Pattern Vectors

## The Problem

Shirozen's pattern_log stores workflow fingerprints as JSON arrays of event types. For vector-based pattern matching, we need to embed these sequences into fixed-dimensional vectors.

## Academic Approaches

### Event2vec
- Constructs **Event Connection Graph**: vertices = unique events, edges = temporal relations
- Feed graph to embedding neural network → learned vectors
- Captures both structural and temporal similarities
- **Application**: Embed tool call sequences as vectors for similarity search

### RMTPP (Recurrent Marked Temporal Point Processes)
- RNN-based hidden unit learns unified representation of history dependency
- Conditional intensity function captures past event timings + event markers
- **Application**: Predict next event timing and type based on history

### weg2vec (Temporal Network Embedding)
- Builds on temporal and structural similarities of events
- Low-dimensional representation capturing latent structures
- Successfully captures similarities between events involving different nodes at different times
- **Application**: Find similar workflow patterns even if they use different tools

### E2Vec (Feature Embedding with Temporal Information)
- Combines event features with temporal context
- Originally for e-book learning analytics — applicable to session analytics
- **Application**: Embed session events with time-of-day / duration context

## Practical Approach for Shirozen

### Simple: Bag-of-Events Embedding
```typescript
// Convert event sequence to fixed-size vector
function embedSequence(events: string[]): Float32Array {
  // 1. Get unique event vocabulary from pattern_log
  const vocab = getEventVocabulary(); // e.g., 50 unique event types

  // 2. Create TF-IDF-like vector
  const vec = new Float32Array(vocab.length);
  for (const event of events) {
    const idx = vocab.indexOf(event);
    if (idx >= 0) vec[idx] += 1;
  }

  // 3. Normalize
  const norm = Math.sqrt(vec.reduce((sum, v) => sum + v * v, 0));
  if (norm > 0) vec.forEach((v, i) => vec[i] = v / norm);

  return vec;
}
```

### Better: Sequence-Aware Embedding
```typescript
// Use ONNX embeddings on the sequence description
function embedSequenceDescription(events: string[]): Float32Array {
  // Convert sequence to natural language description
  const desc = `Workflow: ${events.join(' → ')}`;
  // Use existing ONNX embedding model (384-dim)
  return embed(desc);
}
```

### Best: Temporal Position Encoding
```typescript
// Encode position within sequence + event identity
function embedWithPosition(events: Array<{type: string, relativeTime: number}>): Float32Array {
  const embeddings = events.map((e, i) => {
    const eventEmbed = embed(e.type);        // semantic embedding of event name
    const posEncoding = positionalEncoding(i, events.length); // position in sequence
    const timeEncoding = temporalEncoding(e.relativeTime);    // time offset
    return concat(eventEmbed, posEncoding, timeEncoding);
  });
  // Pool all event embeddings into single sequence vector
  return meanPool(embeddings);
}
```

## Recommendation for Shirozen

**Start with "Better" approach** — use existing ONNX embedding model to embed natural language descriptions of workflow sequences. This:
- Requires no new models or training
- Leverages existing 384-dim ONNX infrastructure
- Enables semantic similarity (e.g., "discover → build" is similar to "query-brain → construct")
- Can be stored in vec_patterns table alongside pattern_log

**Upgrade later** to temporal position encoding if prediction accuracy needs improvement.

## Storage in brain.db

```sql
-- Extend pattern_log with embedding
CREATE VIRTUAL TABLE vec_patterns USING vec0(
  embedding float[384]
);

-- Index pattern embeddings alongside pattern_log entries
-- When new session starts, embed current prefix → KNN against vec_patterns
-- Top-k matches = predicted workflow continuations
```
