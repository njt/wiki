---
url: https://mattwynne.net/dont-fear-the-dark-factory
title: "Don't Fear the Dark Factory"
author: Matt Wynne
date_fetched: 2026-05-15
date_published: 2026-05-12
topics:
  - agent-coding-workflow
---

Matt Wynne recounts being challenged by his boss to adopt the Dark Factory pattern for agentic software development, inspired by Justin McCarthy's work at StrongDM — specifically their commitment to producing software where "humans neither read or wrote the code."

He describes his own journey from skepticism about AI (barely using it as of February 2024 at Mechanical Orchard) to building a daily-use tool in a language he'd never learned. He admits he's "still never read the code" of that tool.

The core thesis addresses concerns from the agile/XP community about whether non-deterministic coding agents can be trusted to produce entire systems nobody has read.

## How a Dark Factory Works

Wynne describes it as "just a really simple loop" — agent sessions connected in a loop with a well-designed validation harness and quality seed input, allowing agents to converge on desired solutions. He emphasizes starting small: automating "mundane, repetitive processes that would normally need a human in the loop" where outcomes can be judged by clear heuristics.

## Practical Example: yaks Project

Wynne's architectural review process: ADRs describe the desired structure, an agent compares code against those ADRs, surfaces recommendations, and another agent implements the first recommendation. Automated tests run, but validation isn't complete until all recommendations are addressed. He writes: "I can leave this thing grinding for an hour or more and when I come back the integrity of the code has been improved."

## Broader Applications

Dark factories aren't just for generating code. They can handle maintenance tasks that "improve the quality and integrity of the code." Wynne suggests using factories for mitigating security vulnerabilities, merging dependency upgrades, or running and triaging mutation tests — while still writing production code by hand if preferred.

## Connection to TDD

He draws a parallel to learning test-driven development: "designing a dark factory is challenging because you have to create this validation harness, and that forces you to think about what you want, before you have it."

## References

- Justin McCarthy / StrongDM: factory.strongdm.ai
- Mary & Tom Poppendieck's Lean Software Development
- Matt Wynne's personal tool (yaks): github.com/mattwynne/yaks
- James Shore: "You Need AI That Reduces Your Maintenance Costs" (jamesshore.com, 2026)
- Navan: "How to Build Your Own Software Factory" (web.navan.dev, May 6, 2026)

Closing note: "No tokens were spilled in the writing of this post. This is entirely hand-crafted, artisanal writing."
