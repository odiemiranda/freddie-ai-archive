---
title: Studio Expansion — SDK Architecture
created: 2026-03-25
tags: [idea, architecture, gemini, minimax, sdk, multimodal, pricing]
status: in-progress
---

## Overview

Replace the Gemini CLI with direct SDK integration and expand to multi-provider, multi-modal agent architecture. Gemini handles understanding (input), MiniMax handles creation (output), Claude orchestrates.

## Provider Capabilities (Researched 2026-03-25)

### Gemini (@google/genai)
- **Package:** `@google/genai` v1.46.0 (TypeScript-native, Bun-compatible)
- **Audio IN:** MP3, WAV, FLAC, AAC, OGG — up to 9.5hrs. 5 min = ~9,600 tokens
- **Image IN:** JPEG, PNG, GIF, WebP. Inline base64 or Files API
- **Image OUT:** Imagen 4 (via API)
- **Video OUT:** Veo 3 (via API)
- **Text:** 1M token context, streaming, system prompts
- **Free tier:** Flash: 10 RPM, 250K TPM, 250 RPD. Pro: 5 RPM, 100 RPD
- **Key:** Google AI Studio (aistudio.google.com)

### MiniMax (REST API)
- **No official SDK** — OpenAI-compatible REST endpoints
- **Music:** `POST /v1/music_generation` — model `music-2.5+`, prompt + lyrics, structure tags, MP3/WAV output ($0.15/track)
- **Text:** M2.5 ($0.30/M in, $1.20/M out), M2.7 ($0.30/M in, $1.20/M out)
- **Image:** `POST /v1/image_generation` — model `image-01`, text-to-image, subject reference ($0.0035/image)
- **Video:** `POST /v1/video_generation` — Hailuo models, async polling, 6-10s clips ($0.19-$0.56/video)
- **Audio OUT only** — cannot analyze audio input
- **Key:** platform.minimax.io

### LiteLLM (rejected)
- Overkill for 2 providers. MiniMax music API isn't a chat endpoint. Docker adds complexity with no payoff.

## Architecture

### Libs (shared infrastructure)
```
libs/
  ai-config.ts     ← API keys, model registry, error types, retry logic
  ai-usage.ts      ← per-call logging, cost estimation, daily/monthly summaries
  gemini.ts         ← Gemini SDK wrapper (text, audio, image, video)
  minimax.ts        ← MiniMax REST client (music, chat, image, video)
```

### Gemini Multi-Soul Agent
```
.claude/agents/gemini.md          ← router (picks soul based on request modality)
.claude/souls/
  echo.md                         ← music critique (text) — existing
  sonic.md                        ← audio analysis (audio input → text)
  iris.md                         ← image analysis / Imagen 4 generation
  veo.md                          ← Veo 3 video generation
```

### MiniMax Multi-Soul Agent
```
.claude/agents/minimax.md         ← router (picks soul based on product)
.claude/souls/
  sol.md                          ← music prompt engineering (text) — existing
  forge.md                        ← music generation API (prompt → audio)
```

### Price Per Token Tracking — DONE (2026-03-25)

| Provider | Model | Input/1M | Output/1M | Free tier |
|----------|-------|----------|-----------|-----------|
| Gemini | 2.5 Flash | Free | Free | 250 RPD |
| Gemini | 2.5 Pro | Free | Free | 100 RPD |
| MiniMax | M2.5 | $0.20 | $0.95 | No |
| MiniMax | M2.7 | $0.30 | $1.20 | No |
| MiniMax | Music 2.5+ | Per gen | — | No |
| Claude | Opus 4.6 | $15.00 | $75.00 | Sub |
| Claude | Sonnet 4.6 | $3.00 | $15.00 | Sub |

~~Usage logged per-call to `vault/studio/usage/` — model, tokens, cost. Penny can surface daily costs in notes.~~ Done. `libs/ai-usage.ts` tracks per-call, `tools/usage.ts` provides summaries. Gemini calls auto-tracked via `libs/gemini.ts`. MiniMax tracking ready when `libs/minimax.ts` is built.

## Migration Plan

### Phase 1: Gemini SDK ~~(solves Windows issues)~~ DONE (2026-03-25)
1. ~~`bun add @google/genai`~~ done
2. ~~Build `libs/gemini.ts` with `generateText()`~~ done (`analyzeAudio()`, `analyzeImage()` deferred to Phase 3)
3. ~~Migrate `consolidate-memory.ts` and `musiccard.ts` to use SDK~~ done
4. ~~Keep `gemini-proxy.ts` as fallback during transition~~ skipped — went straight to Phase 4

### Phase 2: MiniMax API — DONE (2026-03-25)
1. ~~Build `libs/minimax.ts` with `generateMusic()`, `chatCompletion()`~~ done
2. ~~Add `generateImage()` and `generateVideo()` to `libs/minimax.ts`~~ done — all four capabilities in shared library
3. ~~CLI tools (`tools/minimax/`) refactored to use library functions~~ done — music, chat, image, video
4. Create Forge soul — programmatic track generation (deferred to Phase 3)
5. Test: generate a track from a music card's MiniMax Translation field (deferred to Phase 3)

### Phase 3: Multi-Soul Expansion
1. Sonic soul — send music card audio to Gemini for subjective analysis
2. Iris soul — album art / cover generation (MiniMax `generateImage` ready)
3. Veo soul — music video generation (MiniMax `generateVideo` ready)
4. ~~Usage tracking across all providers~~ done — `libs/ai-usage.ts` tracks Gemini, MiniMax, Claude

### Phase 5: Self-Learning Agents — DONE (2026-03-25)
See `vault/notes/inbox/self-learning-agents.md` for full design doc.
1. ~~Post-task reflection engine~~ done — `libs/reflect.ts` + `tools/reflect.ts`
2. ~~Auto-consolidation with threshold~~ done — `consolidate-memory.ts --auto`
3. ~~Soul distillation tool~~ done — `tools/distill-soul.ts`
4. ~~Agent workflow hooks~~ done — Tala, Sol, Rune, Echo all reflect after workflows
5. Learning dashboard (TODO)
6. Consolidation scheduling (TODO)

### Phase 4: Retire Gemini CLI — DONE (2026-03-25)
1. ~~Migrate remaining Echo callers to `libs/gemini.ts`~~ done — Echo now calls `tools/gemini.ts` directly
2. ~~Remove `gemini-proxy.ts` and `gemini` CLI dependency~~ done — also removed `execa` dependency
3. Echo handles discovery and prompt assembly itself (smarter agent, dumber tool)
4. Agent directives (`.gemini/agents/*.md`) loaded as system prompts via `--system-file`

## Key Design Decisions

- **Separate modules, not unified abstraction** — Gemini (understanding) and MiniMax (creation) don't share an interface
- **Context loading stays in calling code** — provider libs accept strings/files, return responses. Don't couple to vault structure
- **API keys in .env** — Bun reads automatically, already gitignored
- **Router pattern for multi-soul** — agent file picks soul based on request type, loads soul's system prompt
