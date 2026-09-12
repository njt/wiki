# Orchestrating AI Code Review at Scale

Cloudflare's production AI code review system: a CI-native OpenCode-powered orchestrator that launches up to seven specialized review agents with a coordinator judge — the most detailed public production report of AI code review at scale, with real cost data ($1.19 avg, $0.98 median), real metrics (131K reviews, 3m39s median), and honest limitations.

---

## Key Quotes

> "Telling an LLM what NOT to do is where the actual prompt engineering value resides."

Ryan Skidmore drops this as an aside in the security reviewer section, but it's the article's sharpest insight. Every reviewer prompt specifies not just what to flag but what to *ignore* — theoretical risks requiring unlikely preconditions, defense-in-depth when primary defenses are adequate, issues in unchanged code, "consider using library X" suggestions. The negation is where the signal-to-noise ratio comes from. This is the practical vindication of the principle articulated in [[Guardrails and Feedback Loops]]: deterministic constraints and clear scoping beat more instructions every time.

> "The median wait time for a first review was often measured in hours."

The problem statement that justifies the whole system. Not "AI review is better than human review" — it's "human review doesn't scale, and rubber-stamp review is worse than no review." This is the same bottleneck math from [[The End of Code Review]] made concrete at Cloudflare scale: 5,169 repos, 48K MRs in a month, and humans who can't possibly keep up.

> "A single structured review comment."

The coordinator's output format is a design decision hiding in plain sight. Instead of seven agents each posting a comment (noise), or seven agents feeding a chat thread (chaos), the coordinator synthesizes everything into one structured post. This is the same principle as the judge pattern in [[Agent Orchestration]] but applied to the *output interface* — the synthesis IS the product, not the raw review data.

> "Break glass was used only 288 times — 0.6% of MRs."

The escape hatch metric is the most honest number in the article. Of 48,095 MRs, humans overrode the AI reviewer only 288 times. Either the AI is remarkably well-calibrated, or developers have learned to trust it, or (most likely) both. The system's bias toward approval is explicit and the data suggests it's the right bias.

> "Not a replacement for human code review, at least not yet with today's models."

The article's closing honesty. Cloudflare isn't claiming to have solved code review — they've built a scaling strategy for when human review capacity is the binding constraint. The system handles the volume; humans handle the judgment calls the models can't make. What's unstated but visible in the metrics: the system already handles 99.4% of decisions autonomously.

---

## Key Themes

- #pattern **Coordinator + specialized reviewers** — The architecturally clean answer to multi-agent review. Seven domain-specific agents each get a tightly scoped prompt. A single coordinator deduplicates, re-categorizes, applies a reasonableness filter, and posts one structured output. This is the planner/worker/judge pattern from [[Agent Orchestration]] applied to code review at production scale.

- #pattern **Negation-first prompt engineering** — The finding that "what NOT to flag" is where prompt engineering value lives. Each reviewer's prompt specifies what to ignore — theoretical risks, defense-in-depth when primaries are adequate, "consider using X" suggestions. Negation is the signal-to-noise lever.

- #tool **Risk-tiered review** — Not all MRs need the same treatment. Trivial changes (≤10 lines) get 2 agents and cost $0.20. Full reviews (>100 lines or >50 files) get 7 agents and cost $1.68. Security-sensitive paths always trigger full review. This is cost-aware architecture, not "run everything at max."

- #concept **Tiered model assignment** — Different tasks get different models: Opus for the coordinator (the hardest job), Sonnet for code quality/security/performance, Kimi K2.5 for documentation and AGENTS.md. With dynamic override via Cloudflare Worker. The same principle as [[The Advisor Strategy]] and [[Thrifty (Tiered Delegation for Claude Code)]] but applied to a review pipeline rather than a coding loop.

- #pattern **Shared context optimization** — Extracting shared MR context to a file avoids 7× duplication across concurrent reviewers. Combined with Anthropic's prompt caching, achieved 85.7% cache hit rate and saved "an estimated five figures." This is [[Agent Memory and Context|context engineering]] as cost engineering.

- #concept **AGENTS.md as infrastructure** — A dedicated reviewer that catches when teams change tooling without updating their AI instructions. If migrating from Jest to Vitest, the AGENTS.md reviewer flags it. This turns `AGENTS.md` from a nice-to-have into CI-enforced infrastructure. Direct application of [[Steering Claude Code]]'s principle that instructions need deterministic enforcement.

