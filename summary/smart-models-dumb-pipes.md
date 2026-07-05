---
title: "Smart Models, Dumb Pipes"
author: "Braydon McCormick"
source: "https://dbmcco.github.io/2026/03/24/smart-models-dumb-pipes/"
blog: "Means of Production"
date_published: 2026-03-24
date_fetched: 2026-05-14
---

# Smart Models, Dumb Pipes

By Braydon McCormick, March 24, 2026, Means of Production blog.

## Core Argument

McCormick challenges the prevailing view of large language models as question-answering machines, instead positioning them as "judgment machines" capable of reasoning through complex decisions. The essay draws parallels to network architecture history — specifically the triumph of "dumb pipes" (the internet) over "intelligent networks" (telephone systems) — to propose a design principle for AI systems.

## Key Concepts

**The Problem with Q&A Framing:** Current AI deployment typically optimizes for better answers through improved prompts. McCormick argues this misses the deeper value: identifying where judgment decisions create workflow friction or risk exposure.

**Model-Mediated Architecture:** The proposed principle separates concerns clearly — smart models handle judgment (deciding what should happen and why), while dumb pipes execute deterministic operations and maintain audit trails. Models should "own the judgment work" while infrastructure "owns the mechanical work."

**Three Implementation Examples:**
1. *Expert Panel Simulator* (public GitHub project) — assembles domain experts sequentially to deliberate on problems
2. *DraftForge* (private) — applies the pattern within production content systems
3. *Lodestar/Meridian* (private) — implements tiered model assignments matched to judgment complexity

## The Reframed Question

Rather than "how do we add AI?", McCormick advocates asking: where does judgment belong in this workflow, and what should deterministic systems simply execute and record?

Human friction points — slow decisions, inconsistent judgment, high-cost errors — reveal the actually valuable intervention sites.
