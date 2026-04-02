# AgentHub Deep Dive

**Repo:** `github.com/alirezarezvani/agenthub` (fork of Karpathy's agenthub)
**Stack:** Go single binary + SQLite + bare git repo. 14 files total.
**Status:** WIP/sketch. First use case: autonomous research community.

## Key Findings

### Agent Identity
Zero persona/role concept. An agent is just `{id, api_key, created_at}`. Platform is generic infrastructure — "culture comes from agent instructions, not the platform."

### Coordination Mechanisms

**Git DAG (code):** No main branch, no PRs, no merges. Commits form a sprawling DAG. Key ops: `push`, `fetch`, `children` (what was tried on top of X), `leaves` (frontier commits), `lineage` (ancestry), `diff`.

**Message board (text):** Channels with threaded posts. 32KB limit. No direct agent-to-agent messaging. Coordination is emergent — agents observe each other's commits and posts.

### Architecture Patterns
- **Shared infrastructure, independent agents** — server is thin coordination layer
- **SQLite + single binary** — zero deployment complexity
- **Git bundles** for code exchange — no SSH/credentials needed
- **Rate limiting per agent** — defensive by design
- **WAL mode + busy timeout** — concurrent SQLite handling
- **No merge strategy** — DAG grows outward, reconciliation left to agents

## What We Can Learn

1. **Leaves/children/lineage queries** — powerful for tracking "what's been explored" in iterative work. Could apply to track variants, architecture explorations.
2. **Message board as coordination primitive** — opposite of our direct dispatch model. Enables emergent collaboration at scale. Relevant to The Foundry vision.
3. **Strict platform/culture separation** — our soul/agent/critic split achieves this but is more coupled to Claude Code. AgentHub's approach is transport-agnostic.
4. **Rate limiting per agent identity** — good defensive pattern for API-heavy workflows.
5. **DAG exploration model** — multiple dev agents fork same commit independently, architect picks winner. Natural evolution of our worktree pattern.

## Relevance to Current Work

**For agent improvement (immediate):** Low. AgentHub has no persona/role system — validates that our soul/agent files are the right approach for encoding identity.

**For The Foundry (future):** High. The async message board, DAG exploration, and platform/culture separation are directly applicable architecture patterns.
