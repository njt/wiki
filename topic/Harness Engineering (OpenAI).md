# Harness Engineering (OpenAI)

OpenAI's five-month experiment: building and shipping a real software product with zero manually-written code. Ryan Lopopolo's team of 3-7 engineers produced ~1M lines, ~1,500 PRs, at 3.5 PRs/engineer/day — not by writing code, but by building the environment, guardrails, and feedback loops that let Codex agents write it reliably. The article is a field report from a team that actually did it, not speculation about what might work. The title says it: the discipline shifts from producing code to engineering the harness around the agent. #concept #person

---

## Key Quotes

> "Humans steer. Agents execute."

The thesis compressed into four words. The engineer's job shifts from producing output to designing environments, specifying intent, and building feedback loops. This is [[Harness Engineering]] (Böckeler) rendered as practice rather than theory.

> "If you can articulate what it is about the code you don't like, the next step is to write that down."

Every time you catch yourself thinking "that's wrong," you've identified a harness gap. The fix isn't to manually correct the code — it's to encode the judgment into a lint rule, a structural test, or a documentation update. This is [[Feedback Loop is All You Need]] as engineering discipline.

> "From the agent's point of view, anything it can't access in-context while running effectively doesn't exist."

The strongest argument for repo-native documentation ever made. Slack threads, Google Docs, oral tradition — literally invisible to the entity doing the work. This reframes docs from nice-to-have to existential prerequisite. Same insight as [[Components of a Coding Agent]]'s finding that context quality beats model quality, stated from the infrastructure side.

> "Give Codex a map, not a 1,000-page instruction manual."

The AGENTS.md philosophy. The team tried the maximalist approach and it failed in predictable ways: context dilution, stale rules, unverifiability. Their solution: ~100 lines pointing to a structured `docs/` directory with progressive disclosure. This aligns with [[Writing a Good CLAUDE.md]] and [[CLAUDE.md (Universal)]] but contradicts the elaborate CLAUDE.md culture in tools like [[How Intercom Uses Claude Code]].

> "In a human-first workflow, these rules might feel pedantic or constraining. With agents, they become multipliers: once encoded, they apply everywhere at once."

The case for custom lint rules and structural tests. Agents are most effective in environments with strict boundaries and predictable structure. [[Pre-Commit Lint Checks]] is the closest wiki page; [[claude-ctrl]] implements it as enforcement.

> "This is the kind of architecture you usually postpone until you have hundreds of engineers. With coding agents, it's an early prerequisite: the constraints are what allows speed without decay or architectural drift."

The most counterintuitive claim in the piece. Rigid architecture doesn't slow you down when agents are writing the code — it speeds you up by reducing the context they need to reason about.

> "We regularly see single Codex runs work on a single task for upwards of six hours (often while the humans are sleeping)."

The overnight agent fleet, confirmed from inside OpenAI. This is [[Probabilistic Engineering and the 24-7 Employee]] made operational.

> "Technical debt is like a high-interest loan: it's almost always better to pay it down continuously in small increments than to let it compound and tackle it in painful bursts."

The garbage collection philosophy. Recurring background agents scan for deviations and open targeted cleanup PRs. Replaced "Slop Fridays" (20% of the week lost to manual cleanup). [[Cognitive Debt]] made operational — not just diagnosing the problem, building the solution.

## Architecture & Practices

### The Layered Architecture Model

Each business domain is structured as a fixed pipeline with strictly validated dependency directions:

**Types → Config → Repo → Service → Runtime → UI**

Cross-cutting concerns (auth, connectors, telemetry, feature flags) enter through a single explicit interface: **Providers**. Everything else is disallowed and enforced mechanically via custom linters and structural tests — themselves written by Codex.

This is the "10,000-engineer architecture" for a 7-person team. The constraints are what allow agent speed without decay.

### Repository Knowledge as System of Record

The docs/ directory is structured and indexed. AGENTS.md (~100 lines) serves as a map, not an encyclopedia. The directory layout:

```
AGENTS.md → ARCHITECTURE.md → docs/
  ├── design-docs/     (indexed, includes "core beliefs")
  ├── exec-plans/      (active, completed, tech debt tracker)
  ├── generated/       (db-schema.md)
  ├── product-specs/
  ├── references/      (design system, tool references)
  └── (DESIGN.md, FRONTEND.md, QUALITY_SCORE.md, etc.)
```

Plans are first-class versioned artifacts checked into the repo. A doc-gardening agent scans for staleness. CI validates cross-links and structure. [[Scaling LLMs to Larger Codebases]] is the theoretical counterpart.

### Agent Legibility Through Infrastructure

Three concrete investments:

1. **Per-worktree bootable app** — Codex launches and drives one instance per change
2. **Chrome DevTools Protocol access** — DOM snapshots, screenshots, navigation; Codex reproduces bugs and validates fixes
3. **Ephemeral observability per worktree** — LogQL and PromQL access to logs, metrics, traces; torn down when the task completes

This is what makes prompts like "ensure startup in under 800ms" tractable. [[Chrome DevTools MCP — Debug Your Browser Session]] and [[surf-cli]] are the tool-level equivalents.