- #pattern **Plugin architecture with lifecycle hooks** — Bootstrap (concurrent, non-fatal), Configure (sequential, fatal), postConfigure (async). VCS coupling isolated in a single `ci-config.ts` file. Configuration via Workers KV with five-second propagation. Production-grade plugin design that most agent frameworks haven't reached.

- #concept **The break-glass metric** — 0.6% override rate is a real trust signal. When humans override AI review only 288 times out of 48K MRs, something is working. The escape hatch exists, but it's rarely needed — which is exactly what you want from an automated system.

---

## Critical Analysis

**This is the article the AI code review discourse needed.** The field has been trapped between "AI will replace code review" manifestos ([[The End of Code Review]]) and "AI can't possibly review code" skepticism, with almost no production data in the middle. Cloudflare published real numbers: cost, latency, token usage, cache hit rates, finding counts by reviewer, risk tier distributions, human override rates. It's the most useful public artifact on the topic by a wide margin.

**The seven-ingredient recipe is more important than any single ingredient.** Any one reviewer (security, performance, code quality) is unremarkable. The coordination layer — deduplication, re-categorization, reasonableness filtering, structured output — is where the system earns its keep. Without it, you get seven comment threads and a wall of noise. With it, you get one clear signal. This is the same lesson from [[Agent Orchestration]]: the harness matters more than the agents.

**The honest limitations section is a flex disguised as humility.** "No architectural awareness," "can't verify downstream consumers," "can't catch subtle concurrency bugs" — these are the same limitations that plague human reviewers. Naming them doesn't weaken the case; it frames what the system is genuinely bad at vs. what humans are also bad at. The difference is that Cloudflare's system reviews 131K MRs in a month at $1.19 each, while humans at the same scale would cost millions and take weeks.

**The AGENTS.md reviewer is the sleeper hit.** Most coverage will focus on the code reviewers and the coordinator architecture. But the AGENTS.md reviewer — an agent whose job is preventing instruction rot in other agents' instructions — is the most forward-looking piece. It treats agent configuration as CI-enforced infrastructure, not documentation. When every team has a CLAUDE.md or AGENTS.md, the reviewer that keeps them honest becomes as essential as the linter. This is a direct implementation of the principle from [[Specifications as the Product]]: the durable artifact is the spec, not the code.

**The "break glass" escape hatch deserves more attention than it'll get.** 288 overrides in 48K MRs. That's a 0.6% override rate. The system is effectively autonomous for 99.4% of decisions. Cloudflare is careful to say this isn't a replacement for human review, and that's probably the right PR positioning, but the numbers tell a different story: they've already replaced human review for the vast majority of changes, with humans as the exception path. The architectural challenge shifts from "design a review system" to "design an escalation system" — and escalation is a harder problem than it looks.

**The model routing control plane is infrastructure that most teams don't know they need.** Being able to disable a model provider across 5,169 repos within five seconds via a KV update — and having failback chains that gracefully degrade — is the kind of operational maturity that separates "we built a demo" from "we built infrastructure." The circuit breakers, timeout budgets, inactivity detection, and error classification (retryable vs. not) are production engineering details that most agent systems skip. The article is worth reading for the infrastructure section alone.

**The Codex is the standards layer this system was always going to need.** Cloudflare's August 2026 follow-up post [[Engineering Standards Enforcement at Cloudflare]] reveals that the seven reviewers don't carry their own rulebooks — they draw from the Cloudflare Codex, a governed corpus of 60+ engineering RFCs with lifecycle states (approved → enforced) and compact JSON extraction for agent consumption. The 230,000 violations and 16,000 blocked merges are the output of a system where the *standards* are as engineered as the *reviewers*. This closes the question left hanging in the April post: where do the rules come from, and who decides when they become blocking?

**What's missing: the false negative rate.** We know what the system found (159,103 findings). We don't know what it missed. Cloudflare acknowledges this implicitly in the limitations section but doesn't provide data. This isn't a criticism — measuring false negatives at 131K-review scale is genuinely hard — but it's the number that would tell us whether automated review is safe or just fast. [[FrontierCode]]'s mergeability benchmark starts to address this from the model side, but we need production studies.

