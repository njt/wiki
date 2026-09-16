---
url: https://addyo.substack.com/p/brownfield-agentic-engineering
title: "Agentic Engineering in Brownfield Codebases"
author: Addy Osmani
date_fetched: 2026-09-16
topics:
  - agent-coding-workflow
  - software-engineering-craft
---

Addy Osmani's field guide to running coding agents against old codebases, where the repository is no longer a complete description of system behaviour and institutional knowledge lives outside the tree. His central organising device is a zone map — green (well-tested, isolated, agents work in a tight loop), yellow (mixed quality, agents work only after characterization tests exist), red (auth, billing, permissions, payroll — human pairing on every step or no work at all). Three rules make zones operational: a person draws the map, zones move only when earned, and the zone sets the verbs.

Beyond zoning, Osmani prescribes writing down only what the code cannot say, persisting research as durable comprehension memos so the next agent doesn't repeat the archaeology, and treating every repeated correction as a missing piece of the harness — promoted into a lint rule, hook, test, or skill. Entry strategy starts with zero-risk work: characterization tests that pin current behaviour (including ugly behaviour the business depends on), then mechanical transforms, then migrations completed in whole units — a migration isn't done while the old path still exists.

He surveys real migrations (Bun's Zig-to-Rust port, Stripe's 3.7M-line TypeScript move, Asana's Enzyme backlog, Shopify's rewrite) and concludes that what transfers between companies is the structure around the agents, not the agents. Agents changed the price of trying implementations, not the evidence needed to choose one; parallelism should come last, and ambiguity now has a visible, countable price.
