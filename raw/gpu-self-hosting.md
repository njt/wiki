---
url: https://aistack.imec-int.com/blog/gpu-self-hosting
title: "How many devs can you fit on a GPU? — self-hosting explained"
subtitle: "A reality check on self-hosting the model behind your coding agent."
author: aistack (imec)
date_fetched: 2026-08-01
date_published: 2026-07-23
site: aistack (imec)
---

# How many devs can you fit on a GPU? — self-hosting explained

*A reality check on self-hosting the model behind your coding agent.*

The authors benchmarked 64 real coding tasks across self-hosted GPUs, rented hardware, and commercial APIs to measure what you actually get when moving away from frontier model providers.

## Cost

A self-owned box, priced at the hours it actually spends working, lands in the same ballpark as renting one. Utilization rates matter greatly and "paint a very different picture for the setups we tested."

## Quality

The smaller model that fits on a single GPU solves roughly a third of the 64 tasks, while the frontier model solved 40. The largest open-weight model matched the frontier result, but only on an 8×B200 node and "a couple parallel sessions at most."

## Bottom line

Buying GPUs is probably not about saving money — better reasons include "data that can't leave the building" and "a stack nobody can rate-limit" (the latter flagged as "a story for another article").
