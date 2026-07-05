---
url: https://claude.com/blog/the-advisor-strategy
title: "The Advisor Strategy: Give Sonnet an Intelligence Boost with Opus"
author: Anthropic
date_fetched: 2026-06-09
date_published: 2026-04-09
---

# The Advisor Strategy: Give Sonnet an Intelligence Boost with Opus

Published April 9, 2026 on the Claude blog. Category: Product announcements. Product: Claude Platform. Reading time: ~5 min.

## Summary

Anthropic introduces the advisor strategy: pair Opus as an advisor with Sonnet or Haiku as an executor. The smaller model runs the task end-to-end (calling tools, reading results, iterating), and only consults Opus when it hits a decision it can't reasonably solve on its own. Opus accesses the shared context, returns a plan (or correction, or stop signal), and the executor continues. The advisor never calls tools or produces user-facing output directly.

This inverts the common sub-agent pattern where a larger orchestrator model decomposes work and delegates to smaller worker models. Here, a smaller, more cost-effective model drives and escalates without decomposition, a worker pool, or orchestration logic.

The advisor tool (`advisor_20260301`) is available natively on the Claude Platform as a single-line API change — no extra round-trips or separate context management needed. It's in beta as of April 2026.

## Article Body

Developers who want to better balance intelligence and cost have converged on what we call the advisor strategy: pair Opus as an advisor with Sonnet or Haiku as an executor. This brings near Opus-level intelligence to your agents while keeping costs near Sonnet levels.

Today we're introducing the advisor tool on the Claude Platform to make the advisor strategy a one-line change in your API call.

### Build cost-effective agents with the advisor strategy

With the advisor strategy, Sonnet or Haiku runs the task end-to-end as the executor, calling tools, reading results, and iterating toward a solution. When the executor hits a decision it can't reasonably solve, it consults Opus for guidance as the advisor. Opus accesses the shared context and returns a plan (or a correction or stop signal — whatever's appropriate), and the executor continues. The advisor never calls tools or produces user-facing output directly.

This inverts a common sub-agent pattern, where a larger orchestrator model decomposes work and delegates to smaller worker models. In the advisor strategy, a smaller, more cost-effective model drives and escalates without decomposition, a worker pool, or orchestration logic. Frontier-level reasoning becomes a resource the executor can call on demand rather than a fixed cost paid on every turn.

In our evaluations, Sonnet with Opus as an advisor showed a 2.7 percentage point increase on SWE-bench Multilingual over Sonnet alone, while reducing cost per agentic task by 11.9%.

We're bringing the advisor strategy to our API with the advisor tool, a server-side tool which Sonnet and Haiku know to invoke when they need guidance or help with a specific task.

In our evaluations, Sonnet with an Opus advisor improved scores across BrowseComp and Terminal-Bench 2.0 benchmarks while costing less per task than Sonnet alone.

The advisor strategy also works with Haiku as the executor. On BrowseComp, Haiku with an Opus advisor scored 41.2%, more than double its solo score of 19.7%. Haiku with an Opus advisor trails Sonnet solo by 29% in score but costs 85% less per task. The advisor adds cost relative to Haiku alone, but the cost-per-quality ratio is dramatically better than stepping up to a larger executor model.

### API Implementation

Declare `advisor_20260301` in your Messages API request, and the model handoff happens inside a single `/v1/messages` request — no extra round-trips or context management. The executor model decides when to invoke it. When it does, we route the curated context to the advisor model, return the plan, and the executor continues.

```python
response = client.messages.create(
    model="claude-sonnet-4-6",  # executor
    tools=[
        {
            "type": "advisor_20260301",
            "name": "advisor",
            "model": "claude-opus-4-6",
            "max_uses": 3,
        },
        # ... your other tools
    ],
    messages=[...]
)
```

Works alongside your existing tools. The advisor tool is just another entry in your Messages API request. Your agent can search the web, execute code, and consult Opus in the same loop.

### Design Partner Quotes

> "It makes better architectural decisions on complex tasks while adding no overhead on simple ones. The plans and trajectories are night and day different."

> "We saw clear improvements in agent turns, tool calls, and overall score — better than a planning tool we built ourselves."

> "On structured document extraction tasks, the advisor tool enables Haiku 4.5 to dynamically scale intelligence by consulting Opus 4.6 as complexity demands, matching frontier-model quality at 5× lower cost."

### Getting Started

The advisor tool is available now in beta natively on the Claude Platform. To get started:

- Add the beta feature header: `anthropic-beta: advisor-tool-2026-03-01`
- Add the `advisor_20260301` to your Messages API request
- Modify your system prompt based on your use case

Anthropic recommends running your existing eval suite against Sonnet solo, Sonnet executor with Opus advisor, and Opus solo.

### Benchmark Notes

- **SWE-bench Multilingual**: Sonnet 4.6 solo used adaptive thinking. Sonnet 4.6 + Advisor used suggested system prompt for coding with thinking turned off. Both runs used high effort with bash and file editing tools. Scores averaged over five trials of 300 problems across nine languages. Opus 4.6 solo also included.
- **BrowseComp**: All runs used thinking turned off with web search and web fetch tools. Sonnet 4.6 runs used medium effort. Sonnet 4.6 + Advisor used suggested system prompt for coding; Haiku 4.5 + Advisor did not. No programmatic tool calling or context compaction. Scores based on 1,266 problems.
- **Terminal-Bench 2.0**: All runs used thinking turned off with bash and file editing tools. Sonnet 4.6 runs used medium effort. Neither advisor run used suggested system prompt for coding. Each task ran in an isolated pod with 3x resource allocation and a 1x timeout. Scores averaged over five attempts.
