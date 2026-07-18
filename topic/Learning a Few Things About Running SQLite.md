# Learning a Few Things About Running SQLite

Julia Evans' field report from running SQLite in production for a Django site: what she learned about `ANALYZE`, backup strategies, concurrent-writer limitations, and the gap between "SQLite is fine for small sites" and actually operating a database. Four years into using SQLite for web projects, she's still discovering features she should have known — and the post is better for that honesty than any expert guide would be.

---

## Key Quotes

> "maybe one day I'll learn to read a query plan"

Evans' signature move: saying the thing every practitioner thinks but most are too self-conscious to write. A 5-second FTS5 query on a 4,000-row table dropped to 0.05 seconds after `ANALYZE`. That's a 100× speedup from a single command — and she'd never needed it before because her earlier SQLite projects didn't push enough through the ORM to trigger the query planner's bad decisions.

> "I just sort of enabled WAL mode, like all the blog posts said to do, and hoped for the best"

The production SQLite advice distilled to one sentence. Every guide says "enable WAL mode." Every guide is right. And enabling WAL mode is the beginning of operating a database, not the end — the `ANALYZE` discovery only happened because WAL mode made concurrent reads possible, which exposed the next bottleneck.

> "this gave me more of an appreciation for why someone might want to use a 'real' database like Postgres"

The scare quotes around "real" are doing the work. Evans isn't conceding that SQLite isn't real — she's acknowledging that SQLite's single-writer architecture has a concrete operational cost she hadn't felt before. The DELETE-causes-timeout-causes-crash cascade is the kind of thing you don't learn from blog posts about how SQLite is "fine for production." You learn it when your workers start dying.

> "I haven't actually tried restoring any of these SQLite backups, but I do have a dead man's switch in place monitoring them"

The most important sentence in the piece and the one most readers will skim past. She's running two backup strategies (restic + litestream), monitoring them, and *has never tested a restore*. This isn't criticism — it's the honest state of most small-site operations. The dead man's switch is the right level of paranoia for the scale: you know when backups stop working, even if you don't know whether they'd actually work.

> "I think it's fun to see how long it takes me to learn sort of basic things about the technologies I use"

The thesis hiding in the conclusion. She's been running SQLite in production since 2022 and learned `ANALYZE` in July 2026. These facts aren't failures — they're data points about how technology learning actually works. You use a tool successfully for years, then discover a feature that was there the whole time, and the tool gets better without changing.

---

## Key Themes

#database #sqlite #production #operations #learning #backup

---

## Critical Analysis

**The post's real subject isn't SQLite — it's the learning curve of production operations.** Evans frames it as "learning a few things about running SQLite," but every lesson generalizes: (1) your ORM will generate queries you don't understand, (2) your first production problem will expose a feature you should have known, (3) backups you haven't tested are wishes, not backups, (4) the gap between "it works on my machine" and "it works when two things happen at once" is wider than it looks.

**The concurrent-writer problem is SQLite's honest limitation, not a gotcha.** SQLite's single-writer architecture is well-documented. But "well-documented" doesn't mean "felt." Evans' DELETE-causes-timeout-causes-crash cascade is what it feels like: a routine cleanup job becomes an outage because there's no queueing, no row-level locking, no graceful degradation. Postgres handles this gracefully not because it's "real" but because it was designed for concurrent access from the start. SQLite wasn't, and no amount of WAL-mode enthusiasm changes that.

**Litestream vs. restic is a false choice — they solve different problems.** Evans runs both. restic gives her full `VACUUM INTO` backups at intervals; Litestream gives her continuous WAL streaming. But neither solves the restore-testing problem she admits to having. The right backup strategy isn't about which tool — it's about whether you've drilled the restore. At her scale (10,000 rows), the simplest thing that could possibly work is `VACUUM INTO` to a timestamped file and a cron job that deletes old ones. Litestream's incremental streaming is elegant but adds operational surface for a database that fits in memory.

**The four-year learning curve is the point, not the confession.** Evans discovers `ANALYZE` after four years of running SQLite in production. This isn't embarrassing — it's normal. Tools have depth you don't need until your workload changes. Her first three SQLite projects were simple enough that the query planner never made a bad call. The fourth one, with Django's ORM generating more complex queries on the same database, crossed a threshold. The lesson isn't "learn ANALYZE sooner" — it's "your tools have features you don't know about, and you'll discover them when your use case demands them."

**What's missing: the operational checklist.** Evans describes a series of discoveries (ANALYZE, batched deletes, backup monitoring) but doesn't consolidate them into a runbook. The post would be stronger with a "here's what I now check on every SQLite deployment" section. As it stands, it's a diary of learning — valuable for fellow travelers, less useful as a reference.

**Comparison to the wiki:** This complements [[SQLite is All You Need for Durable Workflows]] from the opposite direction. That page argues SQLite + Litestream is the right default for agent workflows *in theory*. Evans' post shows what the same stack looks like *in practice* — the rough edges, the discoveries, the things you don't know you don't know. The theoretical case is strong; the experiential case is messier and more useful. Together they make a more complete argument than either makes alone. Her observation that the concurrent-writer problem pushed her toward appreciating Postgres also maps to [[Constraint Decay]]'s finding that databases are the primary failure driver when constraints are imposed on coding agents — the same architectural properties that make SQLite simple also make it brittle under concurrent load. [[The GUS Stack — Go, Unix, SQLite]] takes the SQLite-as-default position as given and builds a full agent-optimized stack around it, treating Evans' operational lessons as table stakes rather than discoveries.

---

*Sources: [[raw/learning-about-running-sqlite]]*
*Last updated: 2026-07-18*
