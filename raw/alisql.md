---
url: https://github.com/alibaba/AliSQL
date_fetched: 2026-07-05
backfilled: true
---

**Alibaba's Enterprise MySQL Branch with DuckDB OLAP & Native Vector Search**

  *Battle-tested in Alibaba's production environment, powering millions of databases*

Features • Quick Start • Docs • Roadmap • Contributing

AliSQL brings enterprise-grade capabilities to MySQL, combining the reliability of InnoDB OLTP with DuckDB's blazing-fast analytics and native vector search — all through familiar MySQL interfaces.

| DuckDB columnar engine delivers  | Built-in HNSW algorithm supporting up to  | Use your existing MySQL tools, drivers, and SQL — zero learning curve | 

| Feature | Description | Status | 
|---|---|---|
| DuckDB Storage Engine | Columnar OLAP engine with automatic compression, perfect for analytics workloads | Available | 
| Vector Index (VIDX) | Native vector storage & ANN search with HNSW, supports COSINE & EUCLIDEAN distance | Available | 
| DDL Optimization | Instant DDL, parallel B+tree construction, non-blocking locks | Planned | 
| RTO Optimization | Accelerated crash recovery for faster instance startup | Planned | 
| Replication Boost | Binlog Parallel Flush, Binlog in Redo, large transaction optimization | Planned | 

```
# Clone the repository
git clone https://github.com/alibaba/AliSQL.git
cd AliSQL
# Build (release mode)
sh build.sh -t release -d ~/alisql
# Install
make install
```

Step-by-step guide:How to set up a DuckDB node

```
# Initialize data directory
~/alisql/bin/mysqld --initialize-insecure --datadir=~/alisql/data
# Start the server
~/alisql/bin/mysqld --datadir=~/alisql/data
```
```
-- Create an analytical table with DuckDB engine
CREATE TABLE sales_analytics (
    sale_date DATE,
    product_id INT,
    revenue DECIMAL(10,2),
    quantity INT
) ENGINE=DuckDB;
-- Run complex analytics (200x faster than InnoDB!)
SELECT
    DATE_FORMAT(sale_date, '%Y-%m') as month,
    SUM(revenue) as total_revenue,
    COUNT(*) as transactions
FROM sales_analytics
GROUP BY month
ORDER BY total_revenue DESC;
```
```
-- Create a table with vector column
CREATE TABLE embeddings (
    id INT PRIMARY KEY,
    content TEXT,
    embedding VECTOR(768)  -- 768-dimensional vectors
) ENGINE=InnoDB;
-- Create HNSW index for fast ANN search
CREATE VECTOR INDEX idx_embedding ON embeddings(embedding);
-- Find similar items using cosine distance
SELECT id, content,
       COSINE_DISTANCE(embedding, '[0.1, 0.2, ...]') as distance
FROM embeddings
ORDER BY distance
LIMIT 10;
```
| Option | Description | Default | 
|---|---|---|
| `-t release|debug` | Build type | `debug` | 
| `-d <dir>` | Installation directory | `/usr/local/alisql` | 
| `-g asan|tsan` | Enable sanitizer (memory/thread) | disabled | 
| `-c` | Enable code coverage (gcov) | disabled | 

**Prerequisites:** CMake 3.x+, Python 3, GCC 7+ or Clang 5+

```
Q4 2025  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
         [x] DuckDB Storage Engine  [x] Vector Index (VIDX)   [x] Open Source
2026     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
         [ ] DDL Optimization       [ ] RTO Optimization      [ ] Replication Boost
             - Instant DDL              - Fast Crash Recovery     - Binlog Parallel Flush
             - Parallel B+tree          - Minimize RTO            - Binlog in Redo
             - Non-blocking Locks                                 - Large TX Optimization
```
| Document | Description | 
|---|---|
| DuckDB Integration Guide | Complete guide for DuckDB storage engine | 
| Vector Index Guide | Native vector storage and ANN search | 
| Release Notes | What's new in AliSQL 8.0.44 | 
| Setup DuckDB Node | Quick setup guide for analytics | 

**External Resources:**

AliSQL became open source in December 2025 and is actively maintained by Alibaba Cloud Database Team.

We welcome contributions of all kinds!

- **Fork**the repository
- **Create**a feature branch (- `git checkout -b feature/amazing-feature`)
- **Commit**your changes (- `git commit -m 'Add amazing feature'`)
- **Push**to the branch (- `git push origin feature/amazing-feature`)
- **Open**a Pull Request

For bugs and feature requests, please use GitHub Issues.

RDSAI CLI is a next-generation, AI-powered CLI that transforms how you interact with AliSQL and MySQL databases. Describe your intent in **natural language**, and the AI agent handles the rest.

```
# Install
curl -LsSf https://raw.githubusercontent.com/aliyun/rdsai-cli/main/install.sh | sh
# Connect and ask in natural language
rdsai --host localhost -u root -p secret -D mydb
mysql> analyze index usage on users table
mysql> show me slow queries from the last hour
mysql> why this query is slow: SELECT * FROM users WHERE name LIKE '%john%'
```
**Key Features:**

- Natural language to SQL conversion (English/中文)
- AI-powered query optimization and diagnostics
- Execution plan analysis with `Ctrl+E`
- Multi-model LLM support (Qwen, OpenAI, DeepSeek, Anthropic, etc.)
- Performance benchmarking with automated analysis

| 
 For bug reports & feature requests | 
 Managed DuckDB analytical instances | 

AliSQL is licensed under **GPL-2.0**, the same license as MySQL.

See the LICENSE file for details.

Made with care by Alibaba Cloud Database Team
