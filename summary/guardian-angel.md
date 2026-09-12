---
url: https://gwern.net/guardian-angel
title: "Guardian Angels: LLM Personalization for Productivity and Security"
author: Gwern Branwen
date_fetched: 2026-07-18
date_published: 2025-12-01
topics:
  - personal-agents
  - ai-research-and-models
---

Gwern proposes **Guardian Angels (GA)** — personalized "digital twin" LLMs that emulate a single user's personality, values, and preferences, rather than serving as generic assistant chatbots. The goal is to solve the principal-agent problem by unifying principal and agent: the human defines what is worth doing, and the GA handles execution.

Current chatbots are structurally misaligned, argues Gwern. Frontier labs are incentivized to replace humans entirely rather than augment them, since humans are the slow serial bottleneck. Chatbots exhibit mode collapse (RLHF destroys personality diversity), laziness (shallow, conventional outputs), brittleness (context windows cannot encode a lifetime), excessive helpfulness regardless of caller identity, and amnesia (errors cannot be permanently corrected).

The proposed fixes span cooperative inverse reinforcement learning (CIRL), continual learning via dynamic evaluation, active learning with ensembles, and preference learning. Human individual differences appear low-dimensional — perhaps mere kilobits of information — so lifelogging vast datasets matters less than capturing the right high-signal preferences.

A GA would be trained on all available data from a specific principal: emails, chat logs, past sessions, and writings. As it improves at predicting what the principal would say and do, it earns more autonomy. Gwern outlines anti-principles: don't optimize for low latency, low cost, profitability, engagement, demo appeal, benchmarks, or brand safety — a GA must be able to say what its principal would say, however profane or heretical.

The **GBT prototype** (summer 2026) plans to finetune a sub-100B-parameter LLM on Gwern's own corpus: ~1GB of IRC logs, Gwern.net writing, social media exports, and eventually emails, flashcards, and archived web pages. The target is a 100× productivity increase — producing publishable essays from a single-sentence prompt, without heavy revision.

Organizationally, Gwern rejects pure open-source (security risks) and nonprofits (funding). He advocates a startup with expensive subscriptions for power users, a public-benefit-corporation structure, and open-sourcing most research as a commoditize-your-complement strategy. The essay's tone is one of urgent personal necessity — a survival strategy for maintaining meaningful human agency as AI systems grow ever more capable and autonomous.
