---
url: https://code.storage/
title: "code.storage — Off the shelf Git infrastructure for your AI application"
author: Pierre Computer Company (CEO: Jacob, jacob@pierre.co)
date_fetched: 2026-05-31
date_published: unknown
---

# code.storage

"Off the shelf Git infrastructure for machines." API-first Git infrastructure product by Pierre Computer Company. $23M raised led by CRV + O1A.

## Value Proposition

Programmable Git repo creation via API — designed for AI-driven coding platforms, agentic frameworks, and applications that need to spin up Git repos programmatically. Avoids rate limits and complex auth flows of traditional Git hosting.

```js
const store = new GitStorage({ name: 'test', key });
const repo = await store.createRepo('repo');
const remote = await repo.getRemoteUrl(); // test.code.storage/repo
```

## Performance

- 60x faster clones than all R2/S3-based storage solutions
- Sharded on distributed Git ref storage, replicated 3+ times
- Can be colocated near agents or on customer hardware
- Warm and cold storage tiers

## Reliability

- 99.99% SLA for multi-AZ cloud deployments
- Transparent failover, zero-downtime migrations, guaranteed consistency
- Self-managed distributions for enterprise
- Live status at status.code.storage

## Features

1. Custom Git endpoints — expose `git clone`, `push`, `fetch` under your own domain
2. Full read/write via SDKs — TypeScript, Python, Go
3. Webhooks for build systems, bots, and AI agents
4. GitHub sync engine — first-class support for GitHub-backed storage

## Pricing

| Tier | Price |
|------|-------|
| Warm storage (touched <7 days) | $1.00/GB/month per replica |
| Cold storage (untouched >7 days) | $0.15/GB/month |
| Inbound bandwidth (push/write) | $0.06/GB |
| Outbound bandwidth (clone/fetch) | $0.15/GB |

Usage-based pricing, BYO cloud via Managed Code Storage, enterprise commitment discounts.

## Security

- Fine-grained audit logs and access controls
- Per-tenant deployments and encryption
- Annual third-party penetration tests
- External code audits

## Team

Built by Pierre Computer Company. Team has "over 150 years of expertise" building distributed systems at Cloudflare, Coinbase, Discord, GitHub, Reddit, Stripe, and X.

Mission: "build the next generation of infrastructure for collaborative computing."

## Taglines

- "YOUR CODE IS A BLOB"
- "WE HOLD YOUR BLOBS IN STORAGE"
- "EACH STORED BLOB IS BACKED BY A GIT REPOSITORY"

The API exposes classic Git workflows, new AI-native Git workflows, and a GitHub sync engine.
