---
title: "AliSQL"
url: https://github.com/alibaba/AliSQL
date_fetched: 2026-05-14
section: "Databases and Data"
topics:
  - databases-and-data
---

# AliSQL: Alibaba's Enterprise MySQL Branch

## Main Purpose
Open-source MySQL fork developed by Alibaba Group that extends MySQL with DuckDB's analytical engine and native vector search.

## Key Features

| Feature | Description | Status |
|---------|-------------|--------|
| DuckDB Storage Engine | Columnar OLAP engine with automatic compression | Available |
| Vector Index (VIDX) | Native vector storage with HNSW, up to 16,383 dimensions | Available |
| DDL Optimization | Instant DDL, parallel B+tree construction, non-blocking locks | Planned |
| RTO Optimization | Accelerated crash recovery | Planned |
| Replication Boost | Binlog Parallel Flush, Binlog in Redo, large transaction optimization | Planned |

## Performance
DuckDB engine delivers approximately 200x speedup on analytical queries compared to InnoDB. Suitable for AI/ML workloads with native vector search supporting COSINE and EUCLIDEAN distance metrics.

## Compatibility
"Use your existing MySQL tools, drivers, and SQL -- zero learning curve." 100% MySQL compatibility.

## Production
Battle-tested across millions of databases in Alibaba's production environment.

## Technical
- License: GPL-2.0
- C++ (60.1%), HTML (29.8%), C (5.0%)
- Current Release: AliSQL-8.0.44-1 (January 2026)
- 5.8k stars, 894 forks
