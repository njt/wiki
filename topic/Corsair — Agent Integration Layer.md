# Corsair — Agent Integration Layer

Corsair is an open-source TypeScript library that acts as the integration-and-permission layer between an AI agent and ~120 SaaS APIs. Register typed plugins (GitHub, Slack, Gmail, Linear, …) and you get a compile-time-checked client — `corsair.slack.api.messages.post(...)` — where every call is gated by a permission matrix backed by a database the agent cannot reach, credentials are envelope-encrypted so the model never sees them, and destructive actions become expiring human-review links rather than "please don't" instructions in the prompt. It is the library-shaped answer to the credential-layer problem this wiki's [[Security and Sandboxing]] page argues is the real frontier.

---

## Architecture

A pnpm-workspace + turborepo monorepo of ~128 packages. `packages/corsair` is the core (~24K lines); `packages/<provider>` holds one package per integration (~122 with an `endpoints/` tree); `www/` is the Next.js marketing/dashboard site; `explorer/` is a CLI plugin catalog; `docs/` carries the guides.

`createCorsair({ plugins, database, kek, multiTenancy, permissions, manual, hub })` (in `packages/corsair/core/index.ts`) returns either a single-tenant client or a multi-tenant wrapper exposing `withTenant(tenantId)`, plus `keys` (integration secrets) and `permissions`/`manage` namespaces.

The plugin contract (`core/plugins/index.ts`) is the whole design in one type: each plugin declares `endpoints` (a nested function tree), `webhooks`, a Zod `schema` (a synced entity store), `keyBuilder` (auth resolution), `endpointMeta` (per-endpoint risk level), `endpointSchemas`/`webhookSchemas` (Zod), `hooks`/`webhookHooks` (before/after), `oauthConfig`, `errorHandlers`, and webhook tenant-matchers.

The client surface is built at runtime by `bindEndpointsRecursively` (`core/endpoints/bind.ts`) — which wraps every function in the readonly guard → permission guard → retry → hooks → keyBuilder pipeline — but typed statically via `UnionToIntersection` over plugin namespaces (`core/client/index.ts`). So IntelliSense knows the full method tree even though the object is assembled by reflection.

Data model (`db/index.ts`): five tables — `corsair_integrations`, `corsair_accounts`, `corsair_entities`, `corsair_events`, `corsair_permissions` — over Kysely (Postgres in prod, SQLite in demo/tests). A hosted "Hub" (`packages/corsair/hub/`) adds OAuth connect + approval UI: dev delivers through the browser (`?d=<token>`), prod through an HMAC-signed envelope POSTed to a registered delivery URL.

## Key techniques

- **DB-gated approval, not prompt consent.** Four modes (open/cautious/strict/readonly) × three risk levels (read/write/destructive) resolve through a `PERMISSION_MATRIX` to allow/deny/require-approval. `require_approval` inserts a `corsair_permissions` row (status: pending→approved→executing→completed, plus denied/expired/failed) carrying the exact JSON args and a 32-byte token, then returns a review URL. The `permissions` namespace deliberately exposes no "set approved" method — approval can only happen out-of-band, so the agent can't escalate itself. Sync mode polls the DB every 500 ms; async returns a blocked result immediately.
- **`runReadonly` via `AsyncLocalStorage`** (`core/permissions/index.ts`). An ambient flag that forces every endpoint to read-only regardless of the developer's config — used to execute agent-authored scripts under a hard read-only guarantee, independent of permission modes.
- **Envelope encryption** (`core/auth/key-manager.ts` + `encryption.ts`). A user-held KEK encrypts per-tenant/per-account DEKs, which encrypt the secret blob. Key managers auto-generate `get_<field>`/`set_<field>` accessors, and `issue_new_dek()` re-encrypts in place. Config writes are serialized through a promise chain — a fix for a real lost-update bug where Outlook's refreshed token was dropped by parallel `Promise.all` writes.
- **Zod as single source of truth** (`core/inspect/index.ts`). A hand-rolled walker reads Zod's internal `_def` to render TypeScript-like type strings and generate MCP tool schemas and docs — the same schema drives runtime validation and the agent-facing tool description.
- **Plugin codegen.** The ~122 integrations are produced and validated by scripts (`generate-plugin.ts`, `generate-plugin-from-json.ts`, `validate-plugins.ts`, `migrate-plugins.ts`), not hand-written.
- **Compile-time permission overrides.** `EndpointPathsOf<T>` makes `'repositories.delete'` a checked literal — a typo in a permission override is a type error, not a runtime footgun.

## Design decisions

- **Library, not service.** Corsair embeds in your process instead of intercepting at the network like [[Clawpatrol]] or as an HTTP gateway like [[OneCLI]]. That buys trivial self-hosting and maximum DX, but the "agent can't go around it" guarantee is conditional: it only holds if the agent process can't reach the database. That's an environment assumption, not a library invariant — worth stating plainly.
- **Type magic over runtime safety.** The generic machinery (`ExtractAuthType`, `UnionToIntersection`, discriminated `KeyBuilderContext`) is brilliant and heavy, layered over a fundamentally dynamic object tree. Classic TypeScript "types as a second, parallel program."
- **Async approval by default.** Blocked calls return a link rather than stalling the agent, which keeps long agent loops moving — but it offloads "did a human actually look?" onto a 10-minute-expiring link, inheriting the consent-theater risk if the human rubber-stamps.

## Comparison notes

- **vs [[Clawpatrol]] / [[OneCLI]]:** same "agent never sees the credential" goal, different layer — network interception, HTTP gateway, and embedded library respectively. Corsair is the only one that gates *per-argument* (the exact JSON args are stored in the permission record) rather than per-protocol-fact or per-route.
- **vs [[How We Contain Claude]]:** that post's 93% prompt-approval rate is the empirical case for Corsair's model — approvals are single-use, expiring, and out-of-band, so "always click allow" isn't available. But Corsair still depends on a human clicking a link, so it doesn't escape the human-is-the-weakest-link problem.
- **vs [[AI Agents In-Depth — Function Calling, MCP and Tool Use Under the Hood]]:** the "LLM selects, harness calls" boundary is exactly where Corsair sits — its permission guard runs inside the harness's tool-call path, and "the agent never sees credentials" answers Smith's credit-card-leakage demo deterministically rather than relying on model refusal.
- **vs [[AI-Ready APIs — Postman AWS Competency]]:** Corsair is "agent-ready APIs" as a library — every endpoint carries risk metadata, a Zod schema, and a description, so the MCP tool list is self-describing. Postman fixes the spec; Corsair wraps the API and adds governance at call time.
- **vs [[Enterprise-Managed MCP Authorization]]:** that's centralized IdP-scoped connector auth; Corsair is per-endpoint, per-tenant policy with multi-tenancy (`withTenant`) and isolated credentials/data per tenant.

---

*Sources: [[raw/corsair]], [[summary/corsair]]*
*Last updated: 2026-08-14*
