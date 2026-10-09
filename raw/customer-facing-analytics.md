---
url: https://motherduck.com/docs/getting-started/customer-facing-analytics/
date_fetched: 2026-10-09
---

# Customer-Facing Analytics Overview

Customer-facing analytics (CFA), or embedded analytics, has requirements that traditional data architectures rarely meet. If you're building SaaS analytics dashboards, white-label reporting, or embedded data visualizations, CFA demands sub-second response times, per-customer isolation, and integration with operational applications — all while serving many concurrent end users.

MotherDuck, the serverless cloud data warehouse built on DuckDB, addresses these needs through two architectural capabilities:

- 
**Hypertenancy**: Each customer gets their own dedicated DuckDB instance (Duckling), providing full compute isolation (so no resource contention between users), predictable performance, and the ability to scale resources independently based on individual customer needs.
- 
**Dual Execution**: Enabled by DuckDB's lightweight architecture, queries can run both in the cloud and directly in the client's browser through WebAssembly, delivering near-instantaneous data exploration and filtering.

This guide explains how MotherDuck's architecture addresses the core CFA challenges and provides implementation patterns you can ship.

## What is customer-facing analytics?

**Customer-Facing Analytics (CFA)** embeds analytics directly into operational applications for external users—customers, partners, or end-users—rather than internal stakeholders. Traditional BI targets internal teams, runs on batch-processed data models, serves a small number of users, and tolerates higher-latency queries.

| Dimension | Traditional BI | Customer-Facing Analytics | 
|---|---|---|
| Audience | Internal (analysts, executives) | External (customers, partners) | 
| Delivery | BI tools (Tableau, Looker) | Embedded in application | 
| Latency | Seconds to minutes acceptable | Milliseconds to low seconds required | 
| Scale | Dozens to hundreds of users | Thousands to millions of users | 
| Isolation | Shared warehouse | Per-customer isolation needed | 
| Tech Stack | Python, BI tools | JavaScript, embedded SDKs | 

"Customer-facing analytics" and "embedded analytics" get used interchangeably. Both describe integrating analytical capabilities directly into a product instead of sending users to a separate BI tool. The difference is one of emphasis: customer-facing analytics focuses on the *audience* (your customers), while embedded analytics focuses on the *delivery* (built into your app). MotherDuck supports both.

### Common use cases

- **SaaS analytics dashboards:**give customers self-serve analytics within your product, covering usage metrics, performance KPIs, and ROI reporting
- **White-label analytics:**offer analytics under your brand that customers can explore without leaving your app
- **Embedded dashboards:**drop interactive charts and tables directly into your application UI
- **Multi-tenant reporting:**serve thousands of customers from one platform while keeping each tenant's data and compute isolated

**What about AI-driven analytics?**
AI-driven analytics enables natural language interactions with data, allowing users to ask conversational questions like "What were our top-selling products last quarter?" and get immediate answers. MotherDuck's hypertenancy and Dual Execution make it well-suited for building AI-driven analytics solutions. The MotherDuck MCP Server includes a Dive Viewer that renders Dives inline in AI chat clients that support MCP Apps. Learn how to build analytics agents with MotherDuck.

## The CFA challenge

Building customer-facing analytics systems presents three core challenges:

### Challenge 1: Technology stack mismatch

For many applications, the data sits in a transactional database (OLTP database) like Postgres or MySQL. Engineers building CFA features often run analytical queries directly in a multi-tenant transactional database, which works until it fails at scale. Row-based storage and transactional databases are not designed for efficient analytical querying.

Operational applications often live in JavaScript/TypeScript, but traditional data tools are Python-centric. Operational teams work with OLTP databases built for transactions, while data teams use OLAP systems tuned for analytics but with their own challenges. Analytical workloads spike with user activity, while transactional loads need steady compute.

### Challenge 2: Latency requirements

Users expect sub-second response times—typical for OLTP systems. Anything slower degrades the application experience. Distributed OLAP systems (BigQuery, Snowflake, Databricks) often have cold starts and coordination overhead that keep them above those targets, even for small datasets.

Teams often add caching layers or refresh pipelines between OLTP and OLAP. That adds complexity, introduces another failure point, and delays data freshness.

### Challenge 3: Multi-tenancy at scale

Switching to an analytics engine is the first step. Many legacy OLAP engines were designed for internal analytics and are provisioned as a single instance or cluster for all customer data, leading to downstream complexities:

- **Overprovisioning**: Resources sized for peak load sit idle most of the time
- **Noisy neighbors**: Large customer impacts small customers
- **Resource contention**: Concurrency limits affect everyone
- **Unpredictable performance**: Query times vary based on load
- **Security concerns**: All customer data in one shared system

## Why MotherDuck for customer-facing analytics?

MotherDuck's architecture aligns with the requirements of Customer-Facing Analytics. Two architectural advantages set it apart:

### 1. Hypertenancy

