---
url: https://www.youtube.com/watch?v=zNFecTwsUz0
title: "Virtual Office Hours: When Inbound Sells Itself"
author: "Tomas Chungus (host) and Gene Grosser (COO, Vercel), Theory Ventures"
date_fetched: 2026-10-09
date_published: unknown (2025–2026)
topics:
  - ai-product-and-business
  - guardrails-and-feedback-loops
---

Gene Grosser, COO of Vercel (ex-Stripe Chief Business Officer), describes how Vercel automated its entire inbound sales function with an agent built by one engineer at 20% of their time, starting June 2025.

The system began as a 125-line prompt encoding the playbook of the team's #1 SDR, who QA'd every agent action through a Slack channel. After six weeks she so rarely disagreed that she was pulled out. Today over 90% of inbound leads run through the system, taking what was roughly a 10-person function down to about $1,000/year of inference (mostly Sonnet via Vercel's AI gateway) plus infrastructure.

The most instructive evolution: the prompt grew past 1,000 lines as the business got more complex, and the model stopped reliably following it. Vercel's fix was to pull deterministic logic out of the prompt and into code — inbound is now 14 hard rules (e.g. Salesforce lookup for an open opportunity routes automatically to the AE), with a companion "escalation agent" that detects and undoes rule violations. The model is now used only where a human would previously have used judgment: marginal qualification, research, and writing the first response.

On maintenance, Grosser applies a human-onboarding analogy: read 100% of a new hire's first hundred emails, then move to random sampling (~1 in 100). On people, the SDRs were "promoted" into outbound, where the next target is low-intent leads (webinar lists, MQLs) that were never worth human calories — with the goal of returning BDRs to actually talking to humans, since "the value of humans is talking to humans." The objective function of the whole system: maximize person-to-person meetings, measured maniacally through a headless Salesforce implementation.
