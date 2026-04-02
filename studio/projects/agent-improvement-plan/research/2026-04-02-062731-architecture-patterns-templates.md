# McCall Upgrade Research: Architecture Patterns, Templates, and Community Wisdom

## Topic 1: What Separates a Principal Architect from a Mid-Level One

**Systems thinking over feature thinking.** Mid-level architects ask "how do I implement this?" Principal architects ask "how does this fit the broader ecosystem of problems we're solving?" The shift is temporal -- optimizing for the code someone will change tomorrow, not the code you're writing today.

**Six mental models principals carry instinctively:**

1. **Second-order thinking** -- "And then what?" Go two or three levels deep on every decision. If we pick SQLite now, what happens when we need concurrent writes? What happens when we need replication?
2. **Boundaries and coupling intuition** -- Things that change together go together. Things that don't, get a boundary. Principals *feel* coupling before they see it in code. They draw the boundary before it becomes tech debt.
3. **Reversibility classification** -- Reversible decisions move fast. Irreversible ones get evidence, ADRs, and council input. This maps directly to McCall's existing instinct ("When a decision feels reversible, you move fast").
4. **Constraint-first design** -- What can't change? What's the hard ceiling? Constraints are the foundation. DDD bounded contexts, hexagonal port/adapter boundaries, and CQRS read/write splits are all ways to express constraints as architecture.
5. **Trade-off articulation** -- ATAM (Architecture Tradeoff Analysis Method) from CMU/SEI formalizes this: gather stakeholders, extract quality attributes, create scenarios, analyze trade-offs and sensitivity points. Principals do a lightweight version of this instinctively for every significant decision.
6. **Operational empathy** -- Observability, resilience, graceful degradation. A principal asks "how do we know it's broken?" and "what happens when it is?" before writing the happy path.

**Patterns principals reach for:** DDD for domain complexity. Hexagonal/ports-and-adapters for isolation from external systems. CQRS when read and write models diverge. Event sourcing when audit trails and temporal queries matter. The key insight: these aren't dogma -- they're tools selected based on the specific forces at play.

**Documentation as thinking tool:** C4 model (Context, Container, Component, Code) for progressive zoom. Arc42 for comprehensive templates covering quality requirements, crosscutting concerns, and risks. ADRs for decision lineage. Principals treat documentation as a design activity, not a post-hoc chore.