**The comparison with [[OpenCodeReview]] is instructive.** Alibaba's OCR uses a single Go binary with per-file concurrent subagents and a hybrid deterministic+agent architecture. Cloudflare uses OpenCode orchestration with domain-specialized reviewers and a coordinator judge. Both arrived at concurrent subagent review independently, both use structured output, both have a filtering/verification pass. But Cloudflare's system is more ambitious — seven specialized domains vs. a single general reviewer, dynamic model routing vs. static configuration, risk-tiered cost optimization vs. one-size-fits-all. The convergence on "subagents + coordinator" as the right architecture is independently validated.

**A third arrival, this time derived rather than discovered.** [[Building a Production AI PR Review Agent]] reaches four specialists (security, quality, testing, docs) plus an aggregator with a confidence gate — from first principles, with no production traffic, by asking what a senior reviewer actually does and assigning each activity a component. That an unfunded course derivation and a system running 131K reviews a month land on the same topology is the strongest available evidence that the shape is correct rather than incidental. The instructive part is what derivation could *not* produce: risk tiering, the 85.7% cache hit rate from shared-context extraction, and the 0.6% break-glass rate are all thresholds that only traffic can set. Reasoning gets you the architecture; operating it gets you the numbers.

---

## See Also

- [[The End of Code Review]] — Monperrus's thesis that this system implements at production scale: agent review with human escalation
- [[OpenCodeReview]] — Alibaba's open-source AI code review CLI: hybrid deterministic+agent, same subagent pattern, different architecture
- [[Agent Orchestration]] — The planner/worker/judge pattern that this system is a production instance of
- [[Guardrails and Feedback Loops]] — "Deterministic enforcement, not instructions" — the principle behind the negation-first prompt strategy
- [[brooks-lint]] — Pure prompt-engineering code review tool: 12 decay risks, Iron Law diagnosis chain
- [[FrontierCode]] — Cognition's mergeability benchmark: what would a human tech lead actually accept?
- [[Agentic Testing]] — Slack's empirical study of agentic testing: the complementary quality layer
- [[Automating Myself Out of Development]] — The bottleneck shift from "no time to code" to "no time to review" that this system addresses
- [[Running an AI-Native Engineering Org]] — Fiona Fung on bottleneck migration: the organizational context this system operates in
- [[Steering Claude Code]] — The instruction-delivery taxonomy: AGENTS.md reviewer as deterministic enforcement of spec quality
- [[Specifications as the Product]] — Why AGENTS.md rot matters: the spec is the durable artifact
- [[The Advisor Strategy]] — Tiered model assignment: Opus for hard tasks, cheaper models for everything else
- [[Thrifty (Tiered Delegation for Claude Code)]] — The same tiered delegation pattern applied to coding rather than reviewing
- [[Security and Sandboxing]] — Prompt injection defense: the XML structure breakout pattern this system guards against
- [[Agent Memory and Context]] — Context engineering: shared context extraction and caching as cost optimization
- [[Loop Engineering]] — Addy Osmani's meta-skill: designing systems that prompt agents, not prompting them yourself
- [[Vibe Coding as a Team Sport]] — Two-gate approval workflow as constructive answer to review collapse
- [[Building 200+ Integrations with OpenCode]] — OpenCode in production: the ecosystem this system is part of
- [[Cloudflare Temporary Accounts for Agents]] — Another Cloudflare agent infrastructure tool: treating agents as first-class platform users
- [[Cloudflare Security Audit Skill]] — The same Cloudflare DNA applied to vulnerability discovery: six-phase parallel-agent pipeline with adversarial validation, the open-source skill that seeded their internal Glasswing harness
- [[StrongDM Factory Techniques]] — The dark factory version of the same thesis: code validated by harness, not review
- [[Metis — ARM AI Security Code Review]] — ARM's security-focused counterpart: tree-sitter call-graph reachability replaces generic agent retrieval, deterministic adjudication gates model decisions, and SARIF-native output targets the SAST ecosystem rather than the code-review workflow

---

*Source: [[summary/orchestrating-ai-code-review-at-scale]] — Ryan Skidmore, Cloudflare Blog, 2026-04-20*
*Last updated: 2026-07-05*
