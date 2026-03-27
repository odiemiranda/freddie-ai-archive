---
name: Suno Lyrics Technical Guidelines
description: Guidelines for using structural and mood tags in Suno lyrics, including reliability and usage limits for predictable AI outputs.
type: knowledge
agent: rune
tags: [lyrics, suno, structure-tags, mood-tags, reliability, constraints, emotion]
---

## Overview
Suno's vocal engine interprets specific metatags within lyrics to guide song structure and emotional delivery. Rune must use these tags strategically, prioritizing reliable ones and adhering to usage limits to ensure predictable and high-quality outputs.

## Structural Tags
- **Reliability:**
    - **High Reliability (Prioritize):** `[Chorus]` (95%), `[Verse]` (90%), `[Short Instrumental Intro]` (90%). These tags reliably influence song structure.
    - **Low Reliability (Avoid):** `[Intro]` (30-40%). This tag often yields unpredictable results and should be avoided.
- **Usage Limits:**
    - **Maximum 5-7 tags per song:** To maintain clarity and effectiveness, limit the total number of structural tags used in a single song. Overuse can confuse the AI.

## Mood Tags
- **Reliability:** `[Mood: X]` tags (e.g., `[Mood: Happy]`, `[Mood: Sad]`) have moderate reliability (75-85%).
- **Usage Limits:** Apply sparingly, with a maximum of 1-2 `[Mood: X]` tags per song. Overuse can dilute their impact or lead to inconsistent emotional delivery.

## Why
- **Predictable Outcomes:** Using high-reliability tags and adhering to limits ensures Suno generates music closer to the intended structure and mood.
- **Clarity & Effectiveness:** Overloading lyrics with too many or unreliable tags can lead to unpredictable or poor-quality generations.
- **AI Interpretation:** Understanding tag reliability helps Rune guide the AI effectively without fighting its limitations.

## How to Apply
1.  **Prioritize High-Reliability Structural Tags:** Always opt for `[Chorus]`, `[Verse]`, and `[Short Instrumental Intro]` when defining song sections.
2.  **Avoid Low-Reliability Tags:** Do not use `[Intro]` as it is largely ineffective.
3.  **Count Structural Tags:** Ensure the total number of structural tags (e.g., `[Verse]`, `[Chorus]`, `[Bridge]`, etc.) does not exceed 7 per song.
4.  **Use Mood Tags Sparingly:** If conveying a specific emotion, use `[Mood: X]` tags, but limit them to one or two per song.

**Consolidated from:**
- `suno-lyrics-metatags.md`
- `20260326-164106-rune-can-now-use-mood-x-tags-to-convey-e-2.md`
- `20260326-164106-rune-must-adhere-to-a-maximum-of-5-7-str-3.md`
- `20260326-164106-rune-must-prioritize-high-reliability-st-1.md`
---
