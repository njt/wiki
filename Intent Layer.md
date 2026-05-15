# Intent Layer

Railly Hugo's framework for giving AI coding agents the tacit knowledge senior engineers carry — by placing `AGENTS.md` files at folder boundaries that document purpose, contracts, and pitfalls the code alone can't convey.

---

## Key Quotes

> "Your best engineers don't grep randomly. They have a mental map."

Hugo opens with the core observation: a senior engineer navigating a codebase isn't searching blindly. They know which config file matters, which folder actually owns the logic, which legacy directory is a trap. The agent has none of this — and burns tokens wandering.

> "Context engineering is designing the full information an agent needs to perform reliably."

This is the disciplinary framing. Context engineering spans system prompts, structured I/O, tool definitions, and RAG/memory. The Intent Layer skill tackles the first piece: system prompt infrastructure.

> "Same bug, with AGENTS.md in place: 16k tokens loaded (not 40k)."

The empirical claim. Hugo's example goes from 40k tokens of dead-end exploration to 16k tokens of direct action — a 60% reduction. The agent "went straight to the config file, found it first try." This isn't a marginal improvement; it's the difference between a tool that's useful and one that's wasteful.

---

## Key Themes

**#concept Context Engineering** — Hugo's term for the discipline of curating the information environment an agent operates within. Distinguished from prompt engineering by its scope: it's about the *system*, not the message.

**#tool Intent Layer** — A skill (`npx skills add crafter-station/skills --skill intent-layer -g`) that scans a codebase, detects existing CLAUDE.md/AGENTS.md files, suggests folder boundaries for new context nodes, and interviews the developer about patterns and pitfalls. Re-runnable as the project evolves.

**#pattern Hierarchical Context Nodes** — `AGENTS.md` files placed at folder boundaries, each documenting what that folder owns, what it doesn't, its contracts, and its footguns. These cascade: an agent entering `services/payments/` gets both the root-level CLAUDE.md and the local `AGENTS.md`.

**#person Railly Hugo** — Developer behind the Intent Layer skill and the broader context engineering framework. Builds on Tyler Brandt's "The Intent Layer" concept and earlier work on "LLMS.md" from an "AI-First Manifesto." Active at [@RaillyHugo](https://x.com/RaillyHugo).

---

## Critical Analysis

**The insight is correct, but the mechanism is fragile.** Hugo's diagnosis — agents lack tacit codebase knowledge — is dead-on. The fix (hierarchical markdown files at folder boundaries) is a pragmatic first step, but it's a documentation solution to a knowledge problem, and documentation rots. The skill's audit/re-run feature acknowledges this, but there's no enforcement mechanism. Nothing stops an engineer from adding a folder without an `AGENTS.md`, or letting an existing one go stale. Compare with [[Feedback Loop is All You Need]] and [[Harness Engineering]] — context files are feedforward (instructions), not feedback (enforcement). They work when maintained; they mislead when neglected.

**The 40k→16k token claim needs scrutiny.** Hugo provides a single anecdote, not a systematic evaluation. Was the bug representative? Was the `AGENTS.md` written *after* diagnosing the bug (contamination)? The claim is directionally credible — better context should reduce wasted exploration — but the magnitude is unverified. Contrast with [[Demystifying Evals for AI Agents]], which argues for rigorous, repeatable measurement.

**This is essentially a structured CLAUDE.md strategy.** The wiki already covers several approaches to agent instruction files: [[CLAUDE.md (Universal)]] for global behavior, [[Writing a Good CLAUDE.md]] for brevity and hand-crafted quality, [[Scaling LLMs to Larger Codebases]] for prompt libraries as feedforward investment. Hugo's contribution is the *hierarchy* — distributing context across folder-level files rather than one monolithic doc. This is genuinely useful for large repos, but it's an organizational insight, not a conceptual breakthrough.

**The multi-agent compatibility is underrated.** Hugo ships this as a skill that works across Claude Code, Codex, Cursor, Copilot, and 10+ other agents. Most context-engineering advice is agent-specific. A cross-platform `AGENTS.md` convention has network effects: the more agents respect it, the more valuable writing one becomes. This is the same bet [[2389 Plugin Marketplace]] makes — that shared conventions beat walled gardens.

**The missing piece is verification.** Hugo's framework tells agents where to look. It doesn't tell them how to know when they've found the right thing, or when the `AGENTS.md` is wrong. The natural complement is something like [[Specsmaxxing]] (acceptance criteria with stable IDs) or [[Pre-Commit Lint Checks]] (deterministic enforcement). Context tells you where; tests tell you whether.

---

## Cross-References

- [[Agent Memory and Context]] — Synthesis page on context management as the real engineering challenge
- [[CLAUDE.md (Universal)]] — The parallel concept: minimal global agent instructions
- [[Writing a Good CLAUDE.md]] — How to write effective agent-facing documentation
- [[Scaling LLMs to Larger Codebases]] — Gill's framework for where to invest: prompt libraries as feedforward
- [[Harness Engineering]] — Feedforward vs. feedback: why context files need enforcement to stay trustworthy
- [[A Practical Guide to Brownfield AI Development]] — Pupius on making agents productive in legacy codebases; tests as system boundaries
- [[How Hightouch Built Their Long-Running Agent Harness]] — Context management, not model ability, is the bottleneck
- [[Designing Agentic Loops]] — Simon Willison on the meta-skill of choosing tools and guardrails
- [[Feedback Loop is All You Need]] — The counterpoint: linters beat prompts; context files are prompts
- [[Coding Agents and Complexity Budgets]] — Token efficiency as a practical constraint
- [[2389 Plugin Marketplace]] — Skills as the distribution model for agent capabilities

---

*Sources: [[raw/intent-layer]]*
*Last updated: 2026-05-15*
