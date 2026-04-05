---
title: "Tech Approach: Phonetic Analysis for Lyrics"
date: 2026-03-29
status: in-progress
source: https://github.com/jwynia/agent-skills/tree/main/skills/creative/music/lyric-diagnostic
tags: [lyrics, phonetics, quality, critics]
---

# Tech Approach: Phonetic Analysis for Lyrics

## Goal

Add syllable stress checking (LY2) and consonant texture analysis (LY5) to the lyrics quality pipeline. Catches lyrics that look good on paper but sound bad when sung.

## Gap

Current lyrics critics check:
- Cliche/dead metaphors (lyrics-intentionality)
- Filler/register (lyrics-intentionality)
- Volta/density/singability (lyrics-structural)
- Section purpose/hooks (lyrics-structural)

Missing:
- **Syllable stress vs. meter** — does "ex-TRA-or-di-NAR-y" land on the right beats?
- **Consonant clusters at wrong moments** — "clutched" in a whispered verse, "strengths" on a held note

## Approach

### Data: CMU Pronouncing Dictionary

The CMU dict maps English words to phoneme sequences with stress markers:
- `HELLO  HH AH0 L OW1` — stress on second syllable (1 = primary)
- `MORNING  M AO1 R N IH0 NG` — stress on first syllable

Source: `cmudict-simplified.json` from lyric-diagnostic, or download from CMU directly. ~130K words.

### Library: `src/libs/phonetics.ts`

Two core functions:

**`checkMeterStress(lyrics, bpm, timeSignature)`**
- Parse lyrics into lines, lines into words
- Look up each word's stress pattern in CMU dict
- For each line: compute expected strong beats from BPM/time signature
- Flag lines where primary stresses land on weak beats
- Output: list of flagged lines with suggested fixes (reorder words, swap synonyms)

**`checkConsonantTexture(lyrics, sectionEnergy)`**
- Parse lyrics into lines with section context ([Energy:] tags)
- For each word: extract consonant clusters (3+ consecutive consonants)
- Flag harsh clusters (`str`, `tch`, `nkth`, `ngths`) in low-energy/intimate sections
- Flag soft/breathy sounds (`sh`, `wh`, `mm`) in high-energy/aggressive sections
- Output: list of texture mismatches with severity

### Integration Points

| Where | How |
|-------|-----|
| `lyrics-structural.md` critic | Add LY2/LY5 check instructions — critic runs phonetics tool, includes flags in output |
| `lyrics-intentionality.md` critic | Add consonant texture awareness to register analysis |
| Rune agent (lyrics.md) | Add phonetic awareness to generation guidelines — avoid known bad patterns at write time |
| `/create-track` P3-005 | After lyrics are written, run phonetic check before tri-critic |
| `/lyrics` workflow | Run phonetic check on each variation before presenting to user |

### Scoring

Phonetic flags are **warnings, not hard fails**:
- **LY2 stress mismatch**: severity = number of mismatched stresses per line / total stresses
- **LY5 texture clash**: severity = harsh cluster count in soft sections + soft cluster count in hard sections
- Both feed into critic output as additional flags — tri-critic reconciliation decides accept/reject

### Limitations

- CMU dict is English-only — non-English lyrics skip phonetic checks
- CMU dict doesn't cover all words — unknown words are skipped, not flagged
- Stress analysis assumes standard meter — irregular/spoken-word styles may false-positive
- Consonant texture is heuristic — some "harsh" clusters are stylistically intentional

## Implementation Steps

1. Download/prepare CMU dict as JSON — store in `src/data/cmudict.json`
2. Build `src/libs/phonetics.ts` with lookup, stress check, texture check
3. Add CLI wrapper `src/tools/phonetics.ts` for critic integration
4. Update lyrics critics to call phonetics tool
5. Update Rune's agent file with phonetic awareness
6. Audit: verify wired into /create-track and /lyrics workflows
