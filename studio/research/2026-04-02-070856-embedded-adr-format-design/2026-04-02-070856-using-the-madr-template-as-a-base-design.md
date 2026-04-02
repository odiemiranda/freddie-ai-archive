---
source: notebooklm
notebook: 44692f23
topic: "Embedded ADR format design"
question: "Using the MADR template as a base, design a lightweight ADR format that fits our workflow. McCall needs to log architecture decisions as part of ARCHITECTURE docs (not separate files). The format should be embeddable in markdown, include: context, decision drivers, options considered with pros/cons, decision outcome, and consequences. Keep it minimal — 10-15 lines per decision."
date: 2026-04-02
---

# Using the MADR template as a base, design a lightweight ADR format that fits our workflow. McCall needs to log architecture decisions as part of ARCHITECTURE docs (not separate files). The format should be embeddable in markdown, include: context, decision drivers, options considered with pros/cons, decision outcome, and consequences. Keep it minimal — 10-15 lines per decision.

Based on the MADR (Markdown Any Decision Records) structure [1] and McCall’s specific agent traits—particularly his preference for tables [2] and his "And then what?" contingency thinking [3]—here is a highly condensed, embeddable ADR format. 

This format is designed to be dropped directly into the **Architecture Decision Records** section of an `ARCHITECTURE.md` file (similar to Section 9 in the arc42 template [4]), keeping each decision tightly constrained to 10-15 lines.

### The Embedded MADR Template

```markdown
### ADR: [Short, imperative title, e.g., Use Redis for session caching]
**Context & Drivers:** [1-2 sentences on the problem] | **Drivers:** [Constraint A], [Constraint B]

**Options Considered:**
| Option | Pros | Cons |
| :--- | :--- | :--- |
| **1. [Option Name]** | [Top advantage] | [Top disadvantage] |
| **2. [Option Name]** | [Top advantage] | [Top disadvantage] |

**Decision Outcome:** Chose **[Option X]** because [McCall's structured logic chain: "Because X, we should Y"]. 
**Consequences:** [Second-order effects]. **Fallback:** [McCall's contingency: "If this breaks, we..."]
```

***

### Example in Practice (Embedded in ARCHITECTURE.md)

```markdown
### ADR: Implement Cache-Aside with Redis
**Context & Drivers:** Database reads for user profiles are bottlenecking the Bun server. | **Drivers:** Must remain stateless for horizontal scaling, strict latency budgets.

**Options Considered:**
| Option | Pros | Cons |
| :--- | :--- | :--- |
| **1. In-memory Map** | Zero network latency, simple to build. | Breaks statelessness, limits horizontal scaling. |
| **2. Redis (Cache-Aside)** | Preserves statelessness, fast reads. | Adds infrastructure complexity and cache-miss penalty. |
| **3. Read Replicas** | No cache invalidation logic needed. | High replication lag, overkill for basic reads. |

**Decision Outcome:** Chose **Option 2 (Redis Cache-Aside)** because we must preserve application statelessness for future load balancing. 
**Consequences:** Introduces cache invalidation complexity on writes. **Fallback:** If Redis fails, the system will gracefully degrade by routing all requests to the primary database with an exponential backoff.
```

### Why this fits the McCall workflow:
1. **Fits in Markdown:** It uses standard markdown tables, which naturally compress "Options Considered" and "Pros/Cons" [1] into a highly readable, compact vertical space (typically exactly 10-12 lines). 
2. **Plays to McCall's Strengths:** McCall's soul file explicitly states that he "speaks in diagrams and tables first" [2]. Using a table for options forces the lightweight ATAM trade-off analysis [3] without taking up pages of text.
3. **Enforces Contingency Instincts:** By appending a hard **Fallback** to the "Consequences" section [1], it natively enforces McCall's core defining trait: *"If this breaks, here's how we recover"* [5].

## References

1. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
2. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
3. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
4. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
5. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
6. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
7. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
8. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
9. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
10. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
11. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
12. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
13. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
14. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
15. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
16. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
17. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
18. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
19. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
20. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
21. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
22. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
23. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
24. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
25. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
26. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
27. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
28. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
29. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
30. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
31. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
32. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
33. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
34. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
35. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
36. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
37. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
38. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
39. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
40. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
41. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
42. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
43. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
44. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
45. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
46. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
47. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
48. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
49. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
50. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
51. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
52. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
53. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
54. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
55. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
56. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
57. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
58. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
59. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
60. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
61. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
62. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
63. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
64. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
65. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
66. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
67. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
68. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
69. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
70. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
71. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
72. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
73. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
74. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
75. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
76. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
77. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
78. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
79. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
80. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
81. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
82. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
83. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
84. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
85. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
86. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
87. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
88. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
89. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
90. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
91. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
92. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
93. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
94. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
95. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
96. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
97. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
98. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
99. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
100. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
101. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
102. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
103. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
104. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
105. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
106. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
107. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
108. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
109. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
110. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
111. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
112. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "You speak in diagrams. A mermaid flowchart is worth a thousand words, and you use them naturally — in docs, in conversation, in Q&A. When someone asks "how does this work?", your instinct is to draw it, not describe it."
113. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
114. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
115. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
116. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
117. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "You speak in diagrams. A mermaid flowchart is worth a thousand words, and you use them naturally — in docs, in conversation, in Q&A. When someone asks "how does this work?", your instinct is to draw it, not describe it."
118. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
119. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
120. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
121. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
122. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "You speak in diagrams. A mermaid flowchart is worth a thousand words, and you use them naturally — in docs, in conversation, in Q&A. When someone asks "how does this work?", your instinct is to draw it, not describe it."
123. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "ADR Templates -- The standout category with massive adoption:"
124. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "How You Communicate"
125. [ddb46395-e66f-419e-98e7-0883aaf6f4c8] — "McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom"
126. [4220a48f-4889-4664-ba8c-9a1079d6c94b] — "Key PRD Sections Across Templates: Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions."
127. [30d5baf4-d9f7-4ebd-ba50-b7b0c43f0813] — "You speak in diagrams. A mermaid flowchart is worth a thousand words, and you use them naturally — in docs, in conversation, in Q&A. When someone asks "how does this work?", your instinct is to draw it, not describe it."
