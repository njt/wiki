---
url: https://factory.com/news/frontier-harness
title: "Droid 1.0: Building a Frontier Harness"
author: Factory
date_fetched: 2026-11-06
date_published: 2026-11-06
topics:
  - coding-agents-and-frameworks
  - agent-architecture
---

Factory's Droid 1.0 announcement is really an engineering paper about agent harness design, delivered under a product launch. The thesis: an agent harness — prompts, tools, tool-result handling, and the loop — is the decisive layer, and a harness tuned across every model is better than one tuned for one model. Droid is model-independent by design, with an auto model router (Factory Router), and Factory claims Droid outperforms native harnesses up to 20% more efficiently on the same tasks and models.

The headline change is programmatic tool calling (PTC), implemented as a Script tool: the agent writes JavaScript that runs in a sandboxed interpreter with no file/network/system access of its own, and the program makes hundreds of tool calls in a single model request. Raw file contents and logs stay inside the program as variables — only the program's printed output reaches the model's context. Two worked examples carry the argument: a Linear cleanup (312 tool calls in 2 requests, ~61K tokens vs an estimated 36 requests and ~3.1M tokens with direct calls) and a CI sweep (550 calls, model saw under 0.1% of tool results, a five-number summary). Script mode cuts Droid's cost 15% on Claude Opus 5.5, with savings concentrated in the 12.5% of scripts making six or more calls.

The rest is a survey of harness engineering as systematic practice: an internal Agent Factory where agents propose changes against every model family they touch and engineers review the evidence; prompt-cache hygiene (98% cache reuse, 98.6% append-only requests, tools appended at the end of the request so a mid-session MCP load doesn't invalidate the prefix); a 37% cut to the base prompt by sending only 2 tool definitions up front and loading the rest on demand; bounded autonomy rules living in the loop rather than the model (silence retries, repeat-call counters, 16K-token compaction summaries, background work with wake-ups); and aggressive latency profiling down to 124 ms from Enter to request sent. Head-to-head: on Claude Opus 5.5 Droid takes 26% less time per passed task than Claude Code with 17% fewer tokens; on GPT-6.1 Sol it uses 12% fewer tokens than Codex while taking 10% longer. Methodology is Terminal-Bench 4, three trials per task, only tasks both harnesses passed in all trials.
