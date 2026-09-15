---
url: https://github.com/supabase/supabase
title: "Supabase — The Postgres Development Platform"
author: Supabase, Inc.
site: GitHub
date_fetched: 2026-09-15
topics:
  - databases-and-data
  - security-and-sandboxing
---

# Supabase — The Postgres Development Platform

Supabase is "the Postgres development platform": a Firebase-like developer experience assembled from enterprise-grade open source tools rather than a 1-to-1 Firebase clone. Hosted Postgres plus auth (GoTrue, JWT), auto-generated REST (PostgREST) and GraphQL (pg_graphql) APIs, realtime subscriptions (an Elixir server that polls Postgres replication, converts changes to JSON, and broadcasts over websockets), S3 file storage with Postgres handling permissions, Postgres and Edge Functions, and a vector/embeddings toolkit. The stated rule: if an MIT/Apache-2 tool exists, use and support it; if not, build and open source it.

The `supabase/supabase` repository itself is not those services — PostgREST, GoTrue, Realtime, Storage, and pg_graphql live in satellite repos. This monorepo is the platform's front end and glue: `apps/studio` (the dashboard, ~587k lines of TS/TSX across ~4,000 files), `apps/docs` (Next.js MDX docs), `apps/www` (marketing), an Astro knowledge base, shared packages (`ui`, `ui-patterns`, `pg-meta` SQL builders, generated Management API types), a `docker/` self-hosting stack (Kong API gateway, PostgREST, GoTrue, Realtime, Storage, imgproxy, pg-meta, Edge runtime, Postgres, Supavisor pooler, with Envoy/nginx/Caddy/pgBouncer/S3 overlays), and an `examples/` tree. A `supabase/` directory dogfoods the platform: config, migrations, seed, and Deno edge functions including a docs search-embeddings pipeline.

Two things make the repo unusually interesting for this wiki. First, Studio embeds a production AI assistant built on the Vercel AI SDK: Bedrock/OpenAI model routing, a 10-step tool loop, remote MCP tools (read-only) plus local `execute_sql`/`deploy_edge_function`/notebook tools, human-approval gates on anything that runs SQL, tool-output sanitization tied to organization data-sharing consent, and a Braintrust eval harness whose expectations include required/forbidden tool sequences and a safety scorer. Second, the repo is a large-scale agent-harness specimen: a dense `AGENTS.md` (with `CLAUDE.md` being a one-line `@AGENTS.md` import), 23 skills under `.agents/skills/` covering a full docs-authoring pipeline (Frame → Draft → Edit → Review → sandbox Test), and a type-level SQL provenance system (`SafeSqlFragment`/`UntrustedSqlFragment` branded types) that treats LLM output as an untrusted taint source and only lets an explicit user gesture promote SQL to executable.

*Sources: [[raw/supabase]]*