MotherDuck provisions a Duckling (DuckDB instance) for each customer (or even for each customer's users). This hypertenancy model isolates customer data and delivers consistent DuckDB performance to each user.

**Why single-node beats distributed compute clusters for CFA**

Traditional data warehouses use distributed computing with coordination overhead, data shuffling, and network latency. Even a fast query typically takes a second or more because of this overhead.

DuckDB and MotherDuck use single-node, optimized columnar execution:

- Zero network hops
- Zero coordination overhead
- Optimized vectorized execution

For CFA workloads that query one customer's data at a time, single-node execution is usually faster than distributed, and MotherDuck can reach **subsecond performance**.

#### Scaling analytics up and out

Each customer (and possibly each of their users) has their **own MotherDuck Duckling** (DuckDB instance). One account could run hundreds or thousands of Ducklings at a time, or none. This serverless model underpins MotherDuck's advantage versus other engines.

MotherDuck's **cold start time is sub ~100ms**, and **per-second billing** (1-second minimum) keeps individual queries cost-efficient.

While MotherDuck supports provisioning one Duckling per user, start simpler. Begin with a single Duckling per customer and introduce per-user isolation or read scaling when monitoring shows sustained high concurrency or when you need tighter performance guarantees.

This isolated Duckling approach with vertical scaling delivers:

- **Perfect isolation**: No noisy neighbors
- **Predictable performance**: Dedicated resources per customer
- **Cost-effective**: Pay only for what each customer needs
- **Easy scaling**: Vertically scale individual ducklings as needed

Scale vertically by upgrading (or downgrading) the Duckling size your application uses for each customer, giving more power to higher-priority customers. If you need more compute or higher concurrency, launch read scaling Ducklings for compute-hungry customers.

MotherDuck offers several Duckling sizes for larger workloads.

For programmatic changes to user settings, refer to our API docs.

### White-label analytics

Many SaaS companies need analytics that look and feel native to their product. MotherDuck's architecture supports white-label analytics by design:

- **Per-customer isolation:**each tenant gets a dedicated Duckling, with no shared infrastructure leaking through
- **Flexible query layer:**use any frontend charting library (Recharts, D3, Observable Plot) with MotherDuck as the SQL backend
- **No vendor branding:**unlike embedded BI tools that surface their own UI, MotherDuck powers your queries behind the scenes
- **DuckDB-Wasm for client-side execution:**ship analytics that run entirely in the browser for maximum responsiveness

### 2. Dual Execution for zero-latency exploration

As you build Customer-Facing Analytics into your product, you need sub-second response times so customers can explore their data quickly. Distributed data warehouses rarely meet that bar.

Because MotherDuck is built on DuckDB, you can connect from any DuckDB client. DuckDB is an in-process database, so it **can run on your server (3-tier) or directly in the client's browser through WebAssembly (1.5-tier)**.

This enables "Dual Execution": combining local data and compute with cloud data and compute in a single query, giving you flexibility to optimize for performance and cost.

**Traditional approach has multiple network hops:**

**DuckDB-Wasm enables client-side execution:**

Because the same DuckDB SQL engine runs on both MotherDuck Ducklings and on your customers' machines, you can offload data processing to their laptops and provide fast data exploration, filtering, and sorting using SQL. Customers do not need to install anything because DuckDB runs inside the web browser using WebAssembly (Wasm).

You can see this experience in Column Explorer and Instant SQL in the MotherDuck UI. Here's a teaser of it in action:

## Implementation patterns

MotherDuck enables three architectural patterns for customer-facing analytics:

### Embedded Dives

**Best for:** Shipping customer-facing dashboards without building a frontend. Create a Dive with a natural language prompt or as code, mint an embed session from your backend, and drop a sandboxed iframe into your app. The Dive brings its own charts, filters, and queries, so there's no frontend data plumbing to build. Embedding Dives requires the Business plan.

**Architecture:**

**Key Benefits:**

- No charting code to write or maintain: iterate on the dashboard with natural language instead of frontend releases
- Per-tenant data: override `required_resources`per session to point the same Dive at each tenant's database
- Deep links: seed filters and selections per session with `initial_state`
- Production stability: pin a specific Dive version per embed session
- Host-page integration: data exports and viewer state sync through `postMessage`
- Fresh data: embedded Dives run on read scaling connections that sync about once a minute, and `REFRESH DATABASE`pulls the latest writes on demand

**When to use:**

- You want dashboards in your product without building and maintaining a charting frontend
- One dashboard definition serves many tenants, each against their own data
- Interactions can live inside the Dive, with `postMessage`covering host-page integration
- Your organization is on the Business plan

Follow the embedding Dives guide for the full setup, from creating the embed session to handling exports and state updates.

### 3-tier architecture

**Best for:** Applications requiring server-side authorization, business logic, or deployments to stateful platforms.

**Typical web application architecture:**

**Key Benefits:**

- Persistent database connection (connection pooling saves ~200ms per request)
- Fast query performance (~50-100ms)
- Server-side security and authorization
- Works with any DuckDB client (Node.js, Python, Go, Rust, Java)

**Performance optimizations:**

- Intermediate table results: Pre-aggregate data on MotherDuck for faster queries
- Prefer one well-structured SQL statement that returns all needed metrics (using SELECT with multiple aggregates, CASE/FILTER, or UNION ALL).
- For multi-step workflows, wrap statements in a BEGIN … COMMIT transaction to ensure atomicity.
- For data movement, use bulk operations (COPY, INSERT … SELECT) instead of many row-by-row calls. MotherDuck is analytical, not transactional: if queries feel slow, set the right expectations and reshape OLTP-style write patterns into batches.
- Application Caching: Cache rarely-changing data on your server to avoid any extra queries on MotherDuck

**When to use:**

- You need server-side authorization and business logic
- You want a traditional, battle-tested architecture
- You're deploying to stateful services (Cloud Run, ECS, Kubernetes)
- Your team works with multiple languages

### Ready to build?

Follow the hands-on Builder's Guide to set up a 3-tier CFA application step by step.

### 1.5-tier architecture (DuckDB-Wasm)

**Best for:** Read-heavy dashboards with `<1GB` data per user where you need maximum performance. This works well for embedded dashboards with interactive charts, tables, and filters that respond in under 10ms because queries execute locally in the user's browser.

**Architecture:**

**Key Benefits:**

- Sub-10ms query latency (queries run locally in browser)
- Near-zero server costs (just data transfer)
- Offline support after initial data load
- Infinite scalability (users provide compute)

**Performance optimizations:**

- **Optimize Initial Load**: Use Parquet compression, limit to- `<50MB`
- **IndexedDB Persistence**: Data survives page reloads
- **Incremental Sync**: Only fetch new data since last sync

**When to use:**

- Read-heavy dashboards with frequent filtering/drilling
- Want `<10ms`query latency
- Data per user is `<1GB`
- Want to minimize server costs

#### Hands-on example

See our 1.5-tier architecture example demonstrating best practices for building a 1.5-tier analytics application using TypeScript, React and the MotherDuck Wasm SDK.

### Comparing the patterns

| Factor | Embedded Dives | 3-Tier | 1.5-Tier (DuckDB-Wasm) | 
|---|---|---|---|
| Frontend to build | None (iframe embed) | Custom UI | Custom UI | 
| Query latency | Sub-second (cloud) | ~50-100ms | ~5-20ms ⚡ | 
| Server cost | $ (session minting only) | $$ (per request) | $ (data transfer only) | 
| Scalability | High (read scaling) | High (auto-scaling) | ♾️ Unlimited | 
| Data per user | Any size | Any size | `<1GB`optimal | 
| Offline support | ❌ No | ❌ No | ✅ Yes | 
| Server-side logic | ❌ Limited (per session) | ✅ Yes | ❌ Limited | 
| Plan availability | Business plan | All plans | All plans | 
| Best for | Dashboards, fast shipping | Complex logic, auth | Read-heavy dashboards | 

### Try MotherDuck for free

Sign up and start building customer-facing analytics in minutes. No credit card required.

### Additional resources

- Embedding Dives in your web application
- Building Analytics Agents with MotherDuck
- Read Scaling Ducklings
- Duckling Sizes

## FAQ

### What is embedded analytics?

Embedded analytics means putting data visualizations, dashboards, and interactive reports directly inside a software application. Users explore data in the product they already use rather than switching to a separate BI tool. MotherDuck powers embedded analytics with sub-second SQL queries and per-user compute isolation.

### What is the difference between embedded analytics and traditional BI?

Traditional BI is built for internal teams using standalone tools like Tableau or Looker. Embedded analytics is for your external customers, living inside your product. That difference creates harder technical requirements: you need lower latency, higher concurrency (potentially thousands of simultaneous users), and per-tenant data isolation. MotherDuck's Duckling architecture handles all three.

### What is white-label analytics?

White-label analytics lets you offer data analytics under your own brand. Your customers see dashboards that match your product's look and feel, with no third-party logos visible. MotherDuck supports this by providing a SQL query engine (DuckDB) that runs behind your UI — there's no user-facing vendor footprint.

### How do you add analytics to a SaaS product?

The fastest path is embedding a Dive: an interactive dashboard served through an iframe, with no frontend to build. If you're building your own frontend, there are two main approaches. In a 3-tier architecture, your server queries MotherDuck and returns results to the frontend. This works well when you have complex auth or business logic. In a 1.5-tier architecture, DuckDB runs directly in the browser through WebAssembly, which is a better fit for read-heavy dashboards where each user's data stays under 1GB. Both approaches give you fast query performance.

### What is multi-tenant analytics?

Multi-tenant analytics means serving multiple customers from one shared platform while keeping each customer's data separate. MotherDuck works differently, through Hypertenancy — every tenant gets a dedicated DuckDB instance (a Duckling). This avoids noisy-neighbor problems and keeps performance predictable while maintaining data isolation between each customer.

### Can I embed dashboards without building a frontend?

Yes. Create a Dive with a natural language prompt or as code, then embed it: your backend mints an embed session through the MotherDuck API and your frontend loads the Dive in a sandboxed iframe. The Dive handles the charts, filters, and queries, and you can point the same Dive at each tenant's database per session. Embedding Dives requires the Business plan. See embedding Dives in your web application for the full setup.
