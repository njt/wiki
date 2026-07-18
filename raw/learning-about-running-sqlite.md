---
url: https://jvns.ca/blog/2026/07/17/learning-about-running-sqlite/
title: "Learning a few things about running SQLite"
author: Julia Evans
date_fetched: 2026-07-18
date_published: 2026-07-17
---

# Learning a few things about running SQLite

Julia Evans describes working on a Django site with SQLite as its database. She read posts about using "SQLite in production for a small site" and agrees it's *totally fine*, but acknowledges she didn't fully grasp that databases are complicated and she lacks experience operating them.

This is her fourth SQLite-backed website, but Django's ORM causes the database to "do more work" than in prior projects. She started by enabling WAL mode, "like all the blog posts said to do," and hoped for the best.

## ANALYZE

While running an FTS5 full-text search query on a 4000-row table, it took 5 seconds. Running `ANALYZE` dropped it to roughly 0.05 seconds. She suspects the issue was "some sort of accidentally quadratic thing." `ANALYZE` generates statistics about row counts and other data so the query planner can make better decisions. She muses that "maybe one day I'll learn to read a query plan."

## Cleaning Up the Database

She describes a recurring scenario:

1. Running a command to delete unwanted rows (e.g., completed tasks from django-tasks-db)
2. The command takes over 5 seconds (she's unsure why DELETE statements are slow — possibly Python code running inside a transaction)
3. Another worker tries to write to the DB during this time and hits the 5-second timeout
4. The worker crashes, causing the VM to shut down

Her workaround is to perform cleanup "in small batches" to keep queries under 5 seconds. This gave her "more of an appreciation for why someone might want to use a 'real' database like Postgres" that supports concurrent writers. She considers taking the site down for scheduled maintenance but hasn't worked out that workflow yet.

## ORM Performance

She's been using Django's ORM freely without monitoring query performance and it's "mostly been going okay other than the ANALYZE thing." The database is small (~10,000 rows) and she expects it to stay that way.

## Backing Up SQLite

She describes two approaches, noting she hasn't actually tested restoring from backups but monitors them with a dead man's switch.

**Way 1: restic**

```bash
sqlite3 /data/calendar.db "VACUUM INTO '/tmp/calendar.sqlite'"
gzip /tmp/calendar.sqlite

# Upload backup to S3
# Sometimes the backup gets OOM killed and so it stays locked, do an unlock
restic -r s3://s3.amazonaws.com/some_bucket/ unlock
# Do the backup & prune old backups
restic -r s3://s3.amazonaws.com/some_bucket/ backup /tmp/calendar.sqlite.gz
restic -r s3://s3.amazonaws.com/some_bucket/ snapshots
restic -r s3://s3.amazonaws.com/some_bucket/ forget -l 1 -H 6 -d 2 -w 2 -m 2 -y 2
restic -r s3://s3.amazonaws.com/some_bucket/ prune
```

Restic backups were sometimes getting OOM-killed, which she found tiresome.

**Way 2: litestream**

She started trying Litestream for incremental backups, calling it potentially "more efficient." The config-driven approach:

```
litestream replicate -config litestream.yml
```

She set `retention: 400h` hoping to retain some database history but admitted "I have no idea if it works."

She backs up to AWS and finds the credential generation process annoying, considering a move to an S3-compatible alternative.

## Multiple Databases

Though her current project uses a single database, she previously split tables across three separate database files for [Mess with DNS](https://messwithdns.net/) since they didn't need to share a database. That project has run on SQLite for four years (since 2022) and "the move from Postgres was a great choice."

## Closing Reflection

Evans finds it fun to see how long it takes to learn "sort of basic things about the technologies" she uses. She'd been using SQLite for web projects since 2022 but only learned about `ANALYZE` the day she wrote this post. She expects to learn "some other very basic feature" in another year or two.

## References

- [The definitive guide to using Django with SQLite in production](https://alldjango.com/articles/definitive-guide-to-using-django-sqlite-in-production)
- [a gist on sqlite performance tuning](https://gist.github.com/phiresky/978d8e204f77feaa0ab5cca08d2d5b27)
