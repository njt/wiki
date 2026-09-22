---
url: https://gist.github.com/3ef64cb852dadd4e90234c69b82252bb
title: "Dr. Egor Kraev on Structured Objects: \"No Escape, No Leak\""
author: Egor Kraev (conference talk transcribed via ytx)
date_fetched: 2026-09-13
date_published: unknown (talk undated; references 2025–2026 context)
topics:
  - guardrails-and-feedback-loops
  - agent-architecture
---

Egor Kraev — ML practitioner "since the last millennium," ex-investment-bank quant, builder of Wise's 30-person AI team, now CTO of Motley AI — argues that the most impactful production use of GenAI is making LLMs emit structured objects *reliably, every time*. Old-school systems "expect objects of a very defined shape," not essays, and the "agent AI nirvana... I don't think will ever happen." Everything in the talk follows from treating the LLM as one unreliable tool among many reliable ones.

He walks an escalation ladder of output-reliability approaches and finds that every standard one leaks. Prompt + parser + hope "probably will not work"; JSON mode guarantees valid JSON but not the right schema; function calling / `with_structured_output` guarantees the schema but not custom validators or semantics; retries just hope for luck; the validator-as-a-tool agentic loop fails because the LLM gives up after 3–4 tries and returns something anyway; `return_direct` closes one leak but not the give-up leak. The recurring law: "you cannot trust the LLM to reliably relay its input to the tool. You just can't, because it's not what it does."

His fix is structural: make the validator the only exit. In his open-source Motley Crew framework, an agent flag forces the agent to return only through the validation tool — "there is no escape, there is no leak" — and if validation fails after X iterations you get `None`: "at least you get a clear failure. You don't get something that the LLM thinks is valid, but it's not." Supporting patterns: the ship-in-a-bottle pattern (the object lives outside the LLM, which acts on it only through semantically natural action tools — the same principle he prescribes for MCP servers: "you don't have atomic transactions as tools, you have meaningful semantically natural actions"); semantic layers instead of LLM-generated SQL ("I laugh evilly whenever I see anybody declare reliable security SQL generation"); an intermediate stripped-down pydantic object that the validator converts into the real artifact; regex/spaCy verification for regulator-facing suspicious-activity reports at Wise; LLM log-likelihoods as features in old-school classifiers; and headless Claude Code for synthetic-data code generation, itself wrapped in validation.

The philosophy: contrary to agentic hype, don't make the LLM a "genius conductor" — make it "a little box surrounded by other boxes" with as little leeway as possible, doing "the one little thing that nothing but an LLM can do," with everything deterministic kept old school. A second production pattern keeps LLMs off the critical path entirely: generate a validated complex object once, then reuse the object — "squishy inputs, but hard outputs."

The gist's own summary is unusually honest about the talk's gaps: zero numbers anywhere (no iteration counts, failure rates, latency, or cost), a hand-waved termination caveat ("so far, restarting the loop has always worked"), validation guarantees shape rather than quality, Guardrails and DSPy untried, security and prompt injection absent, no human-in-the-loop discussion, and an unexamined conflict of interest — the central solution is his own framework.
