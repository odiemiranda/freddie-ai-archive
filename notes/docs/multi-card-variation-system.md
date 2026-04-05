# Multi-Card Variation System

Technical design for multi-card triangulation + axis-shift variation generation.

**Problem:** When a single music card matches, all variations anchor to the same BPM/key/groove/palette. Generations sound samey despite being separate variations.

**Solution:** Discover surfaces multiple cards per build. Each variation uses a different card combination strategy with a forced axis shift.

## Current Flow (single-card)

```
discover("lo-fi shamisen jazz")
  → 1 music card match (best fit)
  → Tala loads card, uses Profile as hard constraints
  → V1: 86 BPM, B minor, shamisen lead, sax + piano
  → V2: 86 BPM, B minor, shamisen lead, sax + piano (slightly different tags)
  → V3: 86 BPM, B minor, shamisen lead, sax + piano (slightly different tags)
```

All three variations orbit the same point.

## Proposed Flow (multi-card triangulation + axis shift)

```
discover("lo-fi shamisen jazz")
  → 3 music card matches: primary (A), secondary (B), contrast (C)
  → Tala loads all three, builds variation matrix

  → V1: Faithful to card A (close reference)
  → V2: Blend A + B (shift instrumentation or tempo)
  → V3: Contrast via C (different energy, lead instrument, or key)
```

## Design

### 1. Discover Changes (`src/tools/discover.ts`)

**Current:** Returns all matching cards, agent picks one.

**Proposed:** When multiple cards match, tag them with roles:

| Role | Selection criteria | Purpose |
|------|-------------------|---------|
| **primary** | Highest keyword overlap with query | Closest reference — faithful anchor |
| **secondary** | Same genre family, different BPM or key | Blend candidate — similar vibe, different angle |
| **contrast** | Same broad category, most different Profile values | Deliberate divergence — prevents sameness |

**Output format change:**

```
MATCHED MUSIC CARDS:
  [primary]  shamisen-jazz-cafe.md (86 BPM, B minor, jazz swing)
  [secondary] shamisen-mix-downtown.md (95 BPM, Db minor, lo-fi hip-hop)
  [contrast]  japan-cafe-light-jazz.md (80 BPM, Bb major, cafe brushes)
```

**Role assignment logic:**
1. Rank all matched cards by keyword relevance → top match = primary
2. From remaining, pick the card with closest mood but most different BPM or key → secondary
3. From remaining, pick the card with greatest Profile distance (BPM delta + key distance + energy difference) → contrast
4. If only 2 cards match → primary + contrast (skip secondary)
5. If only 1 card matches → primary only, use axis shift without card blending

**Profile distance function:**
```
distance(A, B) = |bpm_a - bpm_b| / 20          # normalized BPM delta
               + key_distance(key_a, key_b)     # circle-of-fifths distance, 0-6
               + energy_delta(energy_a, energy_b) # 0 (same), 1 (adjacent), 2 (opposite)
```

### 2. Agent Workflow Changes (`suno.md`)

Add a new section: **Variation Strategy Matrix**

#### When 3 cards are available (primary A, secondary B, contrast C):

| Variation | Card source | BPM | Key | Lead instrument | Groove |
|-----------|------------|-----|-----|-----------------|--------|
| V1 | A only | A's BPM | A's key | A's lead | A's groove |
| V2 | Blend A + B | B's BPM (or midpoint) | A's key or B's key | B's lead, A's texture | Hybrid groove |
| V3 | C + mood twist | C's BPM | C's key | C's lead | C's groove, shifted energy |

#### When 2 cards are available (primary A, contrast C):

| Variation | Card source | Axis shift |
|-----------|------------|------------|
| V1 | A faithful | None — close reference |
| V2 | A with tempo shift | BPM ±10, keep A's palette |
| V3 | C contrast | Different lead, key, energy |

#### When 1 card is available (primary A only):

| Variation | Card source | Axis shift |
|-----------|------------|------------|
| V1 | A faithful | None — close reference |
| V2 | A with instrument swap | Same BPM/key, different lead instrument |
| V3 | A with energy/tempo shift | BPM ±10, shift energy level, different groove density |

#### Axis Shift Definitions

| Axis | What changes | Example |
|------|-------------|---------|
| **Tempo** | BPM ±10-15 | 86 BPM → 95 BPM |
| **Key** | Shift key (relative major/minor, or ±2 semitones) | B minor → D minor, or B minor → Bb major |
| **Lead** | Swap primary instrument | Shamisen lead → piano lead, shamisen as texture |
| **Groove** | Change rhythm feel | Jazz swing → lo-fi hip-hop → bossa nova |
| **Energy** | Shift energy level | Medium → low (intimate) or medium → high (driving) |
| **Density** | Change arrangement fullness | Full band → duo/trio, or sparse → layered |

**Rule:** Each variation must shift at least one axis. V2 shifts 1-2 axes. V3 shifts 2-3 axes. No two variations may share the same lead instrument AND groove AND BPM.

### 3. Variation Logging

Each variation's notes section should log which cards were used and what axes were shifted:

```markdown
## Style Block Notes

**Reference cards:**
- [primary] shamisen-jazz-cafe.md — BPM 86, B minor
- [contrast] japan-cafe-light-jazz.md — BPM 80, Bb major

**Variation strategy:** V3 — contrast card, shifted lead (piano) + key (Bb major)

**Tri-critic summary:** ...
```

This creates a feedback loop — reflect.ts can analyze which card combos + axis shifts produced the best results.

### 4. Brain.db Integration

After generation, reflect.ts logs a decision:

```
decision: "multi-card blend A+C for lo-fi shamisen jazz"
context: "V3 used japan-cafe-light-jazz contrast card, shifted to piano lead + Bb major"
outcome: "kept" | "discarded"
tags: ["variation-strategy", "multi-card", "shamisen", "jazz"]
```

Over time, brain.db accumulates which card combos and axis shifts work — Tala can query this before building.

## Implementation Order

1. **discover.ts** — Add card role assignment (primary/secondary/contrast) + profile distance function
2. **suno.md** — Add variation strategy matrix section + axis shift rules
3. **card.ts** (musiccard) — Ensure Profile section has parseable BPM/key values for distance calc
4. **reflect.ts** — Log card combo + axis shift in decisions
5. **minimax-music** (Sol) — Apply same pattern (secondary priority)

## Scope

- Suno agent (Tala) first — she's the primary consumer of music cards
- MiniMax Music agent (Sol) second — same pattern, different translation format
- Lyrics agent (Rune) — not affected (lyrics don't use music card constraints)
- No changes to music card format — cards stay as-is
- No changes to tri-critic — critics evaluate the final Style block regardless of source strategy
