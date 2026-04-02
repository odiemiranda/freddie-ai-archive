---
name: Memory System Architecture and Management
description: Comprehensive guidelines and mechanisms for managing agent memory, ensuring knowledge integrity, optimizing context, and leveraging advanced retrieval and representation features.
type: knowledge
agent: freddie
tags: [memory, compaction, knowledge_integrity, confidence, state_management, debugging, infrastructure, workflow_management, optimization, reflection, entities, knowledge_graph, relationships, knowledge_representation, vector_search, embeddings, gemini, retrieval, FTS5, architecture, MCP, consolidation, project_management, context_management, brain.db, tool, checkpoints, HTTP, integration, standardization, context_capture, automation]
---

## Strategic Memory Compaction
- **Automatic Triggers:** Memory compaction can now be initiated automatically when tool usage reaches predefined thresholds (e.g., 50 and 80 tool calls).
- **Time-Based Triggers (MCP Server):** Proactive memory management is enhanced by specific time-based triggers within the MCP server: 5-minute reflection, 30-minute knowledge consolidation, and immediate flushing.
- **Pre-Compaction State Save:** Before each memory compaction, the system saves current tool call counts, compaction counts, and timestamps to `out/session-state.json`.
- **Operational Status:** These features are fully implemented and integrated, with pre-compaction state saving operational.

## Knowledge Integrity
- **Contradiction Detection:** Before writing new learnings, the system proactively identifies potential contradictions with existing knowledge-tier files using FTS5 and marks them with a `contradicts:` field in the frontmatter.
- **Operational Status:** Contradiction tracking in `reflect.ts` is live and functional.

## Confidence Decay
- **Dynamic Adjustment:** The system dynamically adjusts the perceived reliability of past decisions, with confidence linearly decaying over 90 days to a minimum of 30%.
- **Operational Status:** Operational confidence decay on `brain.db` decisions is fully implemented.

## Memory Categorization (Typed Memories)
- **Explicit Types:** The memory system now supports explicit memory types: 'episodic', 'semantic', and 'entity'.
- **System-wide Integration:** This categorization is integrated across reflection prompts, the Learning interface, inbox frontmatter, the `brain.db` memories table, sync extraction, and query filters.
- **Default:** 'semantic' is the default type for backwards compatibility.

## Semantic Search (Vector Embeddings)
- **Vector Search Integration:** Memory retrieval now incorporates vector search using Gemini embeddings (`gemini-embedding-001`).
- **Hybrid Ranking:** A hybrid FTS5+vector ranking (0.4/0.6 weights) is used for improved semantic understanding and recall.

## Knowledge Graph (Entity Tracking)
- **Entity & Relation Tables:** Freddie gained entity tracking capabilities with new 'entities' and 'entity_relations' tables in `brain.db`.
- **Typed Relations:** Supports upsert by name and typed relations including `used_with`, `related_to`, `part_of`, and `conflicts_with`.
- **Project-Level Context Management:** Freddie can automatically identify, tag, and register projects as entities, storing their paths in a `project_path` column in `brain.db` and using similarity detection. This enables robust project-aware context management.
- **Weight Reinforcement:** Duplicate entries reinforce entity weights.
- **CLI Management:** CLI commands are available for lookup, search, add, and link operations.

## Automatic Reflection Context Capture
- **Functionality:** The reflection mechanism now reliably performs automatic raw context capture for workflows. This ensures comprehensive session transcripts are consistently available as input for subsequent reflection and memory creation processes.
- **Implementation:** The `reflect.ts` engine has been updated to automatically capture raw context when the `--raw-ref` argument is not explicitly provided.
- **Benefit:** This standardizes and improves the completeness and reliability of reflection inputs without requiring manual specification.
- **Operational Status:** This functionality is confirmed to be working and is fully operational.

## Memory Tools and External Access
- **`agent_save` Tool:** A new `agent_save` tool is available for creating lightweight, write-only checkpoints, improving state management.
- **HTTP Transport:** Core memory management tools are now accessible externally via HTTP. External client access to Freddie's Memory Compaction Protocol (MCP) is exclusively via HTTP, establishing a standardized and simplified integration method.

**Why:** These features significantly improve system efficiency by optimizing context, enhance reliability by preserving state for debugging, and maintain knowledge quality by flagging inconsistencies and prioritizing more recent or frequently reinforced information. Proactive memory management, robust project-aware context, and flexible external integration further enhance the system's capabilities. The advanced memory features enable more sophisticated knowledge representation, highly relevant semantic retrieval, and a deeper understanding and reasoning about interconnected concepts, leading to more effective and nuanced decision-making. Standardizing external access via HTTP for MCP ensures consistent and simplified integration. The automatic raw context capture specifically improves the completeness and reliability of reflection inputs, feeding into these advanced memory processes.

**How to apply:**
- Rely on automatic compaction triggers (both tool-usage and time-based via MCP server) for context optimization based on activity levels and scheduled maintenance.
- Utilize the `out/session-state.json` file for debugging, tracking compaction history, and understanding workflow context around memory optimizations.
- Review learnings flagged with `contradicts:` for potential reconciliation or deeper investigation into conflicting knowledge.
- Understand that the system's decisions will prioritize more recent or reinforced knowledge due to confidence decay, making them more contextually relevant.
- When reflecting, assign the most appropriate `memoryType` (episodic, semantic, entity) to new learnings for structured storage and targeted retrieval.
- Leverage vector search for more effective memory retrieval, especially when keyword search is insufficient or for conceptual/nuanced queries.
- Actively identify and track relevant entities and their relationships, including project-level context via `detectProject()`, in future workflows to build out the knowledge graph, enhancing understanding and reasoning.
- Trust that raw context for reflections is now automatically captured, providing comprehensive input for memory creation without manual intervention.
- Utilize the `agent_save` tool for lightweight state checkpoints.
- Leverage HTTP access for integrating Freddie's memory management with external systems, specifically noting that MCP access is HTTP-only.

*Consolidated from: knowledge_memory_and_integrity_management.md, 20260329-213436-the-agent-s-memory-system-now-actively-m-2.md, 20260330-040626-freddie-can-now-build-and-manage-a-knowl-3.md, 20260330-040626-freddie-can-now-perform-highly-relevant--2.md, 20260330-040626-freddie-now-categorizes-its-memories-int-1.md, 20260330-115541-freddie-s-memory-system-now-leverages-sp-1.md, 20260330-115541-freddie-can-now-automatically-identify-t-2.md, 20260330-115541-freddie-gained-a-new-agent-save-tool-for-3.md, 20260401-080142-external-client-access-to-freddie-s-memo-2.md, 20260401-081158-the-system-s-reflection-mechanism-now-re.md, 20260401-081439-the-reflection-process-now-automatically-1.md*
