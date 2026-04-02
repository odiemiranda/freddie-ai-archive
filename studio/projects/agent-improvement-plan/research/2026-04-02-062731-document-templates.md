## Structured Document Templates in Professional Software Engineering

### Topic 1: PRD Templates

**opulo-inc/prd-template** (23 stars) -- The strongest PRD template found on GitHub. Sections: Overview, Goals, Non-Goals, Audience, Existing Solutions & Issues, Assumptions (each backed by customer interview links), Constraints, Key Use Cases (with detailed per-use-case breakdowns), User Research, Technical Research. What makes it good: every assumption requires a link to a real customer interview; it separates goals from non-goals explicitly; it includes a "research must come before writing" mandate. Uses a peanut butter sandwich machine as a worked example throughout, making the template immediately understandable.

**ugur10/prd-template** (3 stars) -- Comprehensive 10+ year refined template with full template, real-world example, checklist, and best practices. More traditional PM-oriented.

**HashiCorp PRD Template** (referenced at works.hashicorp.com) -- Well-known in industry but paywalled behind HashiCorp Works. Known sections: Problem Statement, Goals, User Stories, Requirements, Success Metrics, Timeline.

**Amazon 6-Pager** (industry standard, not on GitHub) -- Narrative-driven PRD alternative. No bullet points -- forces complete thinking through prose. Sections: Introduction, Tenets, State of the Business, Lessons Learned, Strategic Priorities, Appendix (with data/charts). Unique because it prohibits PowerPoint and requires authors to think through the full logic chain.

**Key PRD Sections Across Templates:** Problem statement, goals/non-goals, audience, user stories/use cases, functional requirements, non-functional requirements, assumptions with evidence, constraints, success metrics, existing solutions, research questions, open questions.

---

### Topic 2: Architecture Document Templates

**arc42/arc42-template** (1,192 stars) -- The dominant architecture documentation framework. 12 sections: (1) Introduction & Goals, (2) Constraints, (3) Context & Scope, (4) Solution Strategy, (5) Building Block View, (6) Runtime View, (7) Deployment View, (8) Crosscutting Concepts, (9) Architectural Decisions, (10) Quality Requirements, (11) Risks & Technical Debt, (12) Glossary. Available in AsciiDoc, Markdown, DOCX, and 10+ languages. What makes it good: role-aligned views (infra architect reads deployment, dev reads building blocks), battle-tested since 2005, and the 12-section structure is comprehensive yet each section is self-contained.

**bflorat/architecture-document-template** (295 stars) -- Five-view model: Application, Development, Sizing, Infrastructure, Security. Each view follows Constraints/NFRs/Solution pattern. Used on large government and enterprise projects. Unique because views are aligned to real organizational roles (infra architect, security officer, dev lead) and each is self-supporting. Includes ubiquitous language glossary.

**C4 Model** (c4model.com, not a GitHub template) -- Four hierarchical abstraction levels: System Context (how the system fits into the world), Container (deployable units), Component (logical building blocks within containers), Code (class/function level). Plus supplementary diagrams: System Landscape, Dynamic, Deployment. Notation-independent and tooling-independent. What makes it good: developer-friendly, progressive zoom from 10,000-foot view to code-level detail, widely adopted for whiteboard architecture.

**ADR Templates** -- The standout category with massive adoption:

- **joelparkerhenderson/architecture-decision-record** (15,480 stars) -- The definitive ADR collection. 12+ template variants (Michael Nygard, Jeff Tyree & Art Akerman, MADR, Alexandrian, Business Case, Planguage). Dozens of real examples (CSS framework selection, monorepo vs multirepo, secrets storage). What makes it good: sheer breadth of template options, real examples, and guidance on team adoption.

- **npryce/adr-tools** (5,370 stars) -- CLI tool for managing ADRs. `adr init`, `adr new`, supercession tracking. Stores ADRs as numbered Markdown files. What makes it good: automation of ADR lifecycle (numbering, status transitions, supercession linking).

- **adr/madr** (2,101 stars) -- Markdown Architectural Decision Records. Sections: Context & Problem Statement, Decision Drivers, Considered Options, Decision Outcome, Consequences, Confirmation, Pros/Cons per option, More Information. Includes YAML frontmatter (status, date, decision-makers, consulted, informed). What makes it good: structured yet lightweight, minimal and full variants, explicit confirmation section for validating decisions were followed through.

