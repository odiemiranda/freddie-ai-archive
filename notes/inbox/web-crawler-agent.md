---
id: idea-20260323
title: Web Crawler Agent
tags: [idea, agent, crawler, automation]
created: 2026-03-23
status: brainstorm
---

# Web Crawler Agent

Authenticated web crawler that can browse Medium, YouTube, LinkedIn, TikTok using session credentials.

## Use Cases (TBD)
- Extract YouTube analytics / video performance
- Browse Medium drafts or published content
- LinkedIn post analytics
- TikTok content/trend research
- Cross-platform content repurposing

## Approach Options

1. **API-first** — YouTube Data API, TikTok Business API (no Medium API exists)
2. **Browser automation** — Playwright with exported cookies for platforms without APIs
3. **Hybrid** — APIs where available, browser for the rest

## Security
- Cookies/credentials in `out/` (gitignored), never in git
- Read-only access — scrape data, don't post or modify
- Rate limiting built in

## Architecture (rough)
- Agent: `.claude/agents/crawler.md`
- MCP route: `servers/freddie-ai/routes/crawler.ts`
- Credentials: `out/crawler-cookies.json` (gitignored)
- Cache: `out/crawler-cache/` (gitignored)

## Status
Idea only. Revisit later. Need to define primary use case first.
