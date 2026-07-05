# Umans Code for Organizations

Umans Code's organization-tier documentation: how teams buy seats, manage capacity pooling, and wire up service-account keys for automated workflows — a product page that doubles as an implicit argument for where the economics of AI coding are heading: humans on flat-rate subscriptions, automations on metered tokens, one invoice.

---

## Key Quotes

> "Generous usage built in, 4 parallel sessions (any 10-minute window)."

The specific number matters less than the framing: Umans doesn't price by token for humans. They price by seat and let individuals go wild within bounds. This is the right call for adoption — nobody wants to think about token budgets while coding — but it's an actuarial bet that will need continuous recalibration as model capabilities (and costs) shift under them.

> "Unused capacity from other seats in the same org offsets overages at 50%."

A genuinely clever pooling mechanic. Two unused session-slots cancel one overage — it's not full fungibility, but it's enough to smooth out the peaks without letting one heavy user drain everyone else's capacity for free. This is the kind of detail that suggests Umans has actually run the numbers on team usage patterns rather than picking round numbers.

> "Token-based billing is available **only** to organizations, and **only** through service-account keys."

The boldface is theirs and it's earned. This is the architectural boundary that makes the whole pricing model coherent: humans get all-you-can-eat (within session limits), automations pay by the token. Without this split, you'd either have metered humans (adoption killer) or flat-rate bots (cost disaster). The service-account key is the enforcement mechanism — no human behind the keyboard, no flat-rate safety net.

> "Cheapest route by far. Use it for the quick, high-volume steps around umans-coder."

The `umans-flash` pitch at $0.15/M input tokens. This is the tiered-model pattern applied to coding agents: a heavy lifter (`umans-coder`/`kimi-k2.7`) for the hard stuff, a cheap model for scaffolding, linting, and rubber-stamp tasks. The same architectural instinct as [[The Advisor Strategy]] and [[Thrifty (Tiered Delegation for Claude Code)]], but built into the pricing page rather than the agent logic.

## Key Themes

- **#tool** Umans Code — AI coding platform with individual and organization tiers, routing to multiple backend models
- **#pattern** Seat-vs-service-account split — flat-rate for humans, metered for automations, enforced by credential type rather than policy
- **#concept** Capacity pooling with partial fungibility — 50% offset rate as a pragmatic middle ground between strict per-seat caps and full org-wide pools
- **#pattern** Tiered model routing — cheap model for volume, expensive model for judgment, presented as a pricing feature not an architectural decision
- **#concept** Manual org provisioning — Umans doesn't have self-serve team creation; every org starts with an email to contact@umans.ai

## Critical Analysis

**The good:** The seat/service-account split is the cleanest pricing architecture I've seen in the AI coding space. It maps directly to how teams actually work — humans code interactively, automations run in CI/cron — and the enforcement mechanism (credential type) is deterministic, not honor-system. The 50% session pooling is clever without being over-engineered. And putting the model-routing advice directly in the pricing table ("use flash for the quick, high-volume steps around coder") is product-thinking disguised as documentation.

**The gaps:** Manual provisioning is a yellow flag at scale — every org starts with "email us." That's fine for early adopters but becomes a sales bottleneck. No SLA or uptime commitment is mentioned anywhere on the page, which matters when you're pitching "production alert triage" as a use case. And the $50/seat pricing is positioned as "generous" without quantifying what generous means — how many tokens? How many hours? The session limit (4 per 10-min window) is the only hard cap, and it's a soft one given the pooling mechanic. This vagueness is strategic (flexibility to adjust) but makes budgeting hard.

**The bet they're making:** Umans is betting that the coding-agent market bifurcates into two tiers: individual developers who want simple subscriptions, and organizations that want a unified platform with both human seats and automation infrastructure. The service-account API (`https://api.code.umans.ai`) is the thin end of a wedge — once teams wire their CI pipelines to Umans service accounts, switching costs go up dramatically. This is platform strategy dressed as pricing documentation.

**What's missing from the page:** No mention of data retention, model training on org data, or compliance certifications (SOC 2, etc.). For a page targeting organizations, these omissions are significant. Also no discussion of what happens when an org outgrows manual provisioning — is there an enterprise tier? SSO? Audit logs beyond the wallet ledger? The page reads like v1 documentation for a product that's still finding its enterprise feet.

**The comparison to self-hosted:** The `umans-flash` pricing at $0.15/M input tokens is competitive with API pricing for the underlying model (Qwen 3.6 35B), suggesting Umans is running thin margins on the cheap tier and making money on the premium tier ($4.00/M output for kimi-k2.7). This is a standard razors-and-blades model inverted — the cheap stuff is genuinely cheap to drive adoption, the heavy lifter carries the margin. Compare to running your own [[Local and Open Source Inference]] setup, where the economics flip once utilization passes a threshold.

## See Also

- [[AI Pricing]] — cross-provider token pricing comparison
- [[The Advisor Strategy]] — Anthropic's tiered-model pattern for coding agents
- [[Thrifty (Tiered Delegation for Claude Code)]] — 2389 Research's cost-optimized model delegation
- [[All Your Agents Are Going Async]] — why durable agent sessions need different infrastructure than interactive ones
- [[Loop Engineering]] — designing systems that prompt agents on schedules (the automation use case Umans is targeting)
- [[Running an AI-Native Engineering Org]] — what happens when an entire team operates this way
- [[Inference Cost Napkin Math]] — the economics underneath those per-token prices
- [[Best Infrastructure Platforms for Coding Agents in 2026]] — where Umans fits in the platform landscape
- [[Security and Sandboxing]] — the security model that service-account keys imply but don't document
- [[Agent Orchestration]] — multi-agent patterns that service accounts enable

---
*Sources: [[summary/umans-code-orgs]]*
*Last updated: 2026-07-05*
