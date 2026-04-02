---
title: "Working Memory Architecture — Warm Cache Between Sessions"
type: research
project: shirozen
topic: storage-retention
date: 2026-04-02
sources:
  - https://redis.io/blog/ai-agent-memory-stateful-systems/
  - https://mem0.ai/
  - https://arxiv.org/abs/2504.19413
  - https://dev.to/oblivionlabz/building-persistent-memory-for-ai-agents-a-4-layer-file-based-architecture-351i
  - https://dev.to/the_bookmaster/the-agent-memory-problem-nobody-solves-a-practical-architecture-for-persistent-context-3bln
  - https://machinelearningmastery.com/7-steps-to-mastering-memory-in-agentic-ai-systems/
tags: [working-memory, mem0, cache, warm-layer, decay, semantic-cache]
---

# Working Memory Architecture — Warm Cache Between Sessions

## Memory Type Hierarchy (Industry Standard)

| Layer | Scope | Persistence | Speed |
|-------|-------|------------|-------|
| **Sensory** | Current turn | None | Instant |
| **Working/Short-term** | Current task | Session-scoped | Fast |
| **Episodic** | Past sessions | Cross-session | Medium |
| **Semantic** | Domain knowledge | Permanent | Medium |
| **Procedural** | How-to patterns | Permanent | Fast |

## Mem0 Architecture (Reference Implementation)

### Core Design
- Memory orchestration layer between AI agents and storage
- Dynamically extracts, consolidates, and retrieves salient information
- Unified APIs for episodic, semantic, procedural, and associative memories
- **26% improvement** in LLM-as-a-Judge metric vs baseline

### Storage Options
- VectorDB: Qdrant, Chroma, Milvus, Pgvector, Redis, Azure AI Search
- Graph: Neo4j, Amazon Neptune for relational memory
- Cache: Redis for semantic caching (microsecond retrieval)

### Graph-Based Memory
- Captures complex relational structures among conversational elements
- Knowledge graph enables subgraph retrieval + semantic triplet matching
- Multi-hop, temporal, and open-domain reasoning
- **~2% higher** overall score vs base configuration

### Decay Mechanisms
- Automatic filtering to prevent memory bloat
- Decay mechanisms remove irrelevant information over time
- Cost optimization through semantic caching

## Working Memory for Shirozen

### The Gap in Current System
- brain.db is cold storage — queries are on-demand
- Auto-memory (MEMORY.md) is static — 200 lines, no prioritization by recency
- No "warm" layer that anticipates what's needed

### Shirozen Working Memory Design

```
┌────────────────────────────────────────────┐
│  Context Window (LLM)                      │
│  ← Shirozen pushes predicted context here  │
└──────────────────┬─────────────────────────┘
                   │
┌──────────────────▼─────────────────────────┐
│  Working Memory (Shirozen service)          │
│  - In-memory cache (Map/LRU)               │
│  - Recently accessed knowledge             │
│  - Active patterns                         │
│  - Predicted needs for current work streak  │
│  - Decay: unused entries expire in hours    │
└──────────────────┬─────────────────────────┘
                   │
┌──────────────────▼─────────────────────────┐
│  brain.db (Cold Storage)                    │
│  - FTS5 + vector search                    │
│  - All knowledge, decisions, references     │
│  - FSRS-based confidence scores             │
│  - Decay: weeks/months                      │
└────────────────────────────────────────────┘
```

### Working Memory Implementation
```typescript
// In-memory LRU cache within Shirozen HTTP service
class WorkingMemory {
  private cache: Map<string, WorkingMemoryEntry>;
  private maxSize: number = 100;
  private ttlMs: number = 4 * 60 * 60 * 1000; // 4 hours

  // Promote from brain.db when accessed
  promote(key: string, entry: BrainDbEntry): void;

  // Auto-populate based on pattern predictions
  prefetch(patternId: string): void;

  // Decay: entries not accessed within TTL are evicted
  sweep(): void;

  // Get most relevant entries for current context
  getRelevant(query: string, limit: number): WorkingMemoryEntry[];
}
```

### Warming Strategies
1. **Session start**: Load knowledge related to predicted workflow pattern
2. **Topic detection**: When first user prompt arrives, warm cache with related entries
3. **Work streak**: If user has been on same topic for 3+ sessions, keep that context warm
4. **Tool usage**: When specific tools are called, warm cache with related decisions
5. **Time-based**: Load context relevant to time-of-day patterns (e.g., morning = task review)

## Semantic Caching for Shirozen

### Concept
- Cache recent query-response pairs from brain.db
- When similar query arrives, return cached result without re-searching
- Use embedding similarity to match queries (threshold > 0.9)
- Cache TTL: 1-4 hours depending on query type

### Benefits
- Sub-millisecond retrieval for repeated context lookups
- Reduces brain.db query load during active sessions
- Natural working memory behavior: frequently needed context stays fast
