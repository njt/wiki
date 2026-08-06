# Designing Agentic Loops

Simon Willison names a skill that practitioners have been fumbling toward but hadn't conceptualized: **designing agentic loops** is the art of crafting the toolset, guardrails, and success criteria that let a YOLO-mode coding agent brute-force its way through problems. The core insight: when you think "ugh, I'll have to try many variations here," that's the signal to delegate — not to grind. An agent is simply "an LLM wrecking its environment in a loop" (Solomon Hykes), and the designer's job is to make that wrecking productive rather than destructive.

---

## Key Quotes

> "An AI agent is an LLM wrecking its environment in a loop." — Solomon Hykes

Willison doesn't flinch from this framing. The whole article is about how to channel wrecking-ball energy into useful work. The answer isn't "be more careful" — it's design better guardrails.

> "I like to use GitHub Codespaces for this — it provides a full container environment on-demand."

The most pragmatic sandboxing advice I've seen: use someone else's computer. If things go wrong, "it's a Microsoft Azure machine somewhere that's burning CPU." This sidesteps the whole local-sandboxing complexity that [[A Deep Dive on Agent Sandboxes]] and [[yolo-cage]] wrestle with.

> "If a credential can spend money, set a tight budget limit."

Obvious in retrospect, rarely done. His Fly.io example — $5 budget, scoped API key, let the agent iterate on Dockerfiles — is a template for countless optimization tasks. This pairs naturally with [[OneCLI]]'s credential injection model and [[You Dont Want Long-Lived Keys]]'s ephemeral credential philosophy.

> The key signal: thinking "ugh, I'm going to have to try a lot of variations here."

This is the article's most actionable takeaway. It's a heuristic, not a methodology — but heuristics beat methodologies when the field is six months old. Compare with [[Compound Engineering]]'s "Plan, Work, Review, Compound, Repeat" cycle and [[Agent Coding Workflow]]'s maturity spectrum.

> "The value you can get from coding agents... is massively amplified by a good, cleanly passing test suite."

This connects directly to [[Spec-Driven Development]] and [[Feedback Loop is All You Need]] — tests aren't about correctness, they're the feedback mechanism that makes agentic loops converge instead of diverging. Without tests, agentic loops are just expensive guessing.

## Key Themes

#concept #tool #pattern #security #coding-agents

### Shell commands over MCP

Willison's contrarian take: shell commands beat MCP for tool design. Coding agents are "really good at running shell commands," and providing one example in AGENTS.md is enough for them to generalize. Capable LLMs already know Playwright and ffmpeg — they don't need a protocol, they need a template. This aligns with [[What I learned building an opinionated and minimal coding agent]]'s finding that four tools outperform elaborate tool collections, and challenges the MCP-heavy architectures of [[Agency]] and the [[2389 Plugin Marketplace]].

### YOLO mode as design problem, not permission problem

The article reframes YOLO from "dangerous thing you shouldn't do" to "dangerous thing you must design around." Three mitigations: sandbox (Docker, [[Apple container]]), someone else's computer (Codespaces), or just take the risk (what most people do). The Anthropic-recommended approach — `--dangerously-skip-permissions` in an internet-restricted container with a firewall script — is the serious answer. See [[Security and Sandboxing]] for the full landscape.

### AGENTS.md as progressive disclosure

Instead of documenting tools exhaustively, Willison gives agents one example invocation and lets them figure out the rest. This is [[Writing a Good CLAUDE.md]] applied to tool design: brevity isn't laziness, it's instruction budget management. Compare with [[CLAUDE.md (Universal)]]'s six-rule approach.

### Tests as the force multiplier

The common thread across debugging, perf optimization, dependency upgrades, and container optimization: automated tests make agentic loops converge. "LLMs themselves are great for building those tests" — a bootstrapping property that [[Compound Engineering]] also exploits. The loop is: use agent to write tests, use tests to guide agent, repeat.

## Critical Analysis

This is Willison at his best: naming a concept before it has a name, in prose that's clear enough to steal. The article has more practical wisdom per paragraph than most whitepapers have per page.

**What it gets right:** The "ugh, many variations" heuristic is genuinely useful. The emphasis on tests as the amplification mechanism is correct and underappreciated. The "someone else's computer" sandboxing advice is the most honest thing written about YOLO safety — most people aren't going to configure Seatbelt or Landlock, but they might spin up a Codespace. The tool-design philosophy (shell commands, not MCP; examples, not docs) cuts through a lot of ecosystem noise.

**What it skips:** The article focuses on single-agent loops. Multi-agent loops — where one agent's output is another's input — are the next frontier and have different failure modes ([[Agent Orchestration]], [[Zero Alignment]]). It also doesn't address the context-window economics of long loops: each iteration consumes tokens, and the model eventually loses the thread ([[Agent Memory and Context]], [[Slate]]). And "just take the risk" as a mitigation strategy is more honest than it is scalable — [[AI Coding Tools Create More Bugs Than They Fix]] documents the consequences.

The article also doesn't cover loops that optimize for *divergence* rather than convergence. [[Morningprint]]'s daily art pipeline is one: a 14-day rolling history fed back into the prompt forces the model to differ sharply from every recent piece in subject, composition, and technique. The loop's job isn't to converge on a correct answer — it's to keep rotating across the whole creative space, with a narrow, rule-governed "continuity" exception for days that genuinely earn it (a holiday after its eve, a resonant anniversary). Willison's "ugh, many variations" signal is inverted: the variations ARE the product.

**The broader frame:** Willison is describing what [[Agent Coding Workflow]] calls Level 3-4 on the maturity spectrum — the human is designing the loop, not running it. The article pairs naturally with [[How Boris Uses Claude Code]] (the creator's YOLO workflow), [[Coding Agents and Complexity Budgets]] (similar practitioner pragmatism), and [[The Claude Code Playbook]] (beginner-to-intermediate tips that land in the same territory).

**The field is six months old.** Willison's closing note — "there's so much more to figure out" — isn't false modesty. Claude Code launched February 2025. We're still in the "naming things" phase, and naming things well is how a field becomes a discipline.

---

*Sources: [[summary/designing-agentic-loops]]*
*Last updated: 2026-05-14*
