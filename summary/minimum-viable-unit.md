---
url: https://brandur.org/minimum-viable-unit
title: "The Minimum Viable Unit of Saleable Software"
author: Brandur
date_fetched: 2026-06-22
date_published: 2026-05-31
tags: [software-economics, build-vs-buy, llm, saas, pricing, indie-dev]
topics:
  - ai-product-and-business
---

# The Minimum Viable Unit of Saleable Software

Brandur announces he's leaving Stainless to work on his side project [River](https://riverqueue.com) full-time. He addresses the skepticism around starting a software company in the age of AI, recounting a LinkedIn anecdote where a company replaced a $400/mo Jira subscription with an LLM-built internal task tracker.

## "Cheap != zero"

The core argument is that while LLMs have made building software cheaper, "they haven't brought it to zero." Brandur walks through the economics: an engineer earning $200k/year costs ~$96/hour. To replace $400/mo Jira, that engineer can spend "no more than 4 hours a month" on maintenance. After factoring in a two-week initial build, he calculates it would take roughly **37 months** to break even.

## "The build threshold"

Contrasts with Salesforce at ~$500/seat/mo. For 50 seats ($25k/mo), you could fund 1.5 full-time engineers to build a clone — making a "build" decision much more plausible.

## "The zone of viability"

Defines a sweet spot where software is priced reasonably enough that buying beats building, even with LLMs available. Two conditions:
- "Sufficient novelty as to make a rebuild-by-LLM non-trivial"
- Pricing "not so exorbitant as to strongly encourage rebuild-by-LLM"

The **minimum viable unit of saleable software** sits at the low end of this zone — below that point, a rebuild costs less effort or money than purchasing.

## "River as a plausible business"

Applies the framework to River, an open-source job queue for Go and Postgres. The Pro version starts at $125/mo for up to 20 developers. He argues the advanced features (workflows, sequential jobs) have enough design thought behind them that replicating them with LLMs would require substantial effort.

## Key Quotes

- "Anything you ship can be instantly displaced by an internal package built by an LLM"
- "LLMs have made software considerably cheaper to build, but they haven't brought it to zero"
- "Maintenance will be an ongoing cost"
- "The math here doesn't pencil out" (referring to the Jira replacement economics)
- "I hate Jira just as much as anyone who's ever used it"

## Core Themes

- **Buy vs. build, re-evaluated:** LLMs shift the calculus but don't eliminate it — ongoing human oversight remains the expensive input.
- **Hidden costs of "free":** Build-your-own via LLM still carries substantial human labor for refinement loops, bug fixes, and feature work.
- **Threshold economics:** A sliding scale exists — cheap products like Jira remain worth buying; expensive ones like Salesforce edge toward build territory.
- **Pricing strategy as defense:** Keeping prices moderate (sublinear, team-based rather than per-seat) helps keep a product inside the zone of viability.
- **LLM-assisted maintenance is not free:** "The most expensive element being the part-time labor of the human in the equation who oversees and verifies results."
