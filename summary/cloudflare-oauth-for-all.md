---
url: https://blog.cloudflare.com/oauth-for-all/
title: "Unlocking the Cloudflare App Ecosystem with OAuth for All"
author: Sam Cabell, Mike Escalante, Adam Bouhmad, Nick Comer
date_fetched: 2026-07-03
date_published: 2026-06-24
topics:
  - developer-tools
---

# Unlocking the Cloudflare App Ecosystem with OAuth for All

Cloudflare opened up self-managed OAuth to all developers, enabling them to create and manage OAuth clients for delegated API access. Previously, third-party OAuth was limited to a small number of manually onboarded partnerships. The company performed a zero-downtime migration of its underlying OAuth engine (Hydra, an open-source project) to make this possible.

## Scaling the Ecosystem Securely

The earlier OAuth solution was sufficient for a small number of managed partners, but Cloudflare recognized that the permissions model, consent experience, and abuse mitigation were not mature enough. They updated the consent experience to clarify which app requests access and what permissions it receives, added dashboard revocation, and made app ownership more visible to prevent OAuth phishing attacks.

## Planning the Upgrade to the OAuth Engine

Cloudflare had deployed Hydra years ago. As usage grew, a major upgrade was needed. The team chose two sequential upgrades (1.X then 2.X) rather than one large jump.

Key challenges with the 1.X upgrade:
- Schema migrations "would claim an exclusive lock on critical tables"
- Added columns and moved data to new tables
- The SDK performed `SELECT *` operations, causing deserialization issues with schema changes

Their solution: rewrote SQL migrations using `CREATE INDEX CONCURRENTLY` and built a "custom version of Hydra which selected explicit columns."

For the 2.X upgrade, they evaluated three options and chose a **blue-green strategy**. They extended token expiry to multiple hours so existing tokens wouldn't need refreshing during the transition. To avoid losing revocations during the switch, they created a queue system using Cloudflare Queues that recorded revocation events, allowing them to be replayed after the database cutover.

## Executing the Upgrade

**Upgrading to 1.X:** The custom migrations ran faster than expected with no user impact. A hard cutover was required since the old version "was unable to introspect tokens that were created by the newer version." Post-upgrade, they saw increased refresh token errors due to stricter invalidation—reused refresh tokens would invalidate the entire chain. This affected Wrangler and MCP clients. They mitigated this by adding "refresh token coalescing behavior" to their routing Worker, caching refresh requests to short-circuit retries.

**Upgrading to 2.X:** The full process involved: enabling a revocation replay queue, copying and restoring the database, targeted data cleanup, simultaneous configuration cutovers across multiple systems, and post-cutover monitoring. The production migrations ran approximately three hours. After cutover, an overeager data cleanup job in their authorization service caused increased 403 errors. Investigation revealed "an issue in one of the Hydra migrations that corrupted the state of certain valid OAuth sessions." They performed data restorations and reduced reliance on static policy data.

## Performance Improvements

Database migration metrics:
- Rows updated: 132.5M
- Rows inserted: 114.7M
- Temp bytes: 136.97GB
- Transaction commits: 22.2k

Hydra performance (before vs. after):
- API P95 latency: 185ms → 101ms (-45%)
- RSS memory: 888MB → 763MB (-14%)
- Go heap alloc: 449MB → 271MB (-40%)
- Goroutines: 4015 → 3076 (-23%)
- CPU: 1.07 cores → 0.67 cores (-37%)

## Self-Managed OAuth for All

The release on June 3, 2026, allows any Cloudflare customer to create their own OAuth applications. The authors describe this as "an important step toward a broader Cloudflare app ecosystem." Developers can now build SaaS integrations, internal developer platforms, and agentic tools with standard OAuth flows.

## Notable Quotes

- "self-managed OAuth" — making it easier to create and manage OAuth clients for delegated access
- "opening up OAuth to all customers was critical to the success of our platform"
- "a blue-green strategy would work"
- "nobody would be able to use existing OAuth apps unless they already had a valid credential"
- "replaying all revocation events that took place in the time window in which they would have been lost"
