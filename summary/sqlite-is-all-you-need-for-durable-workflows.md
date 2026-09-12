---
url: https://obeli.sk/blog/sqlite-is-all-you-need-for-durable-workflows/
title: SQLite is All You Need for Durable Workflows
author: Obelisk (obeli.sk)
date_fetched: 2026-05-31
date_published: 2026-05-29
topics:
  - databases-and-data
---

# SQLite is All You Need for Durable Workflows

## The Durable Part

The article argues that "durable execution is often discussed as if it requires durable infrastructure" but contends this isn't always true. The core claim is that workflow state is what needs durability, while compute resources can remain "cheap and disposable."

The author explains this fits naturally with Obelisk's model: workflow progress lives in an execution log, workflows replay from persisted history, and activities support retries. The key priority is keeping workflow state "easy to inspect."

## Why SQLite Fits

SQLite is attractive because it "gives you transactional durable state without introducing a separate database service." There's no network hop, no extra control plane, and no new operational overhead just for workflow safety. The author states that for many systems, "a local database file is exactly the right level of machinery."

## Litestream Makes It Portable

The article addresses the natural concern about accumulating SQLite files by pointing to Litestream, which "can stream SQLite changes asynchronously to S3-compatible object storage." This enables keeping state close to the runtime while still copying databases for backup, migration, and inspection.

The article notes a caveat: Litestream replication is asynchronous, so "a restore can miss the newest local writes if the SQLite volume disappears before they are copied." The author acknowledges this is acceptable for AI and experimentation workflows but isn't equivalent to a highly available shared database.

The proposed operating model: run an Obelisk server with SQLite, back it up via Litestream, and let an observer pull interesting databases as needed. "The same file can be used for local replay, debugging, and understanding what an agent actually did."

## Why This Works Well For Agents

The article highlights this approach as "especially attractive for AI agents and AI-generated workflows." These systems tend to be bursty and experimental, and are easier to reason about when each agent or tenant has "a small self-contained unit of state." A fleet of small servers in micro VMs or containers, each with its own SQLite database and object storage backup, is described as "simpler, cheaper, and gives better fault isolation" compared to a single large shared system.

## When To Use Postgres Instead

The author acknowledges that "SQLite is not the answer to every deployment shape." Obelisk also supports Postgres, which is appropriate when you need higher availability, broader scalability, or when asynchronous object storage replication isn't the durability model you want.

The article notes that "many workflow systems do not need that on day one" and shouldn't start with more infrastructure than their state requires.

The closing position is that for a large set of cases, "a local SQLite database plus Litestream backup to S3 is enough" — and for the world of AI agents, "that may be the most sensible default."
