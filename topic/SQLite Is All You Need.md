# SQLite Is All You Need

Jay at DB Pro builds a 50K-user social network on a single SQLite file, benchmarks it to 3,654 req/s on the heaviest query (315M requests/day), and argues that the reflex to provision a database server is a habit, not an engineering decision. The WAL-vs-rollback-journal comparison — 5.6× read throughput at 1,000 writes/s — is the article's most original empirical contribution. But the real argument is the "99.99%" frame: you don't have users yet, and if you ever get enough traffic that SQLite can't serve it, you'll have revenue, a team, and a clear signal about what to fix.

---

## Key Quotes

> "There is no step where you provision anything."

The operational simplicity argument compressed to one sentence. A query is a function call, not a network round trip. There is no connection pool because there is no connection. No `await` because there is no network to wait for. This is the unbenchmarked advantage that doesn't show up in req/s charts — the cognitive and operational elimination of an entire service category. [[Your Distributed System Is Slower Than a Laptop]] makes the same point about distributed streaming: the coordination tax is paid in salaries, not CPU cycles.

> "You are engineering for the traffic you imagine, in a future where you already won, and paying for it in complexity now."

The sharpest critique of premature infrastructure in the article. This isn't YAGNI as thrift — it's YAGNI as clarity. Provisioning Postgres for a weekend project isn't just wasteful; it's actively misleading yourself about where the hard problems are. [[The Cost YAGNI Was Never About]] reframes this in economic terms: cheap AI generation amplifies the YAGNI trap, not the escape. Building capacity you don't need is buying an option you have no reason to exercise.

> "SQLite takes a single write lock for the whole database. Writes do not run in parallel, they queue."

The article earns credibility by being honest about limitations. At 23,000 committed transactions/second the queue drains fast, but no amount of hardware makes it two queues. This is the difference between a benchmark that sells SQLite and an honest assessment that lets you decide. Contrast with database marketing that buries the single-writer constraint in a footnote.

> "Ship the file. Go and find the users. That is the hard part."

The closer. The authors built DB Pro themselves and report: "the hardest thing we work on is not query parsing, or schema diffing, or shipping a native app on three platforms. It is user acquisition." The database argument is secondary to the product argument — you're solving the wrong problem, and the database you provisioned is actually making the real problem harder by adding operational surface.

> "Never put SQLite on network storage."

One line of purchasing advice that captures a whole class of failure modes. EBS, NFS, and similar are "networks pretending to be disks" and will cause latency and correctness issues. This is the article at its most practically useful — not arguing that SQLite is magic, but specifying the conditions under which it works.

## Key Themes

#database #sqlite #benchmark #simplicity #pattern #concept

- **WAL mode as the enabler**: The article's empirical contribution is quantifying what WAL buys you. Rollback journal falls to 497 reads/s with 1,000 writes/s and p99 of 134ms; WAL serves 5.6× the reads at p99 4.40ms. This gap is the reason SQLite's old reputation (locking the whole file on every write) no longer applies. [[SQLite is All You Need for Durable Workflows]] makes the same argument for agent workflow state but doesn't provide the comparative data.
- **The honest mixed-workload number is ~2,800 reads/s**: The article doesn't lead with the 51K req/s point-read number. The real figure — seven reader threads plus writes — is 2,800 of the heaviest query. This is still more than most products need, but it's the number that matters for architecture decisions, not the headline.
- **STRICT tables are table stakes**: The endorsement is calibrated — STRICT costs nothing, catches type errors, and the only cost is that you can't ALTER into it. Evan Hahn's argument, filtered through the article's production lens. [[Constraint Decay]] finds databases are the primary failure driver for coding agents; STRICT tables are a cheap deterministic guard that catches a class of errors agents routinely make.
- **The "99.99%" argument is really about attention allocation**: The authors aren't arguing SQLite handles 99.99% of workloads. They're arguing 99.99% of new products never need more than SQLite, and the energy spent on database infrastructure is energy not spent on finding users. This is the article's most important idea, and it's not technical.
- **Backups are solved**: `VACUUM INTO` with zero writer interruption at 45K writes/s, plus Litestream for continuous replication. The backup story for SQLite is simpler than for any client-server database — and the article demonstrates it under load, not in theory. [[Graft]] maps the broader replicated SQLite landscape (Litestream, cr-sqlite, mvSQLite, Turso, rqlite).
- **Measure your own hot path**: The Bun-vs-Node comparison (Bun +8% on cheap queries, −21% on the heavy one) is a meta-lesson. The article isn't selling better-sqlite3; it's demonstrating that runtime choice depends on workload shape, and "the benchmark on the internet" doesn't know your query patterns.

