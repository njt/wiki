# Agency

A self-hosted engine for composing AI agents from reusable natural-language building blocks called primitives — role components, desired outcomes, and trade-off configurations. When an agent underperforms, you change one primitive instead of rewriting the entire prompt. Runs as a background MCP service. Built by Vaughn Tan (@arbois).

---

## Key Quotes

> "A monolithic agent has no user-serviceable component parts to individually fix or replace."

The thesis statement. This is the same argument that drove the Unix philosophy, microservices, and modular programming — applied to the prompt layer. Agency treats prompts as software components: decompose, test, recompose.

> "When something does not work, you change one primitive. Everything else stays the same."

The surgical-improvement guarantee. Every primitive addition potentially improves all *future* agents, not just the current task. This is the compounding loop that makes the overhead of composition worthwhile.

> "Memory *about* agents, not memory *within* them."

The three knowledge layers — primitives, compositions, performance — persist at the system level. No individual agent carries context forward. This is a clean separation of concerns: Agency owns identity and composition; the runtime owns execution and persistence.

## The Case Study That Matters

During v1.2.3 development, an initial PRD review agent scored **26/60** — it read the spec in isolation but couldn't check against the actual codebase. After adding a single one-sentence primitive for codebase-awareness (checking API signatures, SQL validity, testability), a re-run scored **55/60**. The composed agent caught a field name mismatch (`similarity` vs. `score`) that would have silently broken every relevance computation.

Across four review rounds, composed agents caught **14 implementation blockers** that general-purpose Claude agents missed.

This is the strongest evidence in the repo. It's not a benchmark — it's a real development workflow where the modular approach demonstrably outperformed monolithic prompting.

## Key Themes

#agent-composition #mcp #modularity #prompt-engineering #eval-loops

## Critical Analysis

**What's genuinely novel:** The three-type taxonomy (roles, outcomes, trade-offs) is more specific than "break prompts into pieces." Trade-off configurations in particular acknowledge that agent design involves value conflicts — thoroughness vs. speed, exploration vs. focus — and that these should be explicit, not buried in prose. The evaluation loop closes: compose → execute → score → improve primitives → recompose. This is [[Compound Engineering]] applied to prompts themselves.

**The license is a adoption ceiling.** Elastic License 2.0 prohibits commercial hosting, which blocks the most natural deployment pattern (a shared team service). This may be intentional — Vaughn Tan appears to be building a research-informed tool, not a SaaS business — but it caps the potential user base.

**Scope honesty is a strength, not a weakness.** Agency explicitly disclaims long-running autonomous agents. It composes, hands off, and gets out of the way. This makes it complementary to runtimes like [[Slate]] (thread-and-episode architecture), [[Cord]] (dynamic task trees), and [[Serf]] (non-interactive coding agent). Agency handles "what agent should do this?" — the runtime handles "keep doing it."

**The primitive extractor is the unspoken differentiator.** The README mentions a bundled tool that pulls primitives from existing workflows, skills, and sessions. If this works well, it solves the cold-start problem that kills most component libraries. Without good primitives, the composition engine is just overhead.

**The tension with minimalism:** [[What I learned building an opinionated and minimal coding agent]] argues that four tools and no MCP is sufficient because frontier models already understand coding agents through RL training. Agency bets the opposite direction — that explicit, curated primitives outperform implicit model knowledge. The 26→60 case study is a data point for Agency's side, but it's one case study on one codebase.

**The real question is whether primitives are genuinely reusable across domains.** Role components like "identifies logical gaps" might transfer. "Checks SQL schema validity" probably doesn't. The value of the primitive library depends on how much domain-specific vs. domain-agnostic capability exists in agent work. If most useful primitives are domain-specific, the library fragments. If most are domain-agnostic, it compounds.

---

*Sources: [[raw/agency]]*
*Last updated: 2026-05-15*
