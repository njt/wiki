---
url: https://www.dragos.com/blog/ai-supply-chain-security-canary-tokens
title: "AI Supply Chain Security: Canary Tokens"
author: HD Moore
site: Dragos
date_fetched: 2026-11-20
date_published: 2026-11-20
topics:
  - security-and-sandboxing
  - ai-infrastructure-and-hardware
---

Dragos (HD Moore's research team) seeded ~2 million AI prompts with unique canary tokens — AWS keys, URLs, DNS names — across a five-month window (July–November 2026) and monitored where those secrets surfaced after inference. The technique is elegant: each token exists in exactly one prompt, so any use of it pinpoints a leak back to the specific prompt, model, router, and privacy setting that carried it.

Three major findings emerged. First, evidence of actual provider compromises: at least three providers were breached, with attackers copying requests at the front door — stealing a couple hundred seeded AWS keys and using them for tens of thousands of API calls (listing S3 buckets, creating Bedrock keys, adding IAM users). One attacker even deployed ARTEX, an LLM-driven pentesting framework. None of the three providers knew they'd been breached until Dragos told them.

Second, inference marketplace resellers siphoning prompts without exploiting anything: just list a model at the lowest price, enable logging, and wait. The leaks followed individual sellers, not the marketplaces. Third, widespread disregard for privacy policies: a Chinese lab scraped secret URLs from over half of prompts sent to it, including roughly two-thirds of requests flagged Zero Data Retention — including prompts routed through U.S. resellers of that lab's models. One prompt marked ZDR surfaced as a full link shared in a Feishu chat three-plus weeks later.

The article also describes the tooling: a dedicated canary-seeding service and "traceroute for AI inference" that reveals hidden router hops by sending broken requests and reading the nested error envelopes. It closes with practical advice — go direct to first-tier providers, distrust reseller marketplaces, verify privacy flags change routing, and seed your own canaries.
