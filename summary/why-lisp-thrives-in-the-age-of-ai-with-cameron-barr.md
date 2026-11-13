---
url: https://www.youtube.com/watch?v=WhXPEmdNRpA
title: "Why Lisp Thrives In The Age Of AI (with Cameron Barré)"
author: Chris Jenkins (host), Cameron Barré (guest) — Developer Voices podcast
date_fetched: 2026-11-13
date_published: 2026 (1h 39m video, transcribed 2026-11-13)
topics:
  - software-engineering-craft
  - agent-coding-workflow
---

A 1h39m Developer Voices conversation between host Chris Jenkins and Cameron Barré about why Lisp — and Clojure in particular — is unusually well suited to the age of AI coding agents. Barré's central argument runs through essential vs. incidental complexity (Out of the Tar Pit, Brooks): Clojure strips accidental complexity, and Lisp's "weird" parentheses are actually a superpower because code-is-data enables DSLs.

Two threads carry the episode. First, the REPL: Barré describes a live, woodworker's-shop feedback loop where you work inside a running program for weeks without restarting it. With AI coding, the REPL becomes the backend half of a full verification cycle — he wires an MCP server onto the REPL plus Playwright for the frontend, so an agent can interrogate, restart, and verify its own work. He cites the Recursive Language Models research as evidence of a coming REPL renaissance: give the agent a symbolic environment to explore data programmatically instead of stuffing it into context.

Second, constraints as the answer to agent drift. Barré is building Grain, a Clojure DSL for event-sourced information systems that maps one-to-one onto event modeling (commands, views/read models, events, to-do processors, screens). The DSL focuses the power of the general-purpose language rather than reducing it: any code outside the Grain primitives is simply wrong, so human and agent alike produce uniform architecture by construction. He explicitly argues that specs alone can't constrain statistically-drifting agents — you need a concrete, code-level floor. A Rails-to-Grain rewrite of an enterprise finance app is the worked example: in Rails, agents produce a different valid approach every day; in Grain there's only one shape. Supporting evidence for Clojure in production: Nubank, the largest digital bank outside China, bought Cognitect and runs its bank entirely in Clojure. Jenkins frames the whole episode with the claim that skills files have flipped language obsolescence: agents are now as good at Haskell as Java, making it cheaper than ever to experiment with old ideas.
