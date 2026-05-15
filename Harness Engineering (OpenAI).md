# Harness Engineering (OpenAI)

OpenAI's original five-month experiment in zero-handwritten-code development, and the article that launched the term. Ryan Lopopolo's team of 3-7 engineers shipped a real product to real users (~1M lines, ~1,500 PRs) by building a *harness* — lint rules, architectural constraints, garbage-collection agents, and an Elixir orchestrator called Symphony — instead of writing code. The article is practice, not theory: concrete techniques from a team that actually did it, not speculation about what might work. #concept #person

---

## Key Quotes

> "If you can articulate what it is about the code you don't like, the next step is to write that down."

The thesis compressed into a sentence. The engineer's job shifts from *producing* code to *encoding judgment* into machine-readable rules. Every time you catch yourself thinking "that's wrong," you've identified a harness gap. This is [[Feedback Loop is All You Need]] rendered as engineering practice rather than rallying cry.

> "From the agent's point of view, anything it can't access in-context while running effectively doesn't exist."

The strongest argument for repo-native documentation ever made. Not "documentation is good practice" — documentation in Slack, Google Docs, or people's heads literally *does not exist* for the entity doing the work. This reframes docs from a nice-to-have to an existential prerequisite. It's the same insight as [[Components of a Coding Agent]]'s finding that context quality beats model quality, stated from the infrastructure side.

> "Each one is able to reduce slop in a unique way. But because everyone is invested in putting that knowledge into the codebase, everyone else's coding agents have the best guts of everyone on the team."

The compounding mechanism explained. One engineer writes a custom lint rule → every other engineer's agent immediately benefits. Expertise becomes infrastructure. This is [[Compound Engineering]]'s thesis — "add a system, not manual review" — demonstrated at team scale rather than individual scale.

> "It's super easy to add leverage to your codebase by vibing up some new lints."

Deliberately casual framing for the most important practice in the piece. Lint rules are cheap to create (agents write them), instantly enforced, and compound across the team. The word "vibing" is strategic — it lowers the perceived bar for entry. You don't need a formal process; just write the damn rule.

> "The worker at the steam engine didn't disappear when the centrifugal governor was invented. He moved from turning the valve to designing better governors."

The control-theory framing in one metaphor. Engineers aren't being automated away — they're moving up a level of abstraction. The question isn't "will AI replace developers" but "can developers adapt to designing control systems instead of producing output." This is the same intellectual tradition Böckeler draws on in [[Harness Engineering]].

## Key Themes

### The Harness Definition

Agent = Model + Harness. The harness is everything except the model: AGENTS.md, lint rules, test infrastructure, build system, observability, orchestration, garbage collection. Lopopolo claims harness design determines ~80% of agent reliability; model improvements account for ~10-15%. If true, the implication is that model competition is overrated and harness competition is underrated. #concept

### Twelve Practices

1. **AGENTS.md as index, not encyclopedia** — ~100 lines pointing to structured docs/. The maximalist approach failed: context dilution, stale rules. This aligns with [[Writing a Good CLAUDE.md]]'s brevity argument but contradicts the elaborate CLAUDE.md culture in tools like [[How Intercom Uses Claude Code]].

2. **Custom lint rules as encoded taste** — "Vibing up some new lints" is the highest-leverage activity. Agents write the rules themselves. [[Pre-Commit Lint Checks]] is the closest existing wiki page.

3. **Repo-native context** — All decisions, standards, and context must live in the repo. [[claude-ctrl]] implements this as enforcement.

4. **≤1 minute builds** — Build speed as harness constraint. Slow builds break agent flow. Most teams ignore this entirely.

5. **Post-merge human review** — Pre-merge review eliminated. Humans sample post-merge, identify patterns, encode fixes. Review agents handle the rest with a merge bias. This is the most radical practice and the least defended.

6. **Agent-first observability** — Vector, VictoriaMetrics, Grafana, distributed tracing, CDP for UI inspection. Agents debug themselves.

7. **Garbage collection agents** — Recurring background tasks that scan for violations and open cleanup PRs. Replaced "Slop Fridays." This is [[Cognitive Debt]] made operational — not just diagnosing the problem, building the solution.

8. **Symphony (Elixir/BEAM)** — Multi-agent orchestrator. GenServers per task, supervision trees for fault tolerance. Binary merge decision. [[Process-Based Concurrency BEAM OTP]] covers why BEAM fits.

9. **Minimal skills (~6)** — Not proliferating skills, maximizing leverage per skill. A `$land` skill handles the full PR lifecycle. Contradicts the plugin/skill explosion seen in [[How Intercom Uses Claude Code]] and [[2389 Plugin Marketplace]].

10. **Architecture as prerequisite** — "10,000-engineer architecture" for 7 people. Rigid patterns reduce context requirements. [[Smart Models Dumb Pipes]] is the theoretical framing.

11. **Ghost libraries** — Spec-driven distribution. Agents inline-rewrite low-complexity dependencies. Brett Taylor: "Software dependencies are going away."

