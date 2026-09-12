# AliSQL

Alibaba's MySQL fork that grafts DuckDB's columnar OLAP engine and native vector search onto MySQL. Use your existing MySQL tools and drivers -- zero learning curve -- but get 200x analytical query speedup and HNSW vector indexing with up to 16,383 dimensions.

---

## Key Quotes

> "Use your existing MySQL tools, drivers, and SQL -- zero learning curve."

## Key Themes

#database #vector-db

AliSQL's strategy is "don't make people switch databases." Instead of building yet another new database, they take the world's most popular open-source RDBMS and add the capabilities that modern workloads need: fast analytics (DuckDB engine) and vector search (HNSW with cosine and Euclidean distance).

The 200x speedup on analytical queries comes from DuckDB's columnar storage with automatic compression -- fundamentally different from InnoDB's row-oriented architecture. This means AliSQL can handle OLAP workloads that would previously require a separate analytical database or data warehouse.

Battle-tested across millions of databases in Alibaba's production environment. GPL-2.0 licensed (same as MySQL), current release AliSQL-8.0.44-1.

Part of Alibaba's broader data stack alongside [[zvec]] (in-process vector search) and [[OpenSandbox]] (AI application sandboxing). The DDL optimization, RTO optimization, and replication boost features are planned -- suggesting an aggressive roadmap.

## Critical Analysis

The "just add DuckDB and vectors to MySQL" approach is pragmatic and clever. Most organizations already run MySQL; giving them analytical and vector capabilities without migration is a much easier sell than any greenfield database. The 200x claim is plausible for columnar vs. row-oriented analytical queries but will vary wildly by workload. The planned features (instant DDL, parallel B+tree, binlog improvements) target real enterprise pain points. The risk is fork divergence -- staying compatible with upstream MySQL while adding significant features is a maintenance burden that has defeated other MySQL forks. Alibaba's scale and engineering resources make this more credible than most fork efforts.

---
*Sources: [[summary/alisql]]*
*Last updated: 2026-05-14*
