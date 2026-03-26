---
name: SDK architecture research — Gemini + MiniMax direct integration
description: Deep dive research on replacing Gemini CLI with SDK, adding MiniMax API, multi-soul agent expansion
type: project
agent: freddie
tags: [gemini, minimax, sdk, architecture, multimodal, pricing]
---

Completed research on replacing Gemini CLI with direct SDK integration. Key findings:

- **Gemini SDK** (`@google/genai`): TypeScript-native, Bun-compatible, supports audio/image/video input. Free tier: 250 RPD for Flash. Can analyze music card audio directly (5 min = ~9,600 tokens).
- **MiniMax API**: No SDK, but OpenAI-compatible REST. Has music generation API (`/v1/music_generation`) — can generate tracks programmatically. No audio input capability.
- **LiteLLM**: Rejected — overkill for 2 providers, MiniMax music API isn't a chat endpoint.

Architecture: separate `libs/gemini.ts` + `libs/minimax.ts` + shared `libs/ai-config.ts`. Multi-soul pattern for both agents (Echo/Sonic/Iris/Veo for Gemini, Sol/Forge for MiniMax). Price tracking via `libs/ai-usage.ts`.

**Why:** Gemini CLI has Windows issues (session accumulation, timeouts, stdin piping). Direct SDK is pure HTTP — no CLI binary, no process spawning. Also unlocks multimodal (audio analysis for music cards, image gen, video gen).

**How to apply:** Idea doc at `vault/notes/inbox/studio-expansion-sdk-architecture.md`. Phase 1 is Gemini SDK (solves immediate Windows issues). MiniMax and multi-soul expansion are future phases.