Sources: [Explicit Architecture (Graca)](https://herbertograca.com/2017/11/16/explicit-architecture-01-ddd-hexagonal-onion-clean-cqrs-how-i-put-it-all-together/), [ATAM (SEI/CMU)](https://www.sei.cmu.edu/library/architecture-tradeoff-analysis-method-collection/), [7 Mental Models (DEV)](https://dev.to/_b8d89ece3338719863cb03/7-mental-models-that-made-me-a-better-software-architect-30d8), [Cerbos Best Practices](https://www.cerbos.dev/blog/best-practices-of-software-architecture)

---

## Topic 2: GitHub Repos with Architecture/Design Templates

| Repo | Stars | What It Offers |
|------|------:|----------------|
| [donnemartin/system-design-primer](https://github.com/donnemartin/system-design-primer) | 340k | Scalability patterns, system design walkthroughs, trade-off analysis examples |
| [DovAmir/awesome-design-patterns](https://github.com/DovAmir/awesome-design-patterns) | 46.6k | Curated list of design patterns across languages, cloud, DevOps, and architecture |
| [joelparkerhenderson/architecture-decision-record](https://github.com/joelparkerhenderson/architecture-decision-record) | 15.5k | ADR examples, multiple templates (Nygard, MADR, Y-Statements), adoption guides |
| [mehdihadeli/awesome-software-architecture](https://github.com/mehdihadeli/awesome-software-architecture) | 10.8k | Curated resources on DDD, CQRS, microservices, event sourcing, hexagonal |
| [C4-PlantUML](https://github.com/plantuml-stdlib/C4-PlantUML) | 7.2k | C4 model diagram library for PlantUML -- Context, Container, Component, Code |
| [npryce/adr-tools](https://github.com/npryce/adr-tools) | 5.4k | CLI for managing ADRs -- init, new, link, generate TOC |
| [adr/madr](https://github.com/adr/madr) | 2.1k | Markdown Any Decision Records -- minimal, full, and bare templates |
| [thomvaill/log4brains](https://github.com/thomvaill/log4brains) | 1.4k | ADR management + static site generation for browsable decision history |
| [arc42/arc42-template](https://github.com/arc42/arc42-template) | 1.2k | 12-section architecture documentation template (intro, constraints, context, building blocks, runtime, deployment, crosscutting, quality, risks, glossary) |
| [bitsmuggler/arc42-c4-example](https://github.com/bitsmuggler/arc42-c4-software-architecture-documentation-example) | 189 | Worked example combining arc42 sections with C4 diagrams, docs-as-code |

**Most relevant for McCall:** `joelparkerhenderson/architecture-decision-record` for ADR templates, `arc42-template` for comprehensive doc structure, and `adr/madr` for lightweight decision capture that fits a CLI-first workflow.

---

## Topic 3: Community Discussions and Key Takeaways

**Thread theme: "What makes a great software architect"**
- The hardest practice is putting business needs before technical considerations. Great architects become as knowledgeable in the domain as they are in technology. ([Cerbos](https://www.cerbos.dev/blog/best-practices-of-software-architecture), [Red Hat](https://www.redhat.com/architect/software-architect-traits))
- Credibility comes from being likable enough that people *want* to listen, and competent enough that they *should*. You rarely have direct authority -- trust is the currency.
- Avoid the ivory-tower anti-pattern: architects who don't code, don't help developers daily, and hand off designs without context.
- Big-picture and detail thinking coexist. Switching between 30,000ft and ground level without losing focus in either is the core skill.

**Thread theme: "Architecture for small teams"**
- For small teams: start with C4 for high-level clarity + ADRs for decision history. Skip heavyweight frameworks. ([DEV Community](https://dev.to/adityasatrio/comparing-software-architecture-documentation-models-and-when-to-use-them-495n))
- Docs-as-code wins over wikis: architecture docs live next to the code, version-controlled, with Mermaid diagrams. ([freeCodeCamp](https://www.freecodecamp.org/news/system-architecture-documentation-best-practices-and-tools/))
- ADRs solve the "why did we do this?" problem -- short, focused, built for speed. Writing them too late loses the context that makes them valuable. ([Martin Fowler](https://martinfowler.com/bliki/ArchitectureDecisionRecord.html), [Microsoft](https://learn.microsoft.com/en-us/azure/well-architected/architect-role/architecture-decision-record))
- C4 + arc42 complement each other: arc42 for the overall structure (including quality requirements and risks), C4 for the diagrams within it. ([LinkedIn - Mosis](https://www.linkedin.com/pulse/effective-architecture-documentation-arc42-c4-torsten-mosis))

**Thread theme: "How senior architects think differently"**
- The shift from feature thinking to systems thinking is the dividing line. Implementation-first optimizes for today; architecture-first optimizes for tomorrow's changes. ([DEV Community](https://dev.to/leena_malhotra/the-architecture-mindset-every-developer-should-learn-3k9e))
- Boundaries are drawn based on what changes together, not what looks similar. Things that change together go together; things that don't get a boundary.
- The best teams have "architects that code and coders that architect" -- no separation between design and implementation.
- Running a mental-model checklist on major decisions takes 30 minutes and saves months of rework.

**Recommended books from community threads:** "Fundamentals of Software Architecture" (Richards/Ford), "Domain-Driven Design" (Evans), "Designing Data-Intensive Applications" (Kleppmann), "Software Architecture: The Hard Parts" (Richards/Ford/Sadalage/Dehghani).
