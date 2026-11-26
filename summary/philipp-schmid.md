---
url: https://ai.engineer/code/2026/talks/philipp-schmid
title: "Give Your Model a Computer, Not a Toolbox"
author: Philipp Schmid
date_fetched: 2026-11-26
date_published: 2026-11-11
topics:
  - agent-architecture
  - agent-coding-workflow
---

Philipp Schmid (Google DeepMind) argues that coding-agent harnesses have been carrying dead weight. His semi-annual ritual of rebuilding a harness from scratch led him to strip it down to a single tool — bash, running in a clean subshell with an async timeout — and measure what actually changes. The answer: scores stay flat, but tokens per task drop dramatically.

The mechanism is post-training on tool use. Models are rewarded by a verifier, not taught which tools to use, and they discover that dedicated read/edit tools force one edit per turn while a single bash (or inline Python) call can chain 2–4 commands and make all changes in one go. On his ~350-task eval set, four harnesses scored roughly identically — but the bash-only harness was nearly 2x cheaper per task, with better cache hit rates. With Gemini 4 Carbon, bash-only reached up to 3x fewer tokens than the default tool suite; the weaker 3.5 Flash didn't benefit, because it used bash as a single-command tool anyway.

His conclusions: models keep "eating the harness" (tools, system prompts, planning tools, even subagents — the agent can invoke itself via bash, and the filesystem serves as memory); 2027 will be about tokens-per-task rather than accuracy alone; and harness differences invisible on accuracy will show up in cost. He also flags a countercurrent: code-mode (writing TypeScript/Python directly) and MCP with structured outputs may return, partly for security — bash is hard to audit, an abstracted TypeScript layer is not. The closing directive: "give your model a computer, not a toolbox."
