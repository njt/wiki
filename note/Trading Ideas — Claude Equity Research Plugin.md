# Trading Ideas — Claude Equity Research Plugin

A Claude Code plugin that turns an LLM into an institutional-grade equity research analyst. One slash command (`/trading-ideas:research AAPL`) generates an 8-section professional report with BUY/SELL/HOLD ratings, price targets, probability-weighted scenarios, options flow analysis, and insider activity monitoring — formatted to Goldman Sachs standards. Built entirely with Claude Code. 511 stars. Zero configuration, no API keys.

---

## Key Quotes

> "You are a professional equity research analyst providing institutional-grade trading analysis."

The system prompt opens by assigning the model a professional identity. This is prompt engineering 101, but the specificity matters: "institutional-grade" sets a quality bar, "professional" invokes a tone register. Compare with vague role prompts like "you are a helpful assistant."

> "Use specific numbers and percentages where available. Include timeframes for all metrics (YoY, QoQ, etc.). Cite price targets with analyst firm names when possible."

This is the real quality control mechanism. The prompt doesn't just ask for analysis — it mandates evidentiary standards. Every claim must be quantified, time-bound, and sourced. The prompt is a spec, not a suggestion.

> "This analysis is for educational and research purposes only. Not financial advice."

The CYA boilerplate is legally necessary but also honest about what an LLM can (and can't) responsibly do. An LLM has no fiduciary duty, no license, no liability. Saying so explicitly is better than pretending otherwise.

---

## Key Themes

#tool #finance #claude-code #plugin #prompt-engineering

### The Prompt as Financial Analyst

The entire plugin is a single markdown file — a carefully structured system prompt. There's no code, no API integrations, no data pipeline. The prompt instructs Claude to run parallel WebSearch calls and synthesize results into a formatted template. This is prompt engineering at its most ambitious: not tweaking tone or adding guardrails, but encoding an entire professional discipline into a document.

### Quality Through Structure, Not Intelligence

The prompt's quality gates aren't about model capability — they're about output format. Every section requires specific numbers, named sources, probability weightings, and conviction levels. The structure forces rigor. A lazy model can't hide behind vagueness when the template demands "Q4 2024: Revenue $94.9B (+6% YoY)."

### The Marketplace as Distribution Channel

The plugin installs via Claude Code's plugin marketplace (`/plugin marketplace add`). This is the same distribution model as VS Code extensions or Chrome extensions — a platform play. The marketplace.json manifest, plugin.json metadata, and slash command registration form a packaging standard that turns prompts into installable products. The [[2389 Plugin Marketplace]] is testing the same thesis at larger scale.

---

## Critical Analysis

**What works:** The prompt is genuinely well-designed. It forces parallel web searches (efficiency), demands specific numbers and sources (accountability), and provides an exact output template (consistency). The 8-section framework mirrors actual sell-side research structure. As prompt engineering, this is a masterclass — it treats the prompt as a specification document, not a casual instruction.

**What's concerning:** The tool presents itself as "institutional-grade" and "Goldman Sachs-style" while being a thin wrapper around web search + LLM synthesis. Institutional equity research involves models, data pipelines, compliance review, and human judgment. None of that exists here. The prompt asks for options flow data and insider activity — data that free web searches rarely surface at institutional quality. The output will *look* professional but may hallucinate specifics.

**The sharper tension:** This plugin is simultaneously impressive (as prompt engineering) and dangerous (as financial tool). A well-formatted report with specific numbers and confident conviction levels will feel authoritative even when hallucinated. The disclaimer says "not financial advice," but the output format — complete with BUY/SELL/HOLD ratings, price targets, and position sizing — is structurally indistinguishable from advice. Form creates expectation regardless of disclaimers.

**The real innovation:** Not the financial analysis, but the packaging. A single markdown file, distributed via plugin marketplace, installable with one command, delivering a structured professional workflow. This packaging model — prompt-as-product, marketplace-as-channel — is more replicable and consequential than any equity research report it generates.

---

*Sources: [[summary/trading-ideas-claude-equity-research]]*
*Last updated: 2026-05-15*
