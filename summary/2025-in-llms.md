---
title: "2025 in LLMs"
url: https://simonwillison.net/2025/Dec/31/the-year-in-llms/
date_fetched: 2026-05-14
section: "LLMs"
---

# 2025: The Year in LLMs - Simon Willison

Simon Willison's annual retrospective catalogs major developments in large language models throughout 2025.

## Reasoning Models Became the Standard

The year was defined by "reasoning" or inference-scaling approaches, initiated by OpenAI's o1 in September 2024. Every major AI lab released reasoning models in 2025. The breakthrough insight: "reasoning models with access to tools can plan multi-step tasks, execute them, and reason about results to update plans" effectively. This enabled functioning AI-assisted search and superior code debugging.

## Agents Finally Worked

Willison revised his 2025 prediction that agents wouldn't materialize. Defining agents as "LLM systems that can perform useful work via tool calls over multiple steps," he found them extraordinarily useful -- particularly for coding and research. The Deep Research pattern fell out of fashion as faster reasoning models provided comparable results.

## Coding Agents Dominated

Claude Code's quiet February 2025 release became "the most impactful event of 2025." This spawned an ecosystem of CLI tools and asynchronous coding agents (Claude Code for web, Codex Cloud, Google Jules). Anthropic credited Claude Code with "$1bn in run-rate revenue" by December 2nd.

Key quote on tool-use barriers: "Terminal commands with obscure syntax like `sed` and `ffmpeg` are no longer barriers when an LLM can spit out the right command."

## The Chinese AI Breakthrough

Chinese open-weight models dominated rankings by year's end. GLM-4.7, Kimi K2 Thinking, and DeepSeek variants ranked above most non-Chinese models. DeepSeek's January R1 release triggered a "$593bn NVIDIA market cap loss" before recovery. Most Chinese labs released models under permissive open-source licenses (MIT, Apache 2.0).

## Image Generation Innovations

Prompt-driven image editing generated 100 million ChatGPT signups in one week (March 2025). Google's Nano Banana Pro emerged as superior for text-heavy images and professional use cases, while Qwen-Image models offered open-weight alternatives.

## Competitive Landscape Shifts

- **OpenAI**: Lost undisputed lead; still winning consumer mindshare through ChatGPT but challenged across code, image models, and audio
- **Google Gemini**: Strong year with 2.0/2.5/3.0 releases, competitive pricing, in-house TPU advantages, and successful product launches
- **Meta Llama**: Disappointed; Llama 4 models deemed too large; community gravitating toward older 3.1 versions

## YOLO Mode and Risk Normalization

Asynchronous coding agents enable unsafe "YOLO mode" (automatic approval) by default. Willison warned this reflects "Normalization of Deviance" -- repeated risk exposure without consequences creates dangerous complacency, citing the Space Shuttle Challenger disaster as historical precedent.

## Other Notable Trends

- **$200/Month Premium Plans**: ChatGPT Pro, Claude Pro Max 20x, and Google AI Ultra established high-tier pricing
- **Long-Task Capability Growth**: METR data showed frontier models achieving tasks requiring multiple human hours by year's end, with capability time-horizons "doubling every 7 months"
- **MCP's Mixed Results**: Model Context Protocol gained rapid adoption but faced competition from simpler alternatives; initial spec may have been overcomplicated
- **Browser Agents**: OpenAI's ChatGPT Atlas, Claude in Chrome, and Gemini in Chrome raised security concerns; prompt injection remains "a frontier, unsolved security problem"
- **Local vs. Cloud Models**: Open-weight models improved dramatically but frontier cloud models advanced faster; reliable tool-calling remains unavailable in local models, keeping coding agents cloud-dependent

## Willison's Terminology Contributions

- **The Lethal Trifecta**: prompt injection enabling data theft requiring three elements -- private data access, external communication, untrusted content exposure
- **Context Rot**: quality degradation as context windows grow during sessions
- **Context Engineering**: designing input context rather than merely crafting prompts
- **Conformance Suites**: language-agnostic test collections enabling agents to port code across ecosystems

## Personal Projects

Willison built 110 tools on tools.simonwillison.net in 2025, primarily HTML+JavaScript projects using vibe coding. He ported MicroQuickJS to Python using Claude Code on his iPhone. His "pelican riding a bicycle" SVG benchmark became an informal model quality indicator, referenced by Google I/O and Anthropic research papers.

## Conclusions

2025 demonstrated that reasoning + tool-use created functional agent systems, Chinese AI labs could produce world-class models outside US dominance, and serious work could be accomplished through conversational AI interaction. The competitive landscape fragmented significantly -- no single lab maintained clear superiority across all domains. Security risks (prompt injection, browser agents, YOLO mode normalization) remain largely unaddressed despite increased stakes.
