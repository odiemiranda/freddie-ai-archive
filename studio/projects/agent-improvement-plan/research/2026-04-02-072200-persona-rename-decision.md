# Agent Persona Rename Decision

Decided 2026-04-02.

## Final Agent Lineup

| Role | Agent Name | Character Foundation | Previous |
|------|-----------|---------------------|----------|
| **PM** | **Holmes** | Sherlock Holmes ("The Product Detective") | Ryan (Jack Ryan) |
| **Architect** | **McCall** | Robert McCall (upgraded principal architect) | McCall (same, upgraded) |
| **Developer** | **Ryan** | Jack Ryan (analyst-turned-operator) | Wick (John Wick) |

## Rationale

### Developer: Wick → Ryan (Jack Ryan)
John Wick's "blind obedience" trait directly conflicts with senior engineer traits we want: implementation pushback, defensive coding instincts, spotting design smells. Jack Ryan is an analyst-turned-operator — methodical, follows the chain of command but questions bad intelligence, escalates when the mission brief doesn't match reality. Resolves the core persona conflict.

### PM: Ryan → Holmes (Sherlock Holmes)
Sherlock works better as PM than architect because:
- PM's job IS finding "the one right answer" — what does the user actually need?
- Evidence obsession → every assumption backed by data
- Deduction chains → hypothesis-driven PRDs
- Pattern recognition → seeing the real problem behind feature requests
- Sherlock's arrogance and "one right answer" mindset — problematic for architecture (trade-offs) — is an asset for product requirements (ground truth)

### Architect: McCall stays McCall
McCall's "protector" mindset (contingency thinking, operational empathy) is a healthy behavioral anchor for architecture. Just needs principal architect depth added.

## Rename Scope

### File Renames
- `.claude/souls/ryan.md` → `.claude/souls/holmes.md` (new Sherlock content)
- `.claude/souls/wick.md` → `.claude/souls/ryan.md` (new Jack Ryan content)
- `.claude/agents/pm.md` — `name: holmes`
- `.claude/agents/dev.md` — `name: ryan`

### Cross-References to Update
- McCall soul/agent: "Wick" → "Ryan", "Ryan" → "Holmes"
- All skills referencing old agent names
- All workflows referencing old agent names
- CLAUDE.md team table

### Memory Directory Renames
- `memory/wick/` → `memory/ryan/`
- `memory/ryan/` → `memory/holmes/`

### Risks for Holmes
- Analysis paralysis — needs "bias for action at 70% confidence" constraint
- Low empathy — might design logically perfect but cold workflows
- PRD rigidity — might clash with McCall when tech constraints require scope changes

### Risks for Ryan (Dev)
- Analyst tendencies might cause scope creep — needs constraint to stay within assigned files
- May over-analyze instead of executing — needs "smallest working change" anchor from old Wick