12. **Daily standups** — Higher velocity requires MORE human coordination. [[Zero Alignment]]'s thesis confirmed from the other direction.

### Agent Legibility Score

Charlie Guo's seven-metric scoring system: bootstrap self-sufficiency, task entry points, validation harness, linting/formatting, codebase map, doc structure, decision records. OpenAI's own Symfony repo scored a B. This is the first attempt I've seen to quantify "how agent-friendly is this codebase" — and it's more honest than most vendor frameworks since it scored their own work imperfectly. #tool

### The Control Theory Lineage

Lopopolo explicitly frames harness engineering through cybernetics (Greek *kybernetes* = steersman). The engineer becomes a designer of control systems. This connects directly to Böckeler's feedforward/feedback framework and Ashby's Law of Requisite Variety in [[Harness Engineering]]. Both articles independently arrived at control theory as the intellectual foundation — strong convergence evidence. #concept

## Critical Analysis

This is the most important primary source on agentic development published in 2026. Where Böckeler's [[Harness Engineering]] provides the *theory*, Lopopolo provides the *field report*. The article's authority comes from the fact that they actually shipped — this isn't a thought piece, it's an after-action report.

**The "zero code" framing is a stunt that obscures the real insight.** The harness IS code — lint rules, build configs, test infrastructure, the Symphony orchestrator itself. Saying "no code was written" is like saying "no bricks were laid" while building a brick-laying machine. The important claim isn't zero code — it's that the *nature* of the code changed from product logic to meta-engineering. Every lint rule, every architectural constraint, every garbage-collection agent is software. It's just software at a higher level of abstraction.

**The post-merge review model is radical and under-defended.** "Review agents biased toward merging, nothing above P2 priority surfaced" — this is either evidence that the harness works exceptionally well, or evidence that they're not looking hard enough. The article doesn't tell us which. The product was an internal beta application, not a payments system or healthcare platform. The stakes matter enormously, and Lopopolo doesn't address them.

**The daily standup survival is the most honest detail in the piece.** If extreme AI velocity requires *more* synchronous human coordination, not less, then the "dark factory" vision where humans disappear is wrong. Humans don't vanish — they move up a level. But the coordination overhead doesn't necessarily decrease. [[Zero Alignment]]'s warning ("one dev with 24 agents produces chaos") is confirmed here from the other direction: even a team that's doing everything right still needs daily syncs.

**The AGENTS.md finding contradicts the elaborate CLAUDE.md culture.** OpenAI tried the maximalist approach (one giant file with all rules) and it failed. Their conclusion (~100 lines, progressive disclosure) aligns with [[Writing a Good CLAUDE.md]] and [[CLAUDE.md (Universal)]] but contradicts the plugin/skill explosion in tools like [[How Intercom Uses Claude Code]] (100+ skills). There's a genuine tension here that neither side has resolved: is the right answer six skills or a hundred? The data suggests it depends on team size, but nobody has articulated the scaling law.

**The ghost libraries claim is provocative but probably bounded.** "Software dependencies are going away" is true for left-pad. It's not true for Postgres, Kubernetes, or React. The interesting question is where the boundary lies — which dependencies are simple enough to inline-rewrite and which are too complex? Lopopolo doesn't draw this line, and the omission matters.

**The 1-minute build ceiling is the most underrated practice.** Most teams obsessing over model quality have 5-15 minute CI pipelines. Build speed is a hard multiplier on agent effectiveness — every second of build time is a second the agent can't iterate. This is the kind of boring infrastructure constraint that separates teams that make agents work from teams that don't.

**The convergence with Böckeler is striking.** Both articles independently arrived at control theory (cybernetics) as the intellectual foundation, both emphasize feedforward + feedback, both identify garbage collection as essential, both conclude that architecture constrains the agent's output space. When two independent teams at different organizations reach identical conclusions, it's not fashion — it's a real pattern.

Compared to [[Minions — Stripe's One-Shot Coding Agents]]: Stripe focuses on volume (1,000+ unattended PRs/week), OpenAI focuses on reliability (encoding judgment into the harness). These are complementary strategies at different scales. Stripe's approach is breadth-first (many simple PRs), OpenAI's is depth-first (fewer, more reliable PRs). The synthesis is probably: use Stripe's volume for mechanical changes, OpenAI's harness for architectural decisions.

The biggest gap in the article: nothing about how to teach this. Lopopolo's team learned harness engineering through five months of trial and error. There's no curriculum, no playbook, no training program. Every team adopting these practices is rediscovering them from scratch. [[A Practical Guide to Brownfield AI Development]] is the closest thing we have, but it's for legacy codebases, not greenfield harness construction.

---

*Sources: [[raw/harness-engineering-openai]]*
*Note: openai.com returned 403; page reconstructed from The Neuron's detailed summary, ZenML LLMOps Database, and multiple web search results, all published March-April 2026.*
*Last updated: 2026-05-15*
