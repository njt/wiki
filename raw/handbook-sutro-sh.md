---
url: https://handbook.sutro.sh/
date_fetched: 2026-09-04
---

## What is Analytical AI?

In late 2022, the “ChatGPT moment” happened. The masses began to understand the general power of foundation models, and developers immediately started incorporating them into a slew of new products. At the same time a less discussed usage pattern also emerged: data, research, ops, and product teams began using foundation models to process unstructured data and make scaled operational decisions. Put simply: if the AI’s job is to**decide something, rather than create**something, it’s analytical AI.

## Why does Analytical AI matter?

While the distinction may seem subtle, best-practices for analytical purposes often diverge from other generative use cases. This is for a few primary reasons:- Tasks are typically *measurable*. You can create a ground-truth dataset using expert annotations that can be validated against for correctness. Other generative AI outputs are not directly measurable, which is why you need to build evals (a special case of analytical AI) to measure them.
- Tasks are often specific and discriminative, not general and emergent. You use an LLM’s autoregressive reasoning and instruction-following capabilities to make decisions, but reduce “creativity” in favor of consistency. For this reason, the task can often be run on the smallest possible model that’s been evaluated for task accuracy, rather than reaching for the largest, maximally-intelligent model.
- Because analytical AI typically does not involve a transaction with a user, more latency is tolerated - so batch and other flexible workload processing models are acceptable, often saving tremendously on costs and overall processing time. This is analogous to OLTP vs. OLAP/map-reduce style data processing.

## Who is this guide for?

- Data, ML, and analytics teams using LLMs to transform unstructured datasets to structured ones
- AI engineers and product managers building evals and trying to improve the reliability of their AI products
- Operations teams looking to scale the expertise of their domain experts via reliable AI decision models
- Research teams building judges and other verifiable reward functions

## Why did we write this guide?

Sutro builds products to support analytical AI, which we see as an early but emerging space. Many of our customers are just getting started building many of these systems, especially now that more AI products are coming online and generating unstructured data that need analytical processing. We spend a lot of time in the trenches with customers, helping them architect, design, improve, and reason through how to build these systems. It can be thought of as an evolving FAQ as we learn alongside our customers. The goal of this guide is to serve as living reference material for developers who are building analytical AI products, regardless of their choice of tooling (although we hope you’ll come talk to us).## How to Use This Handbook

**Primitives**covers the core analytical AI workload types.

**Patterns**discusses best practices for implementation of the primitives.

**Architectures**are higher-level guides to build end-to-end systems.

**Deployment**covers operational considerations for production use. Each page should be useful on its own, and we recommending starting in the pages most applicable to your current needs. If you are just reading primarily out of curiosity, we recommend starting in the Primitives section.
