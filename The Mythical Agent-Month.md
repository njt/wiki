# The Mythical Agent-Month

Wes McKinney's update to Brooks's "The Mythical Man-Month" for the age of AI coding agents. The core argument: agents are so good at attacking accidental complexity that they generate *new* accidental complexity. At ~100 KLOC the agents start chasing their own tails; at 1M+ lines (like Positron, a VSCode fork) they struggle hard. He calls this the "brownfield barrier" -- the point where agent-generated code becomes so thick that further agent work degrades rather than improves the system.

---

## Key Quotes

> "Agents are so good at attacking accidental complexity, that they generate new accidental complexity that can get in the way of the essential structure that you are trying to build."

> "Every new change has to hack through the code jungle created by prior agents. Call it a 'brownfield barrier.'"

## Key Themes

#complexity #scaling #brownfield #brooks-law #agentic-coding

McKinney's insight is that Brooks's complexity scaling argument wasn't about humans being slow -- it was about the inherent communication overhead of large systems. Agents don't eliminate that overhead; they just move it. When agent A generates 50 files of scaffolding and agent B needs to navigate that scaffolding a week later, the complexity tax is the same whether a human or an agent pays it.

The numbers matter: roborev and msgvault hitting problems at ~100 KLOC, Positron struggling at 1M+. This gives a rough calibration for where agent-driven development breaks down. Below 100K, agents are net positive. Between 100K and 1M, you need careful management. Above 1M, you need fundamentally different approaches.

This connects directly to [[Slowing the Fuck Down]] (Zechner's diagnosis of the same problem from a different angle), [[Cognitive Debt]] (the organizational version of the brownfield barrier), and [[Components of a Coding Agent]] (where Raschka's "context quality > model quality" explains *why* agents choke on large codebases -- context degrades as codebase size increases).

## Critical Analysis

The Brooks analogy is apt but incomplete. Brooks argued that adding people to a late project makes it later because of communication overhead. McKinney argues that adding agent output to a large codebase makes it worse because of complexity overhead. But there's a crucial difference: you can refactor a codebase (reduce complexity) in ways you can't reduce human communication overhead. The question isn't whether the brownfield barrier exists -- it clearly does -- but whether tools like [[workgraph]], [[Three Tier Memory]], and careful [[How to Write a Good Spec for Agents]] can push that barrier further out. The answer is probably yes, but nobody's demonstrated it yet at the 1M+ scale.

---
*Sources: [[raw/the-mythical-agent-month]]*
*Last updated: 2026-05-14*
