# I Stopped Coding and Started Architecting Agents (And Why You Should Too)

Emily Bache — technical coach and TDD practitioner — argues that the skills developers need in the agentic era are the same ones they always needed (small steps, good tests, clean design), just applied one level up: from writing code to engineering the harness around the agent. Her framework is practical and incremental: start with unit tests as Guides, add Sensors as lint scripts, and let each success tighten the flywheel. The post is compact but dense with transferable advice for teams working with legacy codebases. #concept #pattern

---

## Key Quotes

> "I have turned off AI-generated line completions. I find them a huge distraction."

This is the strongest opinion in the post and it's delivered without hedging. Bache isn't anti-AI — she's pro-agent — but she sees line completion as the wrong granularity. It interrupts flow without providing the steering control that agentic loops offer. This aligns with the growing consensus that inline completions produce plausible-looking code that passes neither the correctness nor the design bar. Compare [[Human-in-the-Loop is Tired]] on the psychological cost of constant AI supervision.

> "A better harness leads to better code leads to better harness and code that improves over time."

The harness flywheel. It's the most important idea in the post and the one that distinguishes Bache's framing from simpler "write good prompts" advice. The flywheel means harness engineering compounds: each good design decision gets encoded, each encoded rule raises the floor on future output, and the codebase itself becomes a set of positive examples the agent draws on. This is the same dynamic [[Claude Code Mastery]] describes as CLAUDE.md as "compounding infrastructure," but Bache extends it to include automated verification, not just instruction files.

> "A good AGENTS.md is a model upgrade. A bad one is worse than no docs at all."

Quoting Anthropic. The implication is uncomfortable: most teams' AGENTS.md files are probably in the "worse than nothing" category. A file that's stale, contradictory, or overlong doesn't just fail to help — it actively degrades output by consuming context budget and providing misleading signals. This is the same argument made in [[Steering Claude Code]]'s "every line degrades every other line" and the reason that file length is a correctness concern, not an aesthetic one.

> "Don't download a harness. You won't know what's in it and will be afraid to change it."

The ownership argument. Bache is warning against the same pattern that killed a generation of "best practice" config files: teams adopt something they don't understand, it breaks in ways they can't diagnose, and they're worse off than if they'd started from scratch. The right approach is to grow a harness organically, starting small, so the team understands every rule and feels empowered to change or remove any of them. This is the opposite of the "download this CLAUDE.md" culture and a useful corrective.

> "Harness engineering also involves removing items."

The underappreciated half of maintenance. Every harness accumulates cruft: rules for problems that no longer exist, guidance for model weaknesses that have been patched, sensors checking for patterns the codebase outgrew. Bache frames removal as a first-class harness engineering activity, not an afterthought. This connects to [[Loop Engineering]]'s warning about comprehension debt — a harness you don't understand is a harness you can't trust.

## Key Themes

- **Guides vs. Sensors**: Feed-forward advice (Guides) and post-hoc verification (Sensors) as the two halves of a harness. Guides are probabilistic (instructions in context); Sensors should be deterministic (lint rules, structural tests). Same taxonomy as [[Steering Claude Code]]'s "prompt instructions < hooks < managed settings" enforcement hierarchy.
- **The harness flywheel**: Each successful task produces better Guides and Sensors, which produce better code, which makes future tasks easier. Compounding quality improvement, not one-shot configuration.
- **Start with unit tests**: Bache's entry point is the same as her pre-AI coaching: write good tests, then encode the pattern. Tests as the first Guide. This is bottom-up harness building — the opposite of [[Harness Engineering (OpenAI)]]'s infrastructure-first approach.
- **Legacy code as the killer app**: The flywheel is most valuable where code quality is currently lowest. Better harness → better incremental changes → design improves without risky rewrites. This is a genuinely hopeful message for teams stuck with difficult codebases.
- **Ownership over adoption**: Build your own harness, understand every rule, remove what stops being useful. The harness is part of the codebase, not a plugin.

## Critical Analysis

Bache's post is a good bridge between old-school software craft and new-school agentic development, but it's worth noting what it doesn't say.

**What's here and solid**: The Guides/Sensors distinction is clean and actionable. The flywheel metaphor captures something real — that harness quality compounds — without overselling it. The emphasis on starting small, owning what you build, and removing cruft is all correct and under-discussed elsewhere. And the specific recommendation to start with unit tests is concrete where most "harness engineering" writing stays abstract.

**What's missing**: Bache doesn't address the bootstrapping problem. If your codebase has no tests and the existing code is poor quality, what does the first Guide look like? She says "start with unit tests" but doesn't say how to get the first good one when the agent is producing bad code and you have no Sensors yet. This is the cold-start problem that [[Harness Engineering (OpenAI)]] solves by throwing infrastructure at it (lint rules, structural tests, CI gates) before writing any product code. Bache's bottom-up approach may be more accessible but less complete on day one.

**The line completion dismissal deserves scrutiny**. Bache calls inline completions "a huge distraction" and moves entirely to agentic workflows. But this is an experience-level-dependent claim. Senior developers who can evaluate code at a glance may find completions useful for boilerplate while reserving agents for design work. The right answer is probably "both, at different times" rather than "turn it off." Bache's absolutism here reads as personal preference elevated to universal advice.

**The "don't download a harness" rule is right but incomplete**. Teams absolutely should understand their harness. But the alternative to downloading isn't always building from zero — it's reading someone else's harness, understanding it, and adapting it. The post's framing risks reinventing well-known Sensors (file length checks, complexity gates, test coverage thresholds) that are identical across codebases. Some harness components are commodity; the team-specific parts are the Guides about architecture and domain conventions.

**The biggest unstated tension**: Bache is a TDD coach who spent years teaching developers to write tests first. Now she's saying agents should write the tests, with humans designing the harness that constrains them. This is a much bigger philosophical shift than the post acknowledges. If the agent writes the tests, what happens to TDD as a design discipline? The answer might be "the harness becomes the design discipline" — but Bache doesn't quite say that, and it's the most interesting question her framework raises.

Bache's post pairs well with [[Lean Software Production]] (Matt Wynne's "the product is still working software, but the work is engineering the system that produces it") and contrasts productively with [[Harness Engineering (OpenAI)]], which takes the opposite approach: infrastructure-first, greenfield, top-down. Bache is the legacy-code, bottom-up, incremental path to the same destination.

---
*Sources: [[raw/i-stopped-coding-and-started-architecting-agents]]*
*Last updated: 2026-07-18*
