---
url: https://github.com/malloydata/malloyyo
title: "Malloyyo"
author: Malloy Foundation (malloydata)
date_fetched: 2026-07-18
date_published: 2025
topics:
  - agent-architecture
---

Malloyyo is a Next.js 16 monorepo (~9.5K lines of TypeScript) that turns Malloy semantic models into MCP endpoints for AI agents, with a companion web UI and CLI for publishing models. It lets an AI query a database through a curated semantic surface — the model author defines what's queryable, and the engine enforces it.

The architecture has three tiers. A vendor-neutral, zero-dependency **MCP engine** provides types, helpers, and turnkey tool surfaces (`explore` and `develop`). A **host layer** wires the engine to the real world: database connections, authentication, history recording, and client-profile-aware formatting. A **connection pooling and caching system** treats serverless cold starts as a first-class problem, with per-model-version pools, two-level ModelDef caching (in-memory LRU + Postgres bytea), and per-instance diagnostics.

The most novel contribution is **restricted query governance**: all explore-surface queries route through `loadRestrictedQuery()`, which blocks `import`, `given:` declarations, raw-SQL forms, and connection table references. The AI can only query what the model exposes — no invented columns, no wrong joins, no access outside the model. A companion security measure parks generated SQL on a `HOST_ONLY` channel when `execute:true`, so the agent never sees the SQL it ran.

Other notable design choices include: source-centric tool interfaces (matching how users think) layered over model-keyed internals; field-not-found recovery that detects sibling-field references and suggests the `extend:` fix; annotation dual-channel promotion (`#"` → description, `#(agent)` → instructions); result byte budgeting with row spill instead of cursors; and a single `query` tool with `execute:false` for validation, optimized for client tool-search ranking.

The project uses Drizzle ORM on Neon Postgres, NextAuth v5 for user auth, and a full OAuth 2.1 provider for MCP auth (PKCE S256, refresh token rotation). A standalone CLI covers publishing, linting, and local development. Unlike DAB (which auto-generates endpoints from any schema) or Nubase (which provides a full backend), Malloyyo is deliberately thin — it serves models, connects to your existing databases, and treats the semantic model as the governance boundary.
