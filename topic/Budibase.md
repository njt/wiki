# Budibase

Budibase is an open-source low-code platform for building internal tools and business apps. You drag components onto screens in a Svelte-based visual builder, connect to databases (Postgres, MySQL, Mongo, REST, S3, Google Sheets, and more), add automations with a visual workflow engine, and publish. The runtime renders your app as a recursive Svelte component tree driven by JSON definitions — no compilation step between design and deploy. ~350K lines across 14 packages in a monorepo. GPL-3.0.

## Architecture

The monorepo has four runtime processes, each a separate package:

**[[Server]]** (`packages/server/`, 177K LoC) — The main API server. Koa on Node.js with CouchDB as the primary store and Redis/Bull for queues. Handles all per-app operations: tables, rows, views, screens, automations, datasources, queries, permissions, deployment. Structured as thin controllers → fat SDK layer → CouchDB. Routes self-register via import side-effects into endpoint groups with middleware-based authorization.

**[[Builder]]** (`packages/builder/`, 108K LoC) — Svelte + Routify visual editor. Embeds the client runtime as a preview iframe (`window["##BUDIBASE_IN_BUILDER##"]` flag). The binding engine (`dataBinding.ts`, 2106 lines) discovers bindable properties from the component tree (data providers, current user, URL params, device, state, roles) and converts between readable bindings (`Current User.email`) and runtime-safe format (`[user].[email]`).

**[[Client]]** (`packages/client/`, 20K LoC) — Runtime renderer. `Component.svelte` is a single recursive component that renders any component type: resolves the Svelte constructor from `componentStore`, splits settings into static vs dynamic (bound), enriches bindings via Handlebars+JS sandbox, evaluates conditional UI, and renders with `<svelte:component this={constructor}>` + `<svelte:self>` for children. Reactivity is surgical — only components whose binding strings reference a changed context key re-render.

**[[Worker]]** (`packages/worker/`, 16K LoC) — Separate Koa process (port 4002) for global/SSO operations: authentication (login/logout, SAML, OIDC, Google), user management, invitations, email (Nodemailer + Handlebars templates), tenant lifecycle, licensing, SCIM.

### Shared packages

- **types** — All TypeScript type definitions: documents (table, row, screen, automation, etc.), UI components, bindings
- **backend-core** — Shared backend services: auth (Passport.js, JWT, CSRF, sessions), context (AsyncLocalStorage), tenancy, Redis (ioredis wrapper with 12 logical databases), caching, feature flags (PostHog), permissions (5-level hierarchy)
- **shared-core** — Isomorphic code: search filters (client+server query engine), constants, automation definitions, helpers
- **string-templates** — Handlebars + JS sandbox template engine with three-stage pipeline (preprocess → execute → postprocess). JS bindings use `{{ js "base64" }}` syntax, execute in `isolated-vm` on backend or `@budibase/vm-browserify` on frontend
- **bbui** — Budibase UI component library (Svelte)

### Data model

Every entity is a CouchDB document with a typed ID prefix: `ta_` (tables), `ro_` (rows), `au_` (automations), `screen_` (screens), `view_` (views), `query_` (queries), `role_` (roles), `datasource_` (external datasources), `li_` (relationship links). Each app gets its own CouchDB database. The `context.doInWorkspaceContext(workspaceId, fn)` pattern sets the active database for all operations. Apps exist in `dev` and `published` modes with different DB prefixes.

External datasources (Postgres, MySQL, Mongo, DynamoDB, S3, REST, Snowflake, etc.) are adapters implementing `Integration` (schema introspection) and `IntegrationBase` (runtime execution). Queries against these run in forked Node.js processes via `worker-farm` to prevent event-loop blocking.

## Key Techniques

**Recursive single-component renderer**: Rather than per-component-type renderers, `Component.svelte` handles all component types identically. The component tree is JSON; the renderer resolves constructors dynamically and recurses. This means adding a new component type requires only a Svelte component and a manifest entry — no renderer changes.

**Document ID taxonomy for range queries**: The `{type}{SEPARATOR}{id}` scheme (`ta_abc123`, `ro_ta_abc123_xyz`) means CouchDB queries like `_all_docs?startkey="ro_"&endkey="ro_￰"` efficiently return all rows without secondary indexes. Parent-child relationships are encoded in the ID itself (a row ID embeds its table ID).

**Surgical reactivity**: When context data changes, the context store broadcasts the changed key. Each `Component.svelte` instance checks whether its binding strings contain that key via `handleContextChange(key)` and only re-enriches if they match. Most low-code platforms re-render entire subtrees on any data change.

**Three-stage template pipeline**: Preprocessing converts bracket notation to dots and wraps expressions in helpers → Handlebars compiles and executes → Postprocessing converts `%LITERAL% type-value` markers back to typed values. This hack lets Handlebars (string-based) pass typed values through without loss.

