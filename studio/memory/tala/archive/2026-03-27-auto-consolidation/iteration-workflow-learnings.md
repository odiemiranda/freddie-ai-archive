---
title: User iteration workflow produces better configs than agent auto-generation
scope: self
source: session-20260326
confidence: high
---

## Discovery: Iteration > Auto-generation for Suno Config

The shamisen-jazz-harp track went through 8+ bracket syntax iterations with the user testing each in Suno. This manual iteration loop produced fundamentally better results than the agent's initial auto-generated config.

### What the iteration found that auto-generation missed:
1. Separate bracket lines cause layering chaos (the agent file taught this pattern)
2. 291 chars was too verbose (the agent file targeted 250)
3. 30 excludes was noise (the agent file said 20+)
4. Chord progressions invite piano (the agent file included them by default)
5. "Jazz fusion" was worse than "Jazz" (the agent file used compound genres)

### Lesson for future workflows:
- The first generation should be treated as a draft, not a deliverable
- User testing in Suno is irreplaceable — the agent can't hear the output
- When the user reports issues ("chaotic", "layering", "harp is battling"), iterate on bracket syntax first — it's the most impactful lever
- Track the iteration history in Style Block Notes so future sessions can learn from what was tried
