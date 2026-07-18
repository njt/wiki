---
url: https://www.dbpro.app/blog/sqlite-is-all-you-need
title: "SQLite Is All You Need"
author: Jay
date_fetched: 2026-07-18
date_published: 2026-07-14
site: DB Pro Blog
---

# SQLite Is All You Need

Jay, DB Pro Blog, July 14, 2026

The authors built a social network called Chirp on SQLite — 50,000 users and 1 million posts in a single 343MB file — to test whether SQLite can serve as a production database for real applications. Their conclusion is that for the vast majority of workloads, it absolutely can.

## What They Built

Chirp's dataset in one file:

| Table | Rows |
|---|---|
| users | 50,000 |
| posts | 1,000,000 |
| follows | 2,498,799 |
| likes | 4,999,764 |
| **Total** | **343 MB** |

Every table uses STRICT mode. The backend is a single Node process talking directly to chirp.db — no database server.

## Production Configuration (Five Pragmas)

```javascript
const db = new Database('chirp.db');
db.pragma('journal_mode = WAL');
db.pragma('synchronous = NORMAL');
db.pragma('busy_timeout = 5000');
db.pragma('foreign_keys = ON');
db.pragma('cache_size = -64000');
```

"There is no step where you provision anything."

## Benchmark Results

All tests ran on an Apple M1 laptop (8 cores, 16GB RAM), Node 22 with better-sqlite3 (SQLite 3.49.2).

### End-to-End HTTP Benchmarks (50 concurrent connections)

| Endpoint | req/s | p50 | p99 | Errors |
|---|---|---|---|---|
| GET /post/:id | 51,427 | 0ms | 1ms | 0 |
| GET /u/:handle | 47,776 | 0ms | 2ms | 0 |
| GET /timeline/:id | 3,543 | 13ms | 27ms | 0 |
| Mixed (95% reads, 5% writes) | 3,654 | 13ms | 27ms | 0 |

The heavy timeline query hit 3,654 req/s. The authors calculate "315 million requests a day" on the heaviest endpoint.

### Raw Query Benchmarks

| Query | ops/s | p50 | p99 |
|---|---|---|---|
| Point read (post by id) | 232,011 | 0.004ms | 0.007ms |
| Profile page (two aggregates) | 175,850 | 0.005ms | 0.007ms |
| Home timeline (20 posts + like counts) | 4,247 | 0.232ms | 0.337ms |
| Insert a post (one transaction each) | 23,459 | 0.012ms | 0.089ms |
| Like a post (one transaction each) | 12,618 | 0.016ms | 0.243ms |
| Insert posts (batched, 100 per transaction) | 32,217 rows/s | — | — |

Every write is a "real, committed, durable transaction. Not a batch, not a buffer, not a queue."

## WAL vs. Rollback Journal Comparison

### WAL Mode

| Scenario | reads/s | p99 read | worst read | SQLITE_BUSY |
|---|---|---|---|---|
| Readers only | 17,581 | 1.53ms | 7ms | 0 |
| Readers + 1,000 writes/s | 2,792 | 4.40ms | 17ms | 0 |
| Readers + writer flat out (14,838 w/s) | 2,854 | 5.16ms | 30ms | 0 |

### Rollback Journal (DELETE)

| Scenario | reads/s | p99 read | worst read | SQLITE_BUSY |
|---|---|---|---|---|
| Readers only | 18,439 | 1.23ms | 6ms | 0 |
| Readers + 1,000 writes/s | 497 | 133.85ms | 794ms | 0 |
| Readers + writer flat out (2,806 w/s) | 227 | 586.02ms | 1,762ms | 0 |

Key finding: with a writer at 1,000 writes/s, "WAL serves 5.6x the read throughput" with p99 at 4.40ms vs. 133ms for the rollback journal. Zero SQLITE_BUSY errors under WAL across all scenarios.

## Where SQLite Falls Over (Limitations)

1. Read throughput drops when writes are active — 6.3x fall from 17,581 reads/s to 2,792 with just 1,000 writes/s
2. Single global writer — writes queue, no amount of hardware makes it two queues
3. Single machine, no failover — recovery time equals however long restoring a file takes

### When to reach for Postgres

"Many writers contending on the same rows," needing "read replicas or automatic failover," requiring "a real analytics engine over hundreds of millions of rows," or genuinely needing "the extension ecosystem." But "we might scale one day" is not a good reason.

## Cloud Estimates

| Machine | ~$/mo | Est. timeline req/s | Est. requests/day |
|---|---|---|---|
| Apple M1 (measured) | n/a | 3,543 | 315M |
| Hetzner CAX21 (Ampere Arm) | ~€7 | 1,600–1,900 | ~140M–165M |
| Hetzner CPX31 (AMD) | ~€13 | 1,900–2,300 | ~165M–200M |
| Hetzner CCX13 (dedicated AMD) | ~€13 | 2,000–2,400 | ~175M–205M |
| DigitalOcean Basic (1 vCPU) | ~$12 | 1,300–1,600 | ~110M–140M |
| DigitalOcean Premium AMD (2 vCPU) | ~$28 | 1,800–2,100 | ~155M–180M |

"a $12 droplet serves something like 110 million timeline requests a day"

### Purchasing Advice

- Buy single-core speed, not core count
- Buy enough RAM to hold the database
- Insist on local NVMe — "Never put SQLite on network storage"
- Pay for dedicated cores if p99 matters

## The "99.99%" Argument

Most new apps don't have users — "that is not an insult, it is the base rate."

"You are engineering for the traffic you imagine, in a future where you already won, and paying for it in complexity now."

"If you ever get enough traffic that SQLite cannot serve it, you will have revenue, a team, and an extremely clear signal about exactly what to fix."

"Ship the file. Go and find the users. That is the hard part."

## STRICT Tables

Default SQLite accepts any value type in any column. STRICT enforces types and rejects non-existent types like UUID, DATETIME, or JSON.

## Production Code

Connection setup, prepared statements, HTTP server, and transactions — all in ~50 lines. Notable: "There is no connection pool, because there is no connection. There is no await on the query, because there is no network to wait for."

## Bun vs. Node

On cheap queries, Bun is 8-9% faster. On the heavy timeline query, better-sqlite3 is 21% faster. The article calls this "a reason to measure your own hot path instead of picking a runtime from a benchmark on the internet."

## Backups

"backup taken while 183,608 writes landed (45,902 writes/s)" — completed in 4,060ms, zero violations. Shell one-liner: sqlite3 chirp.db "VACUUM INTO 'backup-$(date +%F).db'"

For continuous replication: Litestream.

## Operational Benefits

- Local development is one file
- Resetting is rm
- Tests get a real database each
- Deploys are a binary and a file
- Nothing to operate — "Nothing to upgrade, monitor, or pay for separately"