**Thread-per-query isolation**: External datasource queries and automation executions fork to separate Node.js processes via `worker-farm`. A slow Postgres query can't block the main server event loop. The trade-off is IPC overhead per query.

**Self-registering routes**: Route files call `endpointGroupList.group().get/post()` as import side-effects. The route registry is the set of files imported by `routes/index.ts` — implicit but grep-friendly. Each group has pre-bound authorization middleware so no route can be accidentally public.

**Context-as-AsyncLocalStorage**: Request-scoped context (tenant, workspace, user identity) propagates through Node.js `AsyncLocalStorage`, not parameter threading. This means any function deep in the call stack can access `getTenantId()` or `getWorkspaceDB()` without explicit parameter passing.

## Design Decisions

**CouchDB over Postgres**: Sacrificed ACID transactions and the SQL ecosystem for JSON-native document storage, built-in replication, and the ability to store arbitrary component trees without schema migration. The document ID prefix system partially compensates for the lack of tables/collections. This is Budibase's most unusual architectural choice — most competitors (Retool, Appsmith, Tooljet, Xano) use Postgres/MySQL.

**JSON component tree over compiled code**: Rather than compiling the visual design into React/Svelte code, Budibase stores the component tree as JSON and interprets it at runtime. This eliminates a compilation step (faster preview, easier undo/redo) but adds interpretation overhead. The trade-off makes sense for internal tools where development speed matters more than rendering performance.

**Separate worker process**: Auth and user management run in a separate Koa process from the app server. Clean separation of concerns, but two Node.js processes to deploy. The server handles per-app operations; the worker handles global/tenant-level operations.

**Handlebars + JS sandbox over custom expression language**: Extended Handlebars with a `{{ js "..." }}` syntax that runs arbitrary JavaScript in `isolated-vm`. More powerful than a custom DSL, but the sandbox is critical — user-written automation scripts run here. The `disabledHelpers` option lets admins restrict dangerous operations.

**Bull/Redis for async work**: Automations, workspace migrations, RAG processing, knowledge sync, and agent tests all use Bull queues. Redis doubles as session store (with per-user caps and TTL) and caching layer (12 logical databases via key prefixes). This is a Redis-heavy architecture — if Redis goes down, sessions, queues, and caches all fail.

## Comparison Notes

**[[Xano]]** is a no-code backend platform (Postgres-based, auto-generated APIs). Budibase is full-stack: it includes the frontend builder and client renderer. Xano focuses on making the backend invisible; Budibase focuses on making the whole app buildable by non-developers.

**[[n8n]]** is a standalone workflow automation tool with 400+ integrations. Budibase's automation engine is embedded in the app-building context — triggers fire from row changes, automations have access to app data and screens. n8n is better for pure workflow automation; Budibase is better when the workflow is part of an app.

**[[DAB]]** (Microsoft's Data API Builder) auto-generates REST/GraphQL/MCP endpoints from databases. Budibase does this too (REST API from connected datasources) but adds the UI layer and automation on top. DAB is an API tool; Budibase is an app platform.

**[[InsForge]]** is an open-source BaaS exposing Postgres, auth, and storage as MCP tools for coding agents. Budibase targets human builders; InsForge targets AI agents. Both are "make data into apps" but for different builders.

**[[DSL-Driven Kanban Boards (Goja-Site)]]** composes apps from chainable JavaScript DSLs. Budibase composes them from drag-and-drop component trees. Both are declarative composition, but Budibase's JSON tree is more approachable for non-developers.

**[[BESSER]]** is an academically-led, model-driven low-code platform that generates full-stack apps across 15+ technology stacks from a single model. Where Budibase targets internal tools with a visual builder + runtime interpreter, BESSER targets general application development with model→code generation. Budibase generates a running app from JSON trees; BESSER generates source code you can modify and maintain independently. The trade-off is flexibility vs. ownership — Budibase apps live inside Budibase, while BESSER-generated code lives wherever you put it.

## Tags

#tool #project #low-code #database #internal-tools #open-source #svelte #couchdb #workflow-automation

## Cross-links

- [[Xano]] — No-code backend, Budibase's counterpart for backend-only use cases
- [[n8n]] — Visual workflow automation, similar automation engine
- [[DAB]] — Auto-generated APIs from databases, similar data-access layer
- [[InsForge]] — BaaS for agents, similar "make data useful" spirit
- [[Databases and Data]] — Hub for data storage and query patterns
- [[Event-Driven vs Polling Architectures]] — Webhook and trigger patterns in Budibase's automation engine
- [[DSL-Driven Kanban Boards (Goja-Site)]] — Another declarative UI composition approach
- [[Phoenix LiveView]] — Server-rendered reactive UI; Budibase is client-rendered but shares real-time WebSocket updates
- [[Apache Burr]] — State machine framework; contrasts with Budibase's event-driven automation model

---
*Source: https://github.com/budibase/budibase — Deep repo analysis from cloned repository.*
*Fetched: 2026-06-21*
