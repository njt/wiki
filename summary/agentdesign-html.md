---
url: https://maheshba.bitbucket.io/blog/2026/07/22/agentdesign.html
title: "The Bottlenecks for AI-Driven System Design: why Principal Engineers are not (yet) obsolete"
author: Mahesh Balakrishnan
date_fetched: 2026-11-02
date_published: 2026-07-22
topics:
  - agent-architecture
  - software-engineering-craft
---

Mahesh Balakrishnan (two decades building and teaching distributed systems) asks why designing a system has not become trivially outsourceable to LLMs the way writing code has. His framing test: could an agentic software factory, given the S3 API plus an in-memory single-node implementation and a scheduler, rebuild S3 at full scale in a month?

He works through candidate bottlenecks and dismisses most of them. Correctness is tractable for strongly consistent systems because the in-memory implementation can serve as the executable specification. Simulation and verification are hard but not the blocker — LLMs make formal specs and proofs easier, and storage-system workloads are simulatable. Even performance optimization becomes hill-climbing on the space of implementations, "bitter-lesson-pilled."

The real bottlenecks are evolution and its environment. Migration — safely moving a complex live system from one state to another — is the hardest problem senior engineers actually face, and every change can make future changes harder. Frangibility breaks the RL-style learn-by-trying loop, because an agent cannot roll out a new S3 version and see what happens. Entropy creates an agentic arms race: while agents figure out safe evolution, nodes fail, fixes accumulate complexity, and zombie agents acting on stale directives add their own entropy. Simulation hits a diversity barrier — you would have to simulate every simultaneously running version touching shared state, plus every application using S3, and you cannot know what is important to simulate.

His proposed way out is abstraction: humans hide complexity behind abstractions, but most abstractions are not precisely expressible in language, and LLMs reason in language. He argues the human-to-agent shift in system design will arrive only when AI invents a novel, useful abstraction for its own use — "much like primates inventing tools" — with the express goal of reasoning better about the design space.