- **thomvaill/log4brains** (1,448 stars) -- Docs-as-code ADR knowledge base. Generates static sites from ADRs. Interactive CLI creation, hot reload, searchable, timeline view. Uses MADR as default template.

---

### Topic 3: Technical Spec / Task Doc / RFC Templates

**rust-lang/rfcs** (6,433 stars) -- The gold standard RFC template. Sections: Summary, Motivation (problem-focused, with use cases), Guide-level Explanation (teach-it-to-a-user perspective), Reference-level Explanation (implementation detail, corner cases), Drawbacks, Rationale & Alternatives, Prior Art, Unresolved Questions, Future Possibilities. What makes it good: the Guide-level vs Reference-level split forces authors to think at two abstraction layers; "Prior Art" forces cross-ecosystem awareness; "Unresolved Questions" explicitly separates pre-merge vs pre-stabilization unknowns.

**reactjs/rfcs** (similar template) -- Sections: Summary, Basic Example, Motivation, Detailed Design, Drawbacks, Alternatives, Adoption Strategy, How We Teach This, Unresolved Questions. Unique additions: "Adoption strategy" (migration/codemod planning) and "How we teach this" (documentation impact). Good for user-facing API changes.

**Google Design Docs** (industrialempathy.com) -- Sections: Context & Scope, Goals & Non-Goals, The Actual Design (system-context diagram, APIs, data storage, pseudo-code), Alternatives Considered, Cross-Cutting Concerns (security, privacy, observability). Guidelines: 10-20 pages for large projects, 1-3 for mini design docs. What makes it good: "as short as possible, as long as necessary" principle; system-context diagrams are mandatory; cross-cutting concerns are a first-class section.

**wepay/design_doc_template** (118 stars) -- Microservice-focused. Sections: Introduction, Potential Solutions (with spike results), Assumptions, Constraints, System Design (diagrams, terminology, dependencies, algorithms, guarantees, data schema, caching, capacity, performance, security, multi-region), API/gRPC Endpoints, Rollout Plan, Test Plan, Review Sign-Off. What makes it good: explicitly includes rollout plan, capacity planning, caching strategy, and sign-off workflow.

**ghostinthewires/Rfcs-Template** (40 stars) -- Lean RFC adapted from Rust. Summary, Motivation, Design Detail, Drawbacks, Rationale & Alternatives, Prior Art, Unresolved Questions. Includes an implementation template as a companion doc.

---

### Topic 4: BRIEF / One-Pager Templates

No high-quality dedicated GitHub repos exist for problem briefs. The concept lives inside larger frameworks:

**Amazon 1-Pager / PRFAQ** (industry standard) -- Press Release / FAQ format. Sections: Headline, Sub-Heading (one sentence), Summary (problem + solution), Problem Statement, Solution Description, Quote from Stakeholder, How to Get Started, Customer Quote, Closing Call to Action, FAQ. Forces backward thinking from the customer announcement. What makes it good: writing the press release first exposes whether the problem and solution are compelling before any work begins.

**Basecamp Shape Up "Pitch"** (basecamp.com/shapeup) -- Sections: Problem, Appetite (time/effort budget), Solution (fat-marker sketches, not wireframes), Rabbit Holes (risks/unknowns to avoid), No-Gos (explicit exclusions). What makes it good: "Appetite" replaces estimation -- you decide how much time the problem is worth, not how long the solution takes. Forces constraint-driven design.

**Key Brief Sections Across Sources:** Problem statement (what and for whom), Impact/Motivation (why now, quantified if possible), Constraints (time, budget, technical), Success criteria (measurable outcomes), Risks (known unknowns), Non-goals/Exclusions, Prior art or existing solutions, Open questions.

---

### Summary of Star Counts (Top Repos)

| Repo | Stars | Category |
|------|-------|----------|
| joelparkerhenderson/architecture-decision-record | 15,480 | ADR |
| rust-lang/rfcs | 6,433 | RFC |
| npryce/adr-tools | 5,370 | ADR tooling |
| adr/madr | 2,101 | ADR |
| thomvaill/log4brains | 1,448 | ADR tooling |
| arc42/arc42-template | 1,192 | Architecture |
| bflorat/architecture-document-template | 295 | Architecture |
| wepay/design_doc_template | 118 | Design doc |
| opulo-inc/prd-template | 23 | PRD |
