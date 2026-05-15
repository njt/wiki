# Minions — Stripe's One-Shot Coding Agents

Stripe's Minions system produces 1,000+ merged PRs per week — unattended coding agents that write code from scratch without human intervention between initiation and PR creation. Built on a forked version of Block's Goose agent, a central MCP server called Toolshed with 400+ tools, pre-warmed devboxes that spin up in 10 seconds, and a hard limit of two CI rounds per run. The North Star: a pull request with zero human-written code. Humans review; Minions write.

---

## Key Quotes

> "a thousand pull requests merged each week at Stripe are completely minion-produced"

At Stripe's scale — hundreds of millions of lines of Ruby, $1T+ payment volume — this is the most credible production proof point for unattended coding agents. Not a demo, not a benchmark, not a greenfield project. Real code in a real codebase with real consequences.

> "one of our most constrained resources is developer attention"

This is the framing shift that justifies Minions. Not "code is expensive to write" — that's been false since Copilot. But "developer attention is the bottleneck" is the economic insight that makes unattended agents a business decision, not a technology experiment.

> "if it's good for humans, it's good for LLMs, too"

Minions use the same internal dev tooling as human engineers — same linters, same CI, same PR templates, same rule files. No special agent infrastructure. This is the [[Smart Models Dumb Pipes]] principle applied at scale: don't build agent-specific pipes when the human pipes already work.

> "there are diminishing marginal returns for an LLM to run many rounds of a full CI loop"

Two rounds max, then stop. This is a data-driven judgment call that most teams get wrong. The instinct is to let the agent iterate until it passes — but the ROI collapses after round two. Pragmatism over perfection.

> "a minion run that's not entirely correct is often still an excellent starting point"

The framing that makes Minions economically viable. Perfectionism kills agent ROI. A 70% correct PR that a human can finish in 10 minutes beats a 95% correct PR that took 45 minutes of agent thrashing.

> "shift feedback left"

The operational principle: catch issues at the earliest possible stage. Pre-run context hydration, pre-push lints (<5 seconds), selective CI from 3M+ tests, autofixes. Every ounce of feedback moves closer to the moment of code generation.

> "Vibe coding a prototype from scratch is fundamentally different from contributing code to Stripe's codebase"

The brownfield barrier acknowledged. Greenfield agents can YOLO; Stripe's agents must navigate hundreds of millions of lines of existing code, homegrown libraries, and Sorbet type checking. This is the problem [[The Mythical Agent-Month]] diagnosed.

---

## Key Themes

- **#pattern Agent + deterministic hybrid** — The core loop interleaves LLM creativity with deterministic git/lint/test steps. Not "agent does everything" but "agent does the creative part, automation does the reliable part"
- **#concept Developer attention as bottleneck** — The economic justification flips from "write code faster" to "multiply developer attention." Engineers spin up multiple Minions simultaneously, especially during on-call
- **#concept Shift feedback left** — Every check moves earlier in the pipeline: context hydration before the run, lints before push, selective CI, autofixes. Two CI rounds max
- **#tool Devboxes** — Pre-warmed, isolated environments in 10 seconds. The infrastructure prerequisite that makes unattended agents practical
- **#tool Toolshed** — 400+ MCP tools in a central server. Both impressive and terrifying: how do you manage context window pressure with that many tools?
- **#pattern Same tooling, different operator** — Minions read the same rule files as Cursor and Claude Code. Conditional rules per subdirectory. The agent is just another consumer of the team's standards

---

## Critical Analysis

**The real insight isn't the agent — it's the harness.** Stripe didn't build a better LLM. They built a better environment: pre-warmed devboxes, deterministic context hydration, selective CI, autofixes, two-round limit. The Minion itself is a forked Goose. The infrastructure around it is the moat. This is [[Harness Engineering]] in production at the largest scale documented so far.

**The two-round CI limit is the most important operational decision in the piece.** Most teams let agents loop until tests pass, burning tokens and producing diminishing returns. Stripe's data says stop at two. This requires organizational discipline — the willingness to say "good enough, human can finish it" rather than "let it run one more time."

**400 tools is a double-edged sword.** Toolshed's breadth is impressive, but the context window pressure must be real. Stripe handles this through pre-run deterministic context hydration — tools run *before* the agent loop starts, narrowing what the agent needs to call. This is smarter than it looks: it turns tool discovery from an LLM problem into a deterministic one.

**The "if it's good for humans" principle sounds obvious but is rarely followed.** So many agent systems rebuild tooling from scratch — custom linters, custom CI, custom everything. Stripe's approach of making agents consume the same infrastructure as humans is both simpler and more maintainable. The agent graduates from special snowflake to just another team member who reads the rulebook.

**The brownfield honesty matters.** Stripe explicitly contrasts Minions with vibe coding prototypes. Hundreds of millions of lines of Ruby, Sorbet typing, homegrown libraries — this is the hardest possible environment for coding agents. If Minions work here, they work anywhere. But the inverse isn't true, and Stripe knows it.

**What's missing from Part 1:** No data on PR quality (acceptance rate, revert rate, bugs found in review), no cost figures, no discussion of failure modes or prompt injection. Part 2 promises implementation details. The cynic reads this as recruiting content disguised as engineering blog — the hiring link at the bottom supports that read. The optimist notes that even recruiting content from Stripe is more substantive than most companies' technical blogs.

---

## Cross-Links

- [[Agent Coding Workflow]] — Minions is the unattended end of the maturity spectrum
- [[Harness Engineering]] — The theory behind why Minions' infrastructure matters more than its model
- [[Guardrails and Feedback Loops]] — Shift feedback left, lint-first, deterministic enforcement
- [[Feedback Loop is All You Need]] — Same rule files as humans, same linters, same CI
- [[Scaling Long-Running Agents]] — Cursor's planner/worker/judge vs Stripe's one-shot approach
- [[Compound Engineering]] — Adding systems (devboxes, Toolshed, selective CI) rather than manual review
- [[Agent Memory and Context]] — Context hydration as a deterministic pre-run step
- [[How Intercom Uses Claude Code]] — The other comprehensive enterprise deployment documented
- [[The Mythical Agent-Month]] — The brownfield barrier Stripe is navigating at massive scale
- [[Components of a Coding Agent]] — Raschka's "harness matters more than model" validated
- [[Building Agents for Production Systems with MCP]] — MCP as the standard integration layer
- [[Smart Models Dumb Pipes]] — Agent + deterministic hybrid is this principle in practice
- [[CLAUDE.md (Universal)]] — Rule files consumed by both human-operated and unattended agents
- [[RepoMirror]] — Simpler one-shot approach: simple prompts beat complex ones
- [[Building 200+ Integrations with OpenCode]] — One-shot economics at a different scale
- [[Specifications as the Product]] — Code is disposable; the Minion is the factory floor

---
*Sources: [[raw/minions-stripe-one-shot-coding-agents]]*
*Last updated: 2026-05-14*
