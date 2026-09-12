---
url: https://blog.cloudflare.com/wallets/
title: Announcing Cloudflare Wallets: the programmable wallet for the agentic Internet
author: Cloudflare
date_published: 2025-08-06
date_fetched: 2026-08-06
topics:
  - agent-architecture
---

# Cloudflare Wallets

Cloudflare announces Cloudflare Wallets, a programmable wallet system for the agentic Internet. The product addresses two structural barriers for AI agents: they lack stable identifiers to sign up for APIs, and they have no native way to pay for them. Wallets provide both, built on stablecoins and the x402 micropayment protocol.

## Two Wallet Types

**Account Wallets** are for humans — they hold funds, delegate spending to agents, and set policies. **Virtual Wallets** are for agents — they operate via API keys with spending caps, allowances, and allow lists that let agents explore autonomously within safe bounds. The key insight is that spending limits aren't constraints on agents; they're enablers. A $10 cap gives an agent more real freedom than a $1,000 budget because the human worries less.

## Identity via cloudflare.pay

Each wallet links to a Cloudflare account through a `cloudflare.pay` domain, giving agents optional, human-readable identities (e.g., `research.example.cloudflare.pay`). This lets merchants know an agent acts on behalf of a specific organization. Cloudflare explicitly compares this to VPN identification: unidentified agents aren't untrustworthy, but they need to prove themselves more. The identity primitive builds on existing Cloudflare infrastructure — Turnstile, Bot Management, and Web Bot Auth.

## The Two-Sided Market

Wallets are one side of a two-sided market play. **Monetization Gateway** (announced earlier) lets Cloudflare customers sell resources headlessly to agents via x402 micropayments. Wallets let agents buy from those merchants. Together they form the payment rails for a headless, machine-to-machine Internet economy.

## Beyond Payments

Cloudflare explicitly positions Wallets as one building block among several for agentic commerce: Monetization Gateway (selling), Wallets (buying), and Identity (attribution). The bet is that with bots driving a majority of web traffic, agents need first-class economic infrastructure — not human-account workarounds.
