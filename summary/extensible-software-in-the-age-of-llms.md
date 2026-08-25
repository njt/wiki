---
url: https://jeremymorrell.dev/blog/extensible-software-in-the-age-of-llms/
title: "Extensible Software in the Age of LLMs"
author: Jeremy Morrell
date_fetched: 2026-08-25
date_published: 2026
---

# Extensible Software in the Age of LLMs

Most web software serves the top of the demand curve; the long tail of unmet needs is different for every user, and shoving more features into one UI makes the product worse for everyone who doesn't need them. LLM-assisted coding has made that long tail reachable — "Software for One," personal apps custom-fit to a single person's workflow.

Morrell's thesis: the web is the most successful software distribution system in the world and shouldn't be left behind, so there is a new opportunity for **extensible web software**. Build a solid, accountable core and let users safely extend it by having LLMs fill in the missing pieces. The pattern already exists locally (Pi, IDEs, game mods, Blender add-ons) and at web scale (Salesforce, a multi-tenant programmable platform since 2007), but the blocker has always been the security of running arbitrary code — and modern sandbox primitives now lower that cost.

The technical requirements: ~$0 when idle, single-digit-millisecond cold starts, hard limits on everything, a solid fault-and-security isolation boundary, and a safe way for code to take actions. The last is the crux — raw `fetch` with API keys leaks and DoSes; proxy filtering is fragile and hard to keep correct; the right shape is to hand untrusted code narrow **capabilities** (IFTTT gives `twitter.post_new_tweet()`, not an API key). He surveys interpreters, V8 isolates, microVMs, and WASM+WASI, then makes the case that Cloudflare Dynamic Workers are the closest production-ready out-of-the-box fit — observability, multi-tenant storage (Durable Objects/R2), durable execution, source control, and hosted LLMs already built in. The closing caveat: platforms are hard, but worth it.
