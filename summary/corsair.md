---
url: https://github.com/corsairdev/corsair
title: Corsair: The Unified Integration Layer for Agents
author: corsairdev
date_fetched: 2026-08-14
date_published: unknown
topics:
  - security-and-sandboxing
---

# Corsair: The Unified Integration Layer for Agents

Corsair is an Apache-2.0, YC-backed TypeScript library (and ~128-package pnpm monorepo) that sits between an AI agent and the SaaS APIs it uses. You register typed plugins — GitHub, Slack, Gmail, Linear, and roughly 120 others — and get back a compile-time-checked client (`corsair.slack.api.messages.post(...)`) where every call passes through a permission gate, credentials are envelope-encrypted so the model never sees them, and destructive actions surface as human-review links instead of in-context prompts.

The core package (`packages/corsair`, ~24K lines) is a factory, `createCorsair({ plugins, database, kek, multiTenancy, permissions, manual, hub })`, that returns either a single-tenant client or a multi-tenant wrapper with `withTenant(tenantId)`. Each plugin declares an `endpoints` tree, Zod schemas, webhook handlers, a `keyBuilder` for auth resolution, and per-endpoint `riskLevel` metadata (read/write/destructive).

Three mechanisms do the heavy lifting. The **permission matrix** maps four modes (open/cautious/strict/readonly) × three risk levels to allow/deny/require-approval; `require_approval` writes a `corsair_permissions` row with a 32-byte token and returns an expiring review URL, and the permissions API deliberately has no "set approved" method — approval can only happen out-of-band. **Envelope encryption** uses a user-held KEK to encrypt per-tenant/per-account data keys that encrypt the actual secrets. **Type-level API generation** derives the entire client surface (and even compile-time-checked permission-override paths) from the plugin definitions via TypeScript conditional types.

An optional hosted "Hub" (code in `packages/corsair/hub/`) provides OAuth connect flows and the approval UI: dev delivers via browser (`?d=<token>`), prod via an HMAC-signed envelope POSTed to a registered delivery URL. The whole thing ships MCP adapters so the typed tool surface plugs directly into Claude Code, Cursor, and other agents.

Key design choice: Corsair is a **library, not a service**. It embeds in your process rather than intercepting at the network (like Clawpatrol) or as a gateway (like OneCLI) — trivially self-hostable and excellent for developer ergonomics, but its "the agent can't go around it" guarantee only holds as long as the agent process can't reach the database.
