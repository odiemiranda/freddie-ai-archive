---
title: "Event Log Schema — Tool Call Tracking for Pattern Mining"
type: research
project: shirozen
topic: pattern-prediction
date: 2026-04-02
sources:
  - https://www.sqliteforum.com/p/building-event-sourcing-systems-with
  - https://learn.microsoft.com/en-us/azure/architecture/patterns/event-sourcing
  - https://www.xes-standard.org/
  - https://www.processmining.org/event-data.html
  - https://www.npmjs.com/package/@smartesting/vmsp
tags: [event-log, event-sourcing, xes, schema, tool-tracking, vmsp, pattern-mining]
---

# Event Log Schema — Tool Call Tracking for Pattern Mining

## Event Sourcing for Shirozen

### Why Event Sourcing?
- Append-only log of all agent actions
- Replay capability: reconstruct what happened in any session
- Pattern mining input: sequential events feed into VMSP/PrefixSpan
- Audit trail: every decision traceable

### Core Schema
```sql
CREATE TABLE event_log (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  session_id TEXT NOT NULL,
  timestamp TEXT NOT NULL,          -- ISO 8601
  event_type TEXT NOT NULL,         -- tool_call | user_prompt | agent_response | workflow_start | workflow_end
  agent TEXT,                       -- tala | rune | sol | echo | mccall | wick | freddie
  tool_name TEXT,                   -- brain.ts | discover.ts | validate-prompt | etc.
  action TEXT,                      -- specific action within tool
  context_topic TEXT,               -- detected topic/workflow
  metadata TEXT,                    -- JSON: additional event data
  sequence_num INTEGER NOT NULL     -- ordering within session
);

CREATE INDEX idx_event_session ON event_log(session_id);
CREATE INDEX idx_event_type ON event_log(event_type);
CREATE INDEX idx_event_agent ON event_log(agent);
CREATE INDEX idx_event_timestamp ON event_log(timestamp);
```

## XES Format Reference

### Standard Structure
Process mining uses XES (eXtensible Event Stream, IEEE 1849):
- **Log** → contains traces
- **Trace** → bundles events for one case (= one session)
- **Event** → single occurrence with attributes

### Mapping to Shirozen
| XES Concept | Shirozen Equivalent |
|-------------|-------------------|
| Log | brain.db event_log table |
| Trace | Session (grouped by session_id) |
| Event | Individual tool call or action |
| Case ID | session_id |
| Activity | tool_name + action |
| Timestamp | timestamp |
| Resource | agent |

## VMSP Library (@smartesting/vmsp)

### Key Finding: TypeScript Sequential Pattern Mining Exists!

```typescript
import { AlgoVMSP } from '@smartesting/vmsp';

// Database = array of sequences, each sequence = array of itemsets
const database = [
  [['discover'], ['query-brain'], ['build-style'], ['validate'], ['tri-critic']],
  [['discover'], ['query-brain'], ['build-style'], ['validate'], ['tri-critic'], ['reflect']],
  [['morning'], ['check-tasks'], ['discover'], ['build-style'], ['validate']],
];

const vmsp = new AlgoVMSP({
  patternType: 'maximal',   // or 'closed'
  maxGap: 3,                // max 3 events between pattern items
  minimumPatternLength: 2,
  maximumPatternLength: 10,
});

const result = vmsp.run(database, 0.5); // min support = 50%
// Returns: patterns with support counts
```

### VMSP vs PrefixSpan
| Feature | VMSP | PrefixSpan |
|---------|------|------------|
| Language | TypeScript (npm) | Python |
| Pattern type | Maximal/Closed | All frequent |
| Gap support | Yes (configurable) | No |
| Performance | Efficient vertical mining | Efficient prefix projection |
| **Verdict** | **Use this** — native TS, gap support | Backup for Python analysis |

## Event Capture Strategy

### What to Log
| Event | Source | Purpose |
|-------|--------|---------|
| `tool_call` | Hook: PreToolUse/PostToolUse | Every tool invocation |
| `user_prompt` | Hook: UserPromptSubmit | Topic detection, intent tracking |
| `workflow_start` | Skill dispatch | Workflow type identification |
| `workflow_end` | Skill completion | Duration, outcome |
| `agent_dispatch` | Subagent creation | Agent usage patterns |
| `search_query` | brain.ts calls | What knowledge is being sought |
| `decision_made` | reflect.ts | What was decided and why |
| `context_push` | Shirozen channel | What context was predicted |
| `prediction_hit` | Validation check | Whether prediction was used |

### Existing Hook: tool-call-counter.ts
- Already tracks tool calls per session
- **Extend**: Add event_log writes alongside counting
- Hook fires on PreToolUse/PostToolUse — perfect capture point

## Pattern Mining Pipeline

```
event_log (raw events)
    ↓ extract session sequences
VMSP (mine maximal patterns)
    ↓ patterns with support counts
pattern_log (store in brain.db)
    ↓ match against new sessions
context_predictions (predict next steps)
    ↓ push via channels
Working Memory → Context Window
```

## Shirozen Integration Points

1. **Capture**: Extend `tool-call-counter.ts` to write to `event_log` table
2. **Mine**: New tool `mine-patterns.ts` runs VMSP on recent sessions
3. **Store**: Write patterns to `pattern_log` table
4. **Predict**: At session start, match current prefix against patterns
5. **Push**: Channel notification with predicted context
6. **Validate**: Track prediction accuracy in `context_predictions`