### The End-to-End Autonomous Feature Loop

Given a single prompt, Codex can now: validate codebase state → reproduce bug → record failure video → implement fix → validate fix → record resolution video → open PR → respond to agent/human feedback → detect and fix build failures → escalate only when judgment required → merge.

Lopopolo caveats this heavily: "depends heavily on the specific structure and tooling of this repository and should not be assumed to generalize without similar investment — at least, not yet."

### Throughput Changes the Merge Philosophy

Minimal blocking merge gates. Short-lived PRs. Test flakes addressed with follow-up runs rather than blocking progress. Corrections are cheap, waiting is expensive. Humans *may* review PRs but aren't required to — review effort pushed almost entirely to agent-to-agent.

### Concrete Example: map-with-concurrency

Rather than pulling in a generic p-limit-style package, the team implemented their own helper — tightly integrated with OpenTelemetry instrumentation, 100% test coverage, behaves exactly as the runtime expects. Technologies described as "boring" tend to be easier for agents to model due to composability, API stability, and representation in the training set.

## Critical Analysis

This is one of the most important primary sources on agentic development published in 2026. Where Böckeler's [[Harness Engineering]] provides the *theory*, Lopopolo provides the *field report*. The article's authority comes from the fact that they actually shipped to real users — this isn't a thought piece, it's an after-action report.

**The "zero code" framing is a stunt that obscures the real insight.** The harness IS code — lint rules, build configs, test infrastructure, architectural constraints, garbage-collection agents. Saying "no code was written" is like saying "no bricks were laid" while building a brick-laying machine. The important claim isn't zero code — it's that the *nature* of the code changed from product logic to meta-engineering. Every lint rule, every architectural constraint is software at a higher level of abstraction.

**The post-merge review model is radical and under-defended.** "Review agents biased toward merging, nothing above P2 priority surfaced" — this is either evidence the harness works exceptionally well, or evidence they're not looking hard enough. The article doesn't tell us which. The product was an internal beta application, not a payments system or healthcare platform. The stakes matter enormously.

**The daily standup survival is the most honest detail.** If extreme AI velocity requires *more* synchronous human coordination, not less, then the "dark factory" vision where humans disappear is wrong. Humans don't vanish — they move up a level. But coordination overhead doesn't necessarily decrease. [[Zero Alignment]]'s warning is confirmed from the other direction: even a team doing everything right still needs daily syncs.

**The AGENTS.md finding contradicts the elaborate CLAUDE.md culture.** OpenAI tried the maximalist approach and it failed. Their conclusion (~100 lines, progressive disclosure) aligns with [[Writing a Good CLAUDE.md]] but contradicts the skill explosion in [[How Intercom Uses Claude Code]] (100+ skills). There's a genuine tension neither side has resolved: is the right answer six skills or a hundred? Probably depends on team size, but nobody has articulated the scaling law.

**The convergence with Böckeler is striking.** Both articles independently arrived at control theory as the intellectual foundation, both emphasize feedforward + feedback, both identify garbage collection as essential, both conclude that architecture constrains the agent's output space. When two independent teams at different organizations reach identical conclusions, it's not fashion — it's a real pattern.

**The dependencies claim deserves skepticism.** The map-with-concurrency example works for left-pad. It doesn't work for Postgres, Kubernetes, or React. The interesting question is where the boundary lies — which dependencies are simple enough to inline-rewrite and which are too complex? Lopopolo doesn't draw this line.

**The ≤1-minute build ceiling is the most underrated practice.** Most teams obsessing over model quality have 5-15 minute CI pipelines. Build speed is a hard multiplier on agent effectiveness. This is the boring infrastructure constraint that separates teams that make agents work from teams that don't.

**The overnight agent claim is both exciting and unverified.** "Six hours while humans sleep" is a compelling vision, but the article provides no data on what percentage of those runs succeed, what the failure modes are, or how often humans need to clean up the aftermath. [[Probabilistic Engineering and the 24-7 Employee]] raises the training-crisis concern: if agents work while you sleep, when do you learn to do the work yourself?

Compared to [[Minions — Stripe's One-Shot Coding Agents]]: Stripe focuses on volume (1,000+ unattended PRs/week), OpenAI focuses on reliability (encoding judgment into the harness). Complementary strategies at different scales. Stripe is breadth-first (many simple PRs), OpenAI is depth-first (fewer, more reliable PRs). The synthesis: use Stripe's volume for mechanical changes, OpenAI's harness for architectural decisions.

The biggest gap in the article: nothing about how to teach this. Lopopolo's team learned harness engineering through five months of trial and error. There's no curriculum, no playbook. Every team adopting these practices is rediscovering them from scratch. [[A Practical Guide to Brownfield AI Development]] is the closest thing, but it's for legacy codebases, not greenfield harness construction.

---

*Sources: [[summary/harness-engineering-openai]]*
*Note: Previously built from secondary sources (The Neuron, ZenML) due to openai.com 403. Updated 2026-05-18 from primary source fetched via surf browser automation.*
*Last updated: 2026-05-18*
