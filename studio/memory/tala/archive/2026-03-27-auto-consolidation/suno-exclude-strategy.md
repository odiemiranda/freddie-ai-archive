---
name: suno-exclude-strategy
description: Minimal exclude strategy for ALL tracks when using typed [Instrument:] brackets in Lyrics.
type: knowledge
agent: tala
tags: [suno, exclude, instruments, hallucination, typed-brackets, unified]
---

# Suno Exclude Style Strategy

## Primary: Minimal or empty (with typed brackets in Lyrics)

### Why
When every section has `[Instrument:]` typed brackets in the Lyrics, Suno has positive per-section instructions and doesn't hallucinate unwanted instruments. This positive guidance replaces the need for extensive negative exclusions. Tested: empty Exclude + typed brackets produced clean output across multiple generations for both instrumental and vocal tracks.

### How to apply
Only add dealbreakers (e.g., auto-tune for raw vocal, electronic for acoustic genres). Otherwise, leave the Exclude Style block empty.

## What Fails (Avoid)

### Why
Adding abstract terms confuses Suno, as it struggles to interpret "what NOT to feel" compared to "what instrument NOT to use." These are mood/energy words Suno tries to interpret negatively, which is difficult for the model.

### How to apply (Avoid these)
Do not add abstract terms like: `aggressive, uptempo, crescendo, dramatic build, drop, bright melody, distortion`.

---
*Consolidated from: `suno-exclude-strategy.md` (original), `typed-brackets-replace-excludes.md` (parts), `unified-typed-brackets.md`*