## Critical Analysis

**This article matters because it's benchmarked, not because it's original.** "SQLite is production-ready" has been argued before (Litestream's docs, the Obelisk article, half of Hacker News). What's new is the data: a real workload, real WAL-vs-rollback comparison, real cloud cost estimates, real backup-under-load measurement. The article doesn't need to be novel — it needs to be convincing, and the data does the convincing.

**The single-writer limitation is the real ceiling, and the article is appropriately honest about it.** WAL mode solves the "readers blocked by writers" problem that gave SQLite its bad reputation. It does not solve the "only one writer at a time" problem. For workloads where writes are the bottleneck — chat apps, gaming leaderboards, financial ledgers — the 23K transactions/second will eventually be a ceiling. But the article's framing is right: if you hit that ceiling, you have a good problem to have.

**The cloud estimates are directionally correct and should be treated as such.** The ±30% caveat is honest. The real takeaway isn't the specific req/s number for a Hetzner CAX21 — it's that the order of magnitude is "hundreds of millions of requests per day on a $7–28/month VPS." That changes the default. The question stops being "why SQLite?" and becomes "why not SQLite?"

**What's missing: the migration story, again.** The article names the conditions for using Postgres but doesn't describe the experience of moving from SQLite to Postgres when those conditions arrive. This is the same gap noted in [[SQLite is All You Need for Durable Workflows]]'s analysis. The "you'll have revenue and a team" dismissal is true but incomplete — migration difficulty is a function of how deeply the application code is coupled to the database, and SQLite's lack of client-server architecture means you can't just point the app at a different hostname and keep going. You have to rewrite the data layer.

**The STRICT table argument underplays the migration cost for existing SQLite databases.** "You cannot ALTER an existing table into strictness" is a footnote, but for any non-trivial existing SQLite database, converting to STRICT means a migration that touches every table. The article is targeting greenfield projects (the Chirp demo), and for those, START STRICT is good advice. For existing projects, the cost-benefit is less clear.

**The article's strongest contribution is the reframe from technical to practical.** The benchmark data is good. The WAL comparison is original. But the "99.99%" argument — that the real bottleneck is user acquisition, not database throughput — is what makes this worth reading even if you already believe SQLite is production-ready. It's not a database article with a product argument bolted on; the product argument is the point, and the database benchmarks are the supporting evidence.

**Connection to the wiki's database landscape:** This sits alongside [[SQLite is All You Need for Durable Workflows]] (same thesis, different domain — workflow state vs. web backends) and [[Graft]] (the replicated SQLite landscape). Together they form a coherent position: SQLite is the right default, Litestream solves backup, and the replicated-SQLite ecosystem (Graft, cr-sqlite, Turso) handles the read-scaling cases where a single file isn't enough. [[Postgres Transactions Are a Distributed Systems Superpower]] is the counterpoint — when you genuinely need Postgres, its transactional guarantees at scale are unmatched. The articles don't disagree; they name different breakpoints.

The Postgres side of that breakpoint is what [[Rust Scalable Backend Services]] assumes as its baseline: Kerkour's Rust services use Postgres for *everything* — job queue, cron leader election via advisory locks, cache invalidation at the service layer — the very infrastructure a single SQLite file makes disappear. The two articles name different bands of the same spectrum: SQLite until you've won, Postgres once your service is big enough that "medium-sized" (10K+ lines, ~100 endpoints) is a deliberate target rather than an accident.

The [[DuckDB ADBC Extension]] page documents a parallel story: a single-file database that handles analytical workloads that used to require clusters. SQLite for OLTP, DuckDB for OLAP — together they cover most of what small teams actually need. [[Your Distributed System Is Slower Than a Laptop]] makes the same argument from the infrastructure side: measure the single-machine baseline before building a distributed system. These three articles are the same thesis applied to different layers of the stack. [[The GUS Stack — Go, Unix, SQLite]] codifies this position as a full-stack prescription specifically optimized for AI coding agents: Go for the language, Unix for the OS layer, SQLite for persistence, and HTMX for server-rendered interactivity — a stack the models already know.

[[Grimmory]] extends the same discipline to a client-server stack: a MariaDB-backed self-hosted library that caps its Hikari pool at 5 connections, runs on virtual threads, and computes recommendations by brute-force cosine over hand-rolled embeddings — shrinking every component to the traffic a single household actually produces, rather than provisioning for a fleet that may never arrive.

---

*Sources: [[raw/sqlite-is-all-you-need]]*
*Last updated: 2026-07-18*
