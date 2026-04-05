---
title: "Eidetic Memory Task 6 — Skill Wiring"
type: task
status: pending
parent: 2026-04-01-000700-eidetic-memory-raw-layer
task: 6
depends: [2, 3, 5]
created: 2026-04-01
author: mccall
---

# Task 6 — Skill Wiring

## Objective

Wire `writeRaw()` and `insertRawFile()` into existing skill workflows so that every completed workflow produces a raw context file and threads the timestamp into `reflect()`.

## What to Build

### Pattern

Every skill endpoint follows this pattern at end-of-workflow:

```typescript
import { writeRaw } from "../libs/raw.js";
import { insertRawFile } from "../libs/brain/index.js";

// ... end of workflow ...

// 1. Write raw context file
const rawResult = writeRaw({
  agents: ["tala", "echo"],
  skill: "/create-track",
  tags: ["jazz", "lo-fi"],
  summary: "Jazz lo-fi track with brush drums",
  sections: [
    { name: "User Input", content: userInput },
    { name: "Discover", content: discoverOutput },
    { name: "Distill", content: distillOutput },
    { name: "Variations", content: variationsOutput },
    { name: "Critic Feedback", content: criticOutput },
    { name: "Final Output", content: finalOutput },
    { name: "Conversation", content: conversationLog },
  ],
});

// 2. Index immediately (don't wait for next sync)
await insertRawFile(rawResult.filePath);

// 3. Reflect with raw_ref
await reflect({
  agent: "tala",
  task: "...",
  workflow: "suno-generation",
  outcome: "success",
  trace: "...",
  rawRef: rawResult.timestamp,
});
```

### Skills to Wire

Each skill file needs the writeRaw + insertRawFile + rawRef threading added. The sections vary by skill type:

| Skill | File | Agents | Sections |
|-------|------|--------|----------|
| `/create-track` | `.claude/skills/create-track/workflow.md` | tala, echo | User Input, Discover, Distill, Variations, Critic Feedback, Final Output |
| `/create-track-variant` | `.claude/skills/create-track-variant/workflow.md` | tala, echo | User Input, Discover, Variations, Critic Feedback, Final Output |
| `/lyrics` | `.claude/skills/lyrics/workflow.md` | rune, echo | User Input, Discover, Distill, Variations, Critic Feedback, Final Output |
| `/minimax-music` | `.claude/skills/minimax-music/workflow.md` | sol, echo | User Input, Discover, Distill, Variations, Critic Feedback, Final Output |
| `/minimax-art` | `.claude/skills/minimax-art/workflow.md` | iris | User Input, Discover, Final Output |
| `/minimax-video` | `.claude/skills/minimax-video/workflow.md` | veo | User Input, Discover, Final Output |
| `/tracknames` | `.claude/skills/tracknames/workflow.md` | (tool) | User Input, Final Output |
| `/create-music-card` | `.claude/skills/create-music-card/workflow.md` | (tool) | User Input, Analysis, Final Output |
| `/architect` | `.claude/skills/architect/workflow.md` | mccall | User Input, Analysis, Council, Decision, Final Output |
| `/dev-start` | `.claude/skills/dev-start/workflow.md` | wick | User Input, Implementation, Final Output |

### Important Notes

- **Skill files are markdown workflows, not TypeScript.** The wiring happens as instructions in the workflow markdown telling the agent to call `writeRaw()` at end-of-workflow. The agent (running as Claude Code) executes these instructions.
- **Sections are best-effort.** Agents capture what they have. If a workflow skipped discover (e.g., a quick variant), that section is empty and writeRaw omits it.
- **Summary should be concise.** One sentence describing what happened. Agents generate this at end-of-workflow.
- **Tags come from the workflow context.** Genre tags, platform tags, concept tags.

### Workflow Markdown Changes

For each skill's `workflow.md`, add a section at the end (before existing reflect step if present, or as the final step):

```markdown
## Raw Context Capture

Before reflecting, capture the full session context:

1. Call `writeRaw()` via bun:
   ```bash
   bun -e "
     import { writeRaw } from './src/libs/raw.js';
     import { insertRawFile } from './src/libs/brain/index.js';
     const result = writeRaw({
       agents: ['{agents}'],
       skill: '{skill}',
       tags: [{tags}],
       summary: '{summary}',
       sections: [
         { name: 'User Input', content: \`{userInput}\` },
         // ... other sections with actual content from this session
       ],
     });
     await insertRawFile(result.filePath);
     console.log(JSON.stringify({ timestamp: result.timestamp, filePath: result.filePath }));
   "
   ```
2. Pass `rawRef: {timestamp}` to the reflect step.
```

### Alternative: Inline Execution

Since skills run inline (not forked), the agent can call writeRaw directly via bun eval at end-of-workflow. The key is that the agent has access to all the context from the session and can assemble the sections.

## Files to Modify

| File | Change |
|------|--------|
| `.claude/skills/create-track/workflow.md` | Add raw context capture step |
| `.claude/skills/create-track-variant/workflow.md` | Add raw context capture step |
| `.claude/skills/lyrics/workflow.md` | Add raw context capture step |
| `.claude/skills/minimax-music/workflow.md` | Add raw context capture step |
| `.claude/skills/minimax-art/workflow.md` | Add raw context capture step |
| `.claude/skills/minimax-video/workflow.md` | Add raw context capture step |
| `.claude/skills/tracknames/workflow.md` | Add raw context capture step |
| `.claude/skills/create-music-card/workflow.md` | Add raw context capture step |
| `.claude/skills/architect/workflow.md` | Add raw context capture step |
| `.claude/skills/dev-start/workflow.md` | Add raw context capture step |

## Acceptance Criteria

1. Each listed skill has a "Raw Context Capture" step in its workflow
2. The step calls `writeRaw()` with appropriate agents, skill, tags, and sections
3. The step calls `insertRawFile()` for immediate indexing
4. The step passes `rawRef` timestamp to the reflect step
5. Sections are appropriate for the skill type (creative skills have Variations/Critic, non-creative don't)

## Test Criteria

Run a skill end-to-end (e.g., `/create-track` with a simple input). Verify:
1. A raw file exists in `vault/studio/memory/raw/`
2. The raw file has correct frontmatter and sections
3. brain.db has the file indexed in `raw_files` and `raw_chunks`
4. The reflect output includes `raw_ref` in inbox file frontmatter
