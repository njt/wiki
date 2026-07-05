# 2025 in LLMs

Simon Willison's annual year-end survey of the LLM landscape. The defining developments: reasoning models became the standard, agents finally worked (especially for coding), Chinese open-weight models dominated rankings by year's end, and Claude Code's quiet February launch became "the most impactful event of 2025" -- credited with "$1bn in run-rate revenue" by December 2nd. The competitive landscape fragmented: no single lab maintained clear superiority across all domains.

---

## Key Quotes

> "Reasoning models with access to tools can plan multi-step tasks, execute them, and reason about results to update plans."

> "Terminal commands with obscure syntax like `sed` and `ffmpeg` are no longer barriers when an LLM can spit out the right command."

> Claude Code's February 2025 release was "the most impactful event of 2025."

## Key Themes

#LLMs #year-in-review #survey #simon-willison #agents #reasoning

### Reasoning Models

Every major AI lab released reasoning models in 2025, following OpenAI's o1 from September 2024. The key unlock: reasoning models with tool access can plan multi-step tasks, execute them, and reason about results to update plans. This enabled functioning AI-assisted search and superior code debugging.

### Agents Arrived

Willison revised his own 2025 prediction that agents wouldn't materialize. Defining agents as "LLM systems that can perform useful work via tool calls over multiple steps," he found them extraordinarily useful for coding and research. The Deep Research pattern fell out of fashion as faster reasoning models provided comparable results.

### Coding Agents

Claude Code spawned an ecosystem of CLI tools and asynchronous coding agents (Claude Code for web, Codex Cloud, Google Jules). This is the category that matters most for [[Components of a Coding Agent]], [[Compound Engineering]], and the broader agentic coding movement tracked across this wiki.

### Chinese AI Breakthrough

GLM-4.7, Kimi K2 Thinking, and DeepSeek variants ranked above most non-Chinese models by year's end. DeepSeek's January R1 release triggered a "$593bn NVIDIA market cap loss." Most Chinese labs released under MIT or Apache 2.0 licenses.

### Competitive Landscape

- **OpenAI**: Lost undisputed lead; still winning consumer mindshare but challenged across code, image models, and audio
- **Google Gemini**: Strong year with 2.0/2.5/3.0 releases, competitive pricing, in-house TPU advantages
- **Meta Llama**: Disappointed; Llama 4 deemed too large; community gravitating toward older 3.1 versions

### YOLO Mode and Risk Normalization

Asynchronous coding agents enable unsafe "YOLO mode" (automatic approval) by default. Willison warned this reflects "Normalization of Deviance" -- repeated risk exposure without consequences creates dangerous complacency, citing the Challenger disaster. This connects directly to [[Pre-Commit Lint Checks]] (automated guardrails beat trust) and [[claude-code-config (Trail of Bits)]] (hooks for enforcement).

### Willison's Terminology

Several terms that entered common usage:
- **The Lethal Trifecta**: prompt injection requiring private data access + external communication + untrusted content exposure
- **Context Rot**: quality degradation as context windows grow during sessions (see [[Context Rot]])
- **Context Engineering**: designing input context rather than merely crafting prompts
- **Conformance Suites**: language-agnostic test collections enabling agents to port code across ecosystems

### Other Notable Trends

- $200/month premium plans established at all major providers
- METR data showed capability time-horizons "doubling every 7 months"
- MCP gained rapid adoption but initial spec may have been overcomplicated
- Browser agents raised prompt injection security concerns
- Open-weight models improved but frontier cloud models advanced faster; reliable tool-calling remained cloud-only
- Willison built 110 tools via vibe coding; ported MicroQuickJS to Python using Claude Code on his iPhone

## Critical Analysis

This is Willison at his best: comprehensive, opinionated, and grounded in personal experience. The piece functions as the de facto historical record for what happened in LLMs in 2025, complementing the more personal perspective in [[Zheng Dong Wang's 2025 Letter]] (focused on compute thesis and second-order effects).

The strongest sections are on coding agents and risk normalization. The Challenger disaster analogy for YOLO mode is particularly sharp -- it reframes "nothing bad has happened yet" from reassurance to warning signal. The weakest section is on image generation, which gets less depth despite driving ChatGPT's biggest signup spike.

For a wiki focused on agentic coding, the key takeaways are: (1) reasoning + tools was the unlock that made agents work, (2) Claude Code defined the category, (3) Chinese open-weight models changed the economics, and (4) security remains an unsolved problem that everyone is ignoring because consequences haven't arrived yet. That last point connects to every guardrails-related page in this wiki.

---
*Sources: [[summary/2025-in-llms]]*
*Last updated: 2026-05-14*
