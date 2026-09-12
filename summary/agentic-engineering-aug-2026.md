---
url: https://wesmckinney.com/blog/agentic-engineering-aug-2026/
title: How Kenn is doing Agentic Engineering
author: Wes McKinney
date_fetched: 2026-08-14
date_published: 2026-08
topics:
  - agent-coding-workflow
---

# How Kenn is doing Agentic Engineering

Wes McKinney (creator of pandas, now at Kenn Software) documents the agentic engineering process and culture his three-person team has evolved since the start of 2026. Kenn merges hundreds of pull requests per week across millions of lines of production code with an empirically low bug rate — and McKinney frames the whole thing against his recent X post: "I think loops are bullshit."

The clarification matters: fully autonomous, no-human-in-the-loop pipelines are the bullshit — anyone promising step-away-from-the-keyboard quality from agents looping on each other's output is "either a) clueless or b) selling you something." What he still runs is **human-operator loops** — the human stays at the design and taste layer while agents do the typing and checking. His token bill is real: ~$56,836/month at API rates, subsidized by coding-agent subscriptions.

The workflow, in brief: start with the right tools (Superpowers and roborev), design together with the human in every important decision, get a second opinion from a different model family, have Superpowers turn the design into a spec, review that spec adversarially until it converges, implement in small pieces with roborev verifying asynchronously, close all reviews with `roborev-fix`, then convert specs and plans into "living architecture documents" rather than retaining them in repos. He bluntly calls the frontier models (5.6-Sol and Fable) "extremely sloppy and almost never suitable for production without substantial hardening."

On top of the process sits the **Clanker Constitution** — a set of operating principles (maintained on GitHub) launched into every agent session: honor the request, act with judgment, finish the job, protect existing work, verify reality, communicate for humans, learn in the right place. The clankers — McKinney's term for coding agents, "since 'agent' gives them too much credit" — are not in charge.

Finally, the tooling: Kenn found the "legacy stack" (GitHub.com, IDEs, raw terminals) unsuitable for their volume, so they built their own — Kenn Forge (workspace for reviewing and landing changes), Ghosthub (multiplexer-native terminal for remote agent sessions), Kata (the system of record for intent), AgentsView (session/token intelligence) and roborev (continuous local code verification) as "accountability engines."
