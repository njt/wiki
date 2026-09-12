---
url: https://yogthos.net/posts/2026-08-17-llm-workflow.html
title: "A practical workflow for LLM-assisted development"
author: Dmitry Sotnikov
date_fetched: 2026-09-04
date_published: 2026-08-17
site: yogthos.net
topics:
  - agent-coding-workflow
---

# A practical workflow for LLM-assisted development

Dmitry Sotnikov (yogthos — Clojure developer and author of *Web Development with Clojure*, currently building the Jolt Clojure compiler) distills months of daily LLM-assisted development into a practical workflow. His central claim: LLMs are an amplifier, not a substitute — they demand domain expertise to wield, and the developer still owns architecture and design.

**The inverse of programming.** Hand-written code is built up function by function; an LLM dumps a large volume of code up front, and the work becomes whittling it down to what you actually need. Sotnikov frames the agentic loop as a genetic algorithm: the model emits a rough solution, tests and feedback act as selection pressure, and the code converges toward whatever the tests reward.

**Delegate the boilerplate, own the architecture.** LLMs excel at well-trodden tasks (endpoints, UIs from API specs), explorative tracing of call graphs, and bridging unfamiliar syntax — he used DeepSeek to write JavaScript as fluently as his native Clojure. But agents trip on context and creativity: they don't know your project's quirks and fall into the "evil genie" naive-implementation trap, so you must spell out constraints and supply scaffolding. Never hand the AI a blank canvas.

**The workflow.** Plan first; have the agent emit a phased Markdown plan and a Mermaid diagram to inspect visually, then break it into independent tasks each shipped as its own branch and PR. Elevate routing logic to first-class state machines. Write tests as the contract — the loop's selection pressure — plus storybooks, Playwright end-to-end tests, and a benchmark suite. Commit every stable state and revert when the agent spirals: a solution that isn't mostly right on the first shot will only accrue kludges.

**The harness matters.** Sotnikov built his own harness (Dirge) with a Janet plugin system and SQLite as project memory and task store, mechanical paren-matching repair, a critic role that reviews diffs, and Behavior Trees so the model is a leaf node inside a deterministic control structure with verifier/critic gates — an approach he credits for letting even a local model solve fairly complex tasks.
