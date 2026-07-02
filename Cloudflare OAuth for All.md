# Cloudflare OAuth for All

Cloudflare's engineering postmortem on opening self-managed OAuth to all customers: a zero-downtime, two-upgrade migration of their Hydra-based OAuth engine (132M rows, 137GB of temp data), the performance wins they got (-45% P95 latency, -40% heap), and what broke along the way. The product outcome — any developer can now create OAuth apps for the Cloudflare platform — is secondary to the migration story, which is a clinic in blue-green database cutovers with queue-based revocation replay.

---

## The Migration Pattern

Cloudflare had two problems: schema migrations that locked critical tables, and an SDK that did `SELECT *` (causing deserialization failures when columns changed). Their solutions are worth stealing directly:

**For schema locking:** Rewrote migrations to use `CREATE INDEX CONCURRENTLY` instead of the defaults. Standard Postgres wisdom, but the discipline to do it across an entire third-party codebase's migration history is notable.

**For `SELECT *`:** Built a custom Hydra fork that selected explicit columns. This is the same fight every ORM user eventually has — the framework optimizes for developer ergonomics, and you optimize for production safety. The fork was the right call; schema changes should never break reads.

**For the 2.X blue-green:** Extended token expiry to multiple hours so existing tokens survived the cutover window. Then built a Cloudflare Queues-based revocation replay system — revocations during the transition were recorded and replayed after the database move. This is the pattern: when you can't make the cutover atomic, make the state window recoverable.

> "replaying all revocation events that took place in the time window in which they would have been lost"

## What Broke

Two production incidents, both instructive:

1. **Refresh token chain invalidation.** Post-1.X, reused refresh tokens invalidated the entire token chain — a stricter policy than before. Wrangler and MCP clients got hit hardest because their retry logic would burn through refresh tokens. The fix: "refresh token coalescing behavior" in their routing Worker — caching refresh requests to short-circuit retries. Classic distributed systems pattern: when clients retry, make the retries idempotent at the edge.

2. **Overeager data cleanup → 403s.** Post-2.X, an authorization service cleanup job was too aggressive, and a Hydra migration corrupted valid OAuth sessions. Required data restoration and a reduction in static policy reliance. The lesson: migration validation needs to test *session continuity*, not just data integrity. A row can be valid and still break the user.

## The Numbers

The scale is worth noting because it's real production data, not synthetic benchmarks:

| Metric | Before | After | Delta |
|--------|--------|-------|-------|
| API P95 latency | 185ms | 101ms | -45% |
| Go heap alloc | 449MB | 271MB | -40% |
| CPU | 1.07 cores | 0.67 cores | -37% |
| Goroutines | 4,015 | 3,076 | -23% |

The migration itself touched 247M rows (132.5M updated, 114.7M inserted) with 22.2K transaction commits. Three hours of production migration time.

## Why This Matters Beyond Cloudflare

Self-managed OAuth sounds like a product feature. The real story is what it enables: a platform where third-party developers can build delegated-access apps without Cloudflare manually onboarding each partner. This is the same thesis as [[Cloudflare Temporary Accounts for Agents]] — make the platform self-service for machines, not just humans.

The June 3, 2026 release means any Cloudflare customer can now:
- Create OAuth applications for SaaS integrations
- Build internal developer platforms with delegated API access
- Ship agentic tools that use standard OAuth flows instead of API-key-sharing hacks

That last one is the quiet signal. When the post mentions "agentic tools" in the same breath as OAuth, it's acknowledging that agents are becoming a first-class consumer of auth infrastructure — not an afterthought. This connects directly to [[Enterprise-Managed MCP Authorization]]'s thesis that auth infrastructure needs to work for agents as well as humans, and to [[Zero Trust for AI Agents]]'s Least Agency principle.

## Key Themes

#oauth #authorization #platform-engineering #database-migration #blue-green #zero-downtime #cloudflare #pattern

## Critical Analysis

**The migration writeup is the real product.** Cloudflare's blog posts are consistently good at operational transparency, and this one continues the tradition. The decision to do two sequential upgrades rather than one big jump, the explicit evaluation of three options for 2.X, and the honest accounting of what broke — this is what real infrastructure work looks like. Most companies publish "we migrated and it was fine." Cloudflare publishes "here's what we broke and how we fixed it."

**The `SELECT *` problem is a proxy for a deeper issue.** The Hydra SDK doing `SELECT *` isn't just bad SQL hygiene — it's a coupling problem between application code and database schema. Every ORM and framework has this tension. Cloudflare's solution (fork and fix) works at their scale, but the general lesson is simpler: if you're building on a framework that abstracts the database, budget for the day you'll need to break the abstraction. The explicit-column fork is the escape hatch.

**The revocation queue is the most interesting architectural detail.** Rather than trying to make the cutover atomic (impossible at this scale), they made it recoverable. Record revocation events during the transition window, replay them after. This is the event-sourcing pattern applied to auth state, and it's applicable far beyond OAuth migrations. Any system with mutable state where you can't afford to lose updates during a migration should consider this pattern.

**What the post doesn't say:** The three-hour migration window implies a ~3-hour window where token revocation was eventually-consistent rather than immediately-enforced. For most applications this is acceptable. For security-critical revocations (compromised credentials), it's a real gap. The post doesn't address this distinction, which matters if you're considering a similar pattern for your own auth infrastructure.

**The agent angle is under-explored.** The post mentions "agentic tools" once and notes that Wrangler/MCP clients were affected by the refresh token issue. But the broader implication — that self-managed OAuth is infrastructure for an agent-native platform — is left implicit. Given [[Cloudflare Temporary Accounts for Agents]] was published five days earlier, the connective tissue is visible but not stated: temporary accounts solve agent *provisioning*, and self-managed OAuth solves agent *delegation*. Together they're a full agent auth stack.

**Compared to [[An Illustrated Guide to OAuth]]:** Bhargava's explainer covers *what* OAuth is and *why* it's designed that way. This Cloudflare post covers *how to run OAuth at scale* and *what breaks when you upgrade it*. They're complementary — the former is protocol design, the latter is protocol operations. If you're building on OAuth, you need both.

---

*Sources: [[raw/cloudflare-oauth-for-all]]*
*Last updated: 2026-07-03*
