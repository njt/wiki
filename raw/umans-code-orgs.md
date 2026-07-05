---
url: https://app.umans.ai/offers/code/docs/orgs
title: Umans Code for Organizations
author: Umans
date_fetched: 2026-07-05
date_published: unknown
---

# Umans Code for Organizations

Documentation page covering how Umans Code works for teams and organizations, extending the individual user guide. Focuses on three core concepts: seats for human team members (flat-rate), service-account keys for automations (pay-per-token from a shared org wallet), and unified billing.

## At a Glance: Three Core Concepts

1. **Seats** for human team members — flat-rate per person with generous built-in usage.
2. **Service-account keys** for automations — pay-per-token, drawing from a shared org wallet.
3. **One invoice** — both seat subscriptions and service-account usage settle together on the same organization billing.

## Seats (for Humans)

Admins purchase seats from the **Billing → Members** interface and assign them to teammates. Each seat includes:

- "Generous usage built in, 4 parallel sessions (any 10-minute window)."
- Access to the same four models available to individual plans: `umans-coder`, `umans-kimi-k2.7`, `umans-flash`, and `umans-glm-5.2`.
- Teammates follow the standard Quick Start flow — running `umans claude` is sufficient.

### Pricing Tiers

| Plan | Monthly | Yearly |
|---|---|---|
| Seat | $50/seat/month | $500/seat/year (~17% off) |
| Extra session | $20 each | $20 each |

### Sessions Pool Across the Organization

When a team exceeds a seat's 4-session limit, extra sessions cost $20 each. However, "unused capacity from other seats in the same org offsets overages at 50%": two unused session-slots cancel out one extra session. If spare capacity covers the overage, nothing is charged.

## Service Accounts (for Automations)

Service-account keys operate without any human behind them. They consume from the organization wallet at per-token rates, meaning you pay only for what an automation actually uses.

**Key restriction:** "Token-based billing is available **only** to organizations, and **only** through service-account keys." Personal accounts remain on flat-rate Pro/Max subscriptions.

### Common Use Cases

- **Production alert triage & mitigation** — A bot watches alerts from PagerDuty, Sentry, or Grafana, pulls repository context, and drafts a mitigation plan before waking a human. Cost: dollars of tokens per incident rather than a full seat.
- **Automated code review on every PR** — Runs a review pass using the same models engineers use in their IDE. Typically far cheaper than hosted alternatives.
- **Scheduled maintenance** — Migration passes, dependency bumps, flaky-test investigations — any task that could be handed to an agent running on a cron schedule.

The shared pattern: non-interactive, bounded token budget, measurable value per run.

## Token Pricing (Service-Account Usage)

Per-model rates in USD per 1 million tokens. Each request debits exactly what the model consumed (input, output, cache reads, cache writes) with no per-call minimums.

| Model | Origin Model | Input / 1M | Output / 1M | Cache Read / 1M | Notes |
|---|---|---|---|---|---|
| umans-kimi-k2.7 | kimi-k2.7-code | $0.9500 | $4.0000 | $0.1900 | Routes to what umans-coder uses today as the recommended default; built for deep multi-step coding |
| umans-glm-5.2 | glm-5.2 | $1.4000 | $4.4000 | $0.2600 | "Latest GLM with the largest context window; slightly cheaper cache reads than 5.1" |
| umans-glm-5.1 | glm-5.1 | $1.4000 | $4.4000 | $0.2900 | Previous GLM, superseded by 5.2; still billable for existing automations while being wound down |
| umans-flash | qwen3.6-35b-a3b | $0.1500 | $1.0000 | $0.0500 | "Cheapest route by far. Use it for the quick, high-volume steps around umans-coder." |

### Important Distinctions

- Seat usage remains flat-rate — none of the above token pricing applies to human seats.
- Wallets can be topped up manually or set to auto-refill from **Billing → Wallet**.
- Per-key usage breakdowns are available in the ledger view under **Billing → Wallet**.

## Getting Started — Step by Step

1. **Contact the team.** Organizations are provisioned manually. Email contact@umans.ai with your team size and the first admin's email.
2. **First admin accepts the invitation.** The org is created and an invite link is sent to the named admin, who signs in to activate it.
3. **Admin buys seats** via Billing → Members → *Get More Seats*.
4. **Invite teammates.** They sign in and run `umans claude` normally — no extra setup required.
5. **Create a service account** for any automation via Billing → Wallet → *Create service account*. Store the key like any other secret and point your bot at `https://api.code.umans.ai`.

## Support Channels

- **Discord** — community Q&A at discord.gg/Q5hdNrk7Rw
- **Email** — contact@umans.ai
- **Dashboard** — manage the org at app.umans.ai/billing

## Footer / Site Navigation

- Tagline: "Umans · code. Built for serious agentic work."
- Links: User Guide, Terms, Privacy, Contact
- Social: Discord, X (Twitter), LinkedIn — all under the umans_ai handle
