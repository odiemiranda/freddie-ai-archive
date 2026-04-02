---
title: "brain.db Integration — FSRS Decay + sqlite-vec + Pattern Storage"
type: research
project: shirozen
topic: brain-db
date: 2026-04-02
sources:
  - https://github.com/open-spaced-repetition/ts-fsrs
  - https://www.npmjs.com/package/ts-fsrs
  - https://github.com/AustinShelby/simple-ts-fsrs
  - https://alexgarcia.xyz/sqlite-vec/js.html
  - https://github.com/asg017/sqlite-vec/blob/main/examples/simple-bun/demo.ts
  - https://alexgarcia.xyz/sqlite-vec/features/knn.html
  - https://github.com/stephenc222/example-sqlite-vec-tutorial
tags: [brain-db, fsrs, sqlite-vec, decay, vector, bun, typescript]
---

# brain.db Integration — FSRS Decay + sqlite-vec + Pattern Storage

## FSRS (Free Spaced Repetition Scheduler) for Knowledge Decay

### What is FSRS?
- Modern replacement for SM-2 (Anki's old algorithm)
- Based on DSR model: **Difficulty, Stability, Retrievability**
- Forgetting curve: R = e^{-t/S}
- Academic-backed, widely adopted (Anki 23.10+)

### TypeScript Implementations
- **ts-fsrs** (official): `npm install ts-fsrs` — Node 18+, ESM/CJS/UMD
- **simple-ts-fsrs**: Minimal implementation, good for understanding
- **@squeakyrobot/fsrs**: Pure TS, FSRS v4.5 + optional v6
- **femto-fsrs**: Lightweight alternative

### How FSRS Maps to Shirozen

Current brain.db decay model:
- 90/180 day staleness penalties
- Decision confidence decay (failed outcomes 2x faster)
- Completed project downweight (0.5x)

**FSRS upgrade**:
```typescript
import { FSRS, Card, Rating } from 'ts-fsrs';

// Each knowledge entry = a "card"
const fsrs = new FSRS();
const card = new Card(); // fresh knowledge entry

// When knowledge is validated (used successfully in a workflow):
const result = fsrs.repeat(card, new Date());
const updatedCard = result[Rating.Good]; // or Rating.Easy, Rating.Hard

// When knowledge fails (led to bad outcome):
const failResult = fsrs.repeat(card, new Date());
const failedCard = failResult[Rating.Again]; // rapid decay

// Retrievability = probability this knowledge is still accurate
// If retrievability < threshold (e.g., 0.7), flag for re-validation
```

### New brain.db Columns for FSRS
```sql
ALTER TABLE memories ADD COLUMN fsrs_difficulty REAL DEFAULT 0.3;
ALTER TABLE memories ADD COLUMN fsrs_stability REAL DEFAULT 1.0;
ALTER TABLE memories ADD COLUMN fsrs_retrievability REAL DEFAULT 1.0;
ALTER TABLE memories ADD COLUMN fsrs_last_review TEXT;
ALTER TABLE memories ADD COLUMN fsrs_reps INTEGER DEFAULT 0;
ALTER TABLE memories ADD COLUMN fsrs_lapses INTEGER DEFAULT 0;

-- Same for decisions table
ALTER TABLE decisions ADD COLUMN fsrs_difficulty REAL DEFAULT 0.3;
ALTER TABLE decisions ADD COLUMN fsrs_stability REAL DEFAULT 1.0;
```

## sqlite-vec + Bun Integration

### Already Working
brain.db already uses sqlite-vec with bun:sqlite. Key patterns:

```typescript
import { Database } from "bun:sqlite";
import * as sqliteVec from "sqlite-vec";

const db = new Database("brain.db");
sqliteVec.load(db);

// KNN query
const results = db.query(`
  SELECT rowid, distance 
  FROM vec_memories 
  WHERE embedding MATCH ?
  ORDER BY distance 
  LIMIT 5
`).all(new Float32Array(embedding).buffer);
```

### What Shirozen Adds
- **Pattern embeddings**: New `vec_patterns` table for workflow fingerprint vectors
- **Context predictions**: New `vec_predictions` for predicted context vectors
- **Cross-store similarity**: Compare pattern embeddings against memory/decision/reference embeddings

## New Tables for Shirozen

### pattern_log
```sql
CREATE TABLE pattern_log (
  id INTEGER PRIMARY KEY,
  pattern_sequence TEXT NOT NULL,  -- JSON array of event types
  support_count INTEGER DEFAULT 1,
  confidence REAL DEFAULT 0.5,
  first_seen TEXT NOT NULL,
  last_seen TEXT NOT NULL,
  prediction_accuracy REAL,  -- % of times pattern completed as predicted
  fsrs_stability REAL DEFAULT 1.0,
  fsrs_retrievability REAL DEFAULT 1.0
);

CREATE VIRTUAL TABLE vec_patterns USING vec0(
  embedding float[384]  -- ONNX local embeddings (384-dim)
);
```

### decision_feedback
```sql
CREATE TABLE decision_feedback (
  id INTEGER PRIMARY KEY,
  decision_id INTEGER REFERENCES decisions(id),
  outcome TEXT NOT NULL,  -- success|failure|partial|abandoned
  outcome_details TEXT,
  recorded_at TEXT NOT NULL,
  session_raw_id TEXT,  -- link to raw session
  confidence_delta REAL  -- how much to adjust decision confidence
);
```

### context_predictions
```sql
CREATE TABLE context_predictions (
  id INTEGER PRIMARY KEY,
  session_id TEXT,
  predicted_at TEXT NOT NULL,
  prediction_type TEXT NOT NULL,  -- workflow|topic|agent|tool
  predicted_value TEXT NOT NULL,
  confidence REAL DEFAULT 0.5,
  was_correct INTEGER,  -- NULL until validated, 0 or 1
  validated_at TEXT
);
```

## Shirozen Integration Points

1. **reflect.ts** → After workflow, log decision feedback + extract patterns
2. **consolidate-memory.ts** → Run contradiction detection, update FSRS scores
3. **morning.ts** → Query pattern_log for predicted context, include in briefing
4. **startup hook** → Push predicted context via channels based on recent patterns
5. **brain.ts --search** → Factor FSRS retrievability into ranking scores
