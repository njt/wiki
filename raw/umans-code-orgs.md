---
url: https://app.umans.ai/offers/code/docs/orgs
date_fetched: 2026-07-05
backfilled: true
---

# Umans Code for Organizations

**One wallet. Seats for your team. Service-account keys for automations.**

This page extends the User Guide. CLI install, tool setup, models, and the API work the same way for organizations. What follows is only what changes when you bring a team.

## At a glance

- **Seats**for humans. Flat-rate per person, generous usage that stays out of your way, same CLI flow as individual plans.
- **Service-account keys**for automations. Pay per token, from a shared organization wallet.
- **One invoice.**Seat subscriptions and service-account usage settle on the same organization billing.

## Seats (for humans)

Admins buy seats from **Billing → Members** and assign them to teammates. Each seat includes:

- **Generous usage built in, 4 parallel sessions**(any 10-minute window).
- Same models as individual plans (`umans-coder`,`umans-kimi-k2.7`,`umans-flash`,`umans-glm-5.2`).
- Teammates follow the standard Quick Start. Run `umans claude`and they're in.

| Monthly | Yearly | |
|---|---|---|
| Seat | $50 / seat / month | $500 / seat / year (~17% off) | 
| Extra session | $20 each | $20 each | 

### Sessions pool across the org

Extra sessions beyond a seat's 4 are $20 each. **Unused capacity from other seats in the same org offsets overages at 50%**: two unused session-slots cancel one extra session. Nothing is charged if spare capacity covers the overage.

## Service accounts (for automations)

Service-account keys run without a human behind them. They draw from the **organization wallet** at per-token rates. You only pay for what the automation actually consumes.

**Availability:** Token-based billing is available **only** to organizations, and **only** through service-account keys. Personal accounts use the flat Pro / Max subscription plans.

### What teams use them for

- **Production alert triage & mitigation.**A service-account bot watches alerts (PagerDuty, Sentry, Grafana), pulls context from the repo, and drafts a qualified mitigation plan before a human is woken up. Dollars of tokens per incident instead of a full seat.
- **Code review on every PR.**Run a review pass automatically with the same models your engineers use in their IDE. Typically a fraction of the cost of hosted alternatives, and consistent with how the team already codes.
- **Scheduled maintenance.**Migration passes, dependency bumps, flaky-test investigations. Anything you'd normally hand to an agent on a cron.

Good fits share a shape: non-interactive, bounded token budget, value per run is measurable.

## Token pricing

Per-model rates for service-account usage. Prices in USD per 1 million tokens. Each request debits exactly what the model consumed (input, output, cache reads, cache writes), with no per-call minimums.

| Model | Origin model | Input / 1M | Output / 1M | Cache read / 1M | Notes | 
|---|---|---|---|---|---|
| umans-kimi-k2.7 | kimi-k2.7-code | $0.9500 | $4.0000 | $0.1900 | What umans-coder routes to today (the recommended default), so default usage bills at these rates. Built for deep, multi-step coding work. | 
| umans-glm-5.2 | glm-5.2 | $1.4000 | $4.4000 | $0.2600 | Latest GLM with the largest context window; slightly cheaper cache reads than 5.1. | 
| umans-glm-5.1 | glm-5.1 | $1.4000 | $4.4000 | $0.2900 | Previous GLM, superseded by 5.2. Still billable for existing automations while we wind it down. | 
| umans-flash | qwen3.6-35b-a3b | $0.1500 | $1.0000 | $0.0500 | Cheapest route by far. Use it for the quick, high-volume steps around umans-coder. | 

- Seat usage stays flat-rate. Nothing here applies to human seats.
- Top up the wallet manually or enable auto-refill from **Billing → Wallet**.
- Per-key usage breakdowns live in **Billing → Wallet**(ledger view).

## Getting started

- **Talk to us.**Organizations are provisioned by our team. Email contact@umans.ai with your team size and the email of your first admin.
- **Your first admin accepts the invitation.**We create the org and send an invite link to the admin you named. They sign in to activate it.
- **Admin buys seats**for the team: Billing → Members →- *Get More Seats*.
- **Invite teammates.**They sign in and run- `umans claude`normally. No extra setup.
- **Create a service account**for any automation (Billing → Wallet →- *Create service account*). Store the key like any other secret and point your bot at- `https://api.code.umans.ai`.
