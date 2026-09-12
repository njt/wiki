---
url: https://threedots.tech/post/ddd-and-ai-coding/
title: "Domain-Driven Design matters more when AI writes your code"
author: Miłosz Smółka
date_fetched: 2026-09-04
date_published: undated
site: threedots.tech
category: Essay
topics:
  - software-engineering-craft
---

# Domain-Driven Design matters more when AI writes your code

## Core Thesis

Miłosz Smółka argues that the ideas behind Domain-Driven Design (DDD) are *more* relevant, not less, as AI coding agents take over implementation — because DDD was never strictly about code. The hard part of software engineering has always been understanding the problem domain and modeling it well, and AI only automates the part that was always easier: writing the code.

## What Hasn't Changed Since 2003

Eric Evans's *Domain-Driven Design* (2003) made the point that engineers gravitate toward elaborate frameworks and technology to avoid the harder work of modeling the domain. Smółka observes the same pattern today, relocated: instead of obsessing over frameworks, teams "follow the model benchmarks and optimize our agentic setup to generate *better code*." Evans's warning — that we still try to solve domain problems with technology — applies unchanged, only now the CEOs believe it too.

## The Domain Model Isn't an Artifact

The foundation of DDD is *knowledge crunching*: domain experts and engineers working together to understand what the software should do. It's tempting to let an AI agent research your documents and produce the domain model for you, but that "misses the point" — the value is that *you and your team* understand how the domain works, not that an impressive artifact exists. An AI-generated wall of text no one reads is worthless. Real projects fail from engineers working on the wrong thing, no one knowing what's needed, and scope creep — not from a missing document.

## Design Before Generating Code

Writing code was always easier than reading it; an agent single-shotting a big feature makes this worse, producing PRs like a lone-wolf developer dropping a massive change. The fix is to design and discuss the solution as a team *before* coding — first the domain parts, then the technical. You don't need a complete spec; sticky notes and a whiteboard suffice. Code review should be a double-check that the implementation is correct, "not the start of a discussion about whether the approach makes sense at all."

## Ubiquitous Language for Agents

DDD's Ubiquitous Language — speaking the same language across teams and in code — maps directly onto prompting. Smółka's cloud-cost anecdote: he told an agent to cut memory usage and it saved "many megabytes" for $1/month, because he never told it the real goal was cutting *cost*. Precise, shared names also shape agent output: "Add user to CRM and support after it's created" is vague, whereas "Once the user signs up… asynchronously create 1) a customer entry in the CRM, 2) a profile in the support system" makes the goal obvious. Bounded Contexts matter too — agents that see your whole repo will naively unify similar entities, so make clear they are separate for a reason.

## Developers Become Domain Experts

You can't fully trust models, and "working in the domain is how you build trust that you know what you're doing." The interesting AI story isn't cloning known apps; it's that experienced engineers who already know a domain can replace expensive third-party software with an in-house version, faster and cheaper than the license. For engineers, learning to work with an unknown domain pays off more than focusing on technical skills alone.

## Don't Delegate Thinking

The only reason Smółka can judge whether an agent's output makes sense is that he's "spent long hours thinking about such problems in the past." Relying on AI to do the thinking is dangerous because you can't build new mental models without thinking — his analogy is reading a book versus a summary: reading forces you to build a complex model in your mind, while a summary is forgotten "ten seconds later when you close the browser tab." He plans to write a follow-up on DDD's tactical patterns.
