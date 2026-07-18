# SQLite is All You Need for Durable Workflows

The Obelisk team argues that durable workflow execution doesn't require durable infrastructure — workflow state needs persistence, but compute can stay cheap and disposable. SQLite gives you transactional durability without a separate database service, and Litestream bridges the gap from local file to portable backup by streaming changes to S3. For AI agent workloads in particular, the per-tenant SQLite + micro VM model is simpler, cheaper, and better-isolated than a shared Postgres cluster. The article is refreshingly honest about the trade-off: async replication means you can lose the newest writes on crash, and the authors explicitly say Postgres is the right answer when that matters.

---

## Key Quotes

> "Durable execution is often discussed as if it requires durable infrastructure."

The frame is the insight. The durable execution conversation keeps conflating state durability with infrastructure durability because we've been conditioned by distributed systems thinking. The article breaks them apart: one needs to survive, the other doesn't.

> "SQLite gives you transactional durable state without introducing a separate database service."

No network hop, no control plane, no new operational surface. This is the "right level of machinery" argument — not that SQLite is sufficient for everything, but that for a large class of systems, a local database file is exactly the right tool and anything more is overengineering.

> "A restore can miss the newest local writes if the SQLite volume disappears before they are copied."

The honesty is what makes this credible. Litestream replication is asynchronous. If your VM dies between the write and the S3 flush, those writes are gone. The article doesn't minimize this — it says this is acceptable for AI and experimentation workflows *and that it's not equivalent to a highly available shared database*.

> "The same file can be used for local replay, debugging, and understanding what an agent actually did."

The inspectability argument. A SQLite file is a self-contained artifact — you can copy it, open it, query it. This matters enormously for debugging agent behavior, which is already opaque enough without adding database indirection.

> "A fleet of small servers in micro VMs or containers, each with its own SQLite database and object storage backup, is simpler, cheaper, and gives better fault isolation."

The per-tenant architecture argument. This inverts the traditional database story (scale up, share, consolidate) and says: for agents, you want the opposite — isolation, simplicity, blast radius. This is the database equivalent of [[Smart Models Dumb Pipes]].

> "Many workflow systems do not need that on day one."

The discipline argument. Start with the infrastructure your state actually requires, not the infrastructure you imagine needing at scale. Postgres is the upgrade path, not the starting line.

---

## Key Themes

#concept #database #workflow #agents #infrastructure

---

## Critical Analysis

**The core insight is correct, and it's not new — but it needed saying in this context.** The "SQLite is enough" argument has been made for web apps (Litestream's original pitch), for edge computing ([[Graft]]), and for analytics (DuckDB). Applying it to durable workflows for AI agents is the natural extension, and the article makes it cleanly. What's new is the framing: durable execution ≠ durable infrastructure.

**The async replication caveat is more significant than the article suggests.** "Acceptable for AI and experimentation workflows" is doing a lot of work. If your agent has been running a multi-hour workflow and the VM dies before Litestream flushes, you lose that progress. For experimentation, fine. For production agents handling real work, that's a meaningful failure mode that needs explicit handling — and the article doesn't address what that handling looks like beyond "use Postgres instead."

**The per-tenant SQLite model is the right default for agents, full stop.** The article makes the architectural argument well: better fault isolation, simpler reasoning, cheaper operations. But it underplays the operational complexity of managing a fleet of SQLite databases. Litestream helps (backup), but questions like "how do you query across all tenants" or "how do you migrate schemas across a fleet" are left unanswered. These are the problems that shared databases solve, and they don't go away just because you've chosen SQLite.

**The "day one" argument is a discipline worth preserving.** The article's most important contribution might be the reminder that you don't need Postgres on day one. Too many agent startups reach for distributed Postgres before they have a single paying customer. The article gives permission to start simple — and names the conditions under which you should upgrade.

**The due diligence companion:** [[How to Corrupt an SQLite Database]] catalogs every failure mode the SQLite team knows about — the shadow side of "SQLite is all you need." If you're building on SQLite for durable workflows, that page is your operational checklist for which failure modes apply to your deployment and whether WAL mode + Litestream covers them.

**What's missing: the migration story.** If you start with SQLite + Litestream and later need Postgres, what's the path? The article mentions Obelisk supports both, but doesn't describe the operational experience of migrating. This is the gap between "start simple" and "grow up" that every architecture guide needs to address.

**Comparison to the wiki's database landscape:** This sits between [[Graft]] (SQLite + object storage for edge replication) and [[All Your Agents Are Going Async]] (durable state as the hard half of agent infrastructure). Unlike Graft, it's not trying to solve distributed writes — it's about single-tenant durability with backup. Unlike the async agents piece, it's concrete about the storage layer rather than the protocol layer. And unlike [[Dolt]] or [[CodeMira]], it's not trying to build a new database — it's arguing that vanilla SQLite with one well-chosen companion tool is the right answer for most cases. [[SQLite Is All You Need]] makes the parallel argument for web backends — same thesis, different domain, with the benchmark data this article lacks.

---

*Source: [[summary/sqlite-is-all-you-need-for-durable-workflows]]*
*Last updated: 2026-05-31*
