---
source: reflection
agent: tala
task: "morning shamisen jazz — harp layering caused by genre label"
workflow: suno-generation
outcome: partial
scope: shared
confidence: high
tags: [genre, suno, fusion, culture-as-genre, instrument-flooding]
session: 2026-03-26T02:00:00
---

## Observation
Track "Morning Stillness" had persistent harp layering problems despite aggressive constraints. Root cause was "Japanese folk fusion" in the Style genre label. Suno interpreted this as "bring the full Japanese folk instrument palette" — adding koto-like tones, additional string layers, and traditional percussion patterns on top of the explicitly requested instruments.

## Learning
Never use a culture, tradition, or region as a genre label in Suno Style blocks. Suno treats these as "add everything from this tradition" rather than "blend subtly." The specific instrument in the prompt already signals the cultural flavor — naming the culture on top doubles the signal and floods the output.

Pattern: `{rhythm-template} {instrument} fusion` — e.g., "Jazz shamisen fusion" not "Japanese folk fusion." The genre gives Suno the rhythm template (jazz = swing, brushed drums). The instrument gives the cultural color (shamisen = Japanese). No need to say "Japanese" when shamisen is already there.

Tested: changing "Lo-fi jazz, Japanese folk fusion" to "Jazz shamisen fusion" fixed the layering issue.

## Evidence
- Broken: "Lo-fi jazz, Japanese folk fusion" → Suno added unwanted koto-like layers and traditional percussion
- Fixed: "Jazz shamisen fusion" → clean output, only requested instruments
- Working reference: `tracks/2026-03-25/161416-morning-window-groove/first-sip.md` used "Jazz fusion" with shamisen as an instrument, no culture label — no layering issues
