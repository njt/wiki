---
url: https://github.com/budibase/budibase
title: Budibase — Open-Source Low-Code Platform
author: Budibase
date_fetched: 2026-06-21
date_published: 2023-06-21
topics:
  - misc
---

# Budibase — Architectural Analysis (Deep Repo Clone)

## Summary

Budibase is an open-source low-code platform for building internal tools and business apps. It combines a drag-and-drop UI builder (Svelte), a declarative component tree renderer (Svelte runtime), a CouchDB-backed data engine with external datasource adapters, an automation engine (Bull/Redis queues), and an AI subsystem (LiteLLM-based agents with RAG). The monorepo contains 14 packages: server (Koa, 177K LoC), builder (108K LoC Svelte), client runtime (20K LoC Svelte), worker (Koa SSO/global management, 16K LoC), plus shared packages for types, backend-core, frontend-core, string templates, and BBUI component library.

## Repo Structure (packages/)

```
packages/
├── server/          # Main API server (Koa, CouchDB, Bull/Redis, 176,924 lines)
├── builder/         # Low-code builder UI (Svelte/Routify, 108,075 lines)
├── client/          # Runtime app renderer (Svelte, 20,001 lines)
├── worker/          # Global SSO/user management (Koa, 16,062 lines)
├── backend-core/    # Shared backend services (auth, DB, context, Redis, cache)
├── frontend-core/   # Shared frontend components and utilities
├── shared-core/     # Isomorphic code (filters, constants, automation defs)
├── types/           # TypeScript type definitions (all document types, UI types)
├── string-templates/# Handlebars + JS sandbox template engine
├── bbui/            # Budibase UI component library
├── pro/             # Proprietary/enterprise features
├── sdk/             # External SDK
├── cli/             # CLI tooling
└── upgrade-tests/   # Upgrade/migration test suites
```

## Architecture Deep Dive

### 1. Server (packages/server/src/)

**Entry**: `src/index.ts` → `src/app.ts` → `src/koa.ts` (Koa app construction) → `startup/index.ts` (bootstrap sequence)

**Bootstrap order**: File system init → Redis init → Write-through cache → Event emitter → Feature flags → WebSockets → Plugin watcher → Version check → Queue init (events, RAG, knowledge sync, agent tests, automations, workspace migrations) → Route registration → Admin user creation → JS runner init → LiteLLM readiness → Ready state

**Route architecture**: Three-layer self-registering system:
1. `api/index.ts` — Single Koa Router with middleware pipeline: body parser, correlation IDs, logging, IP extraction, user-agent → auth (JWT/session) → tenancy → active tenant → licensing → workspace context → CSP → audit → workspace migrations
2. `api/routes/index.ts` — 40+ route files self-register via import side effects into endpoint groups (globalBuilder, builder, creator, builderAdmin, public). Pattern: `endpointGroupList.group({middleware, first}).get(...)`
3. `api/routes/public/index.ts` — Public REST API v1 at `/api/public/v1` with rate limiting, CORS, and resource-pattern middleware

**SDK/Service layer**: Controllers are thin — they delegate to `sdk/` which contains business logic modules: tables, rows, views, automations, datasources, queries, permissions, workspaces, users, screens, backups, plugins, deployment, links, rowActions, oauth2, navigation, ai, dev. The SDK layer never touches the database directly — it uses the context system.

### 2. Database Layer (backend-core + integrations)

**Primary DB**: CouchDB (via PouchDB-compatible adapter). In-memory mode for testing.

**Document model**: All data is CouchDB documents with typed ID prefixes:
- `ta_` — Tables
- `ro_` — Rows
- `au_` — Automations
- `view_` — Views
- `screen_` — Screens
- `query_` — Saved queries
- `role_` — Access roles
- `datasource_` — External datasource configs
- `li_` — Relationship links

**Workspace per app**: Each "app" (workspace) gets its own CouchDB database. The `context.doInWorkspaceContext(workspaceId, fn)` pattern sets the active database. Apps exist in `dev` (editable) and `published` (deployed) mode with different database prefixes.

**External datasource adapters** (`integrations/`): Postgres, DynamoDB, MongoDB, Elasticsearch, CouchDB, SQL Server, S3, MySQL, REST, Firestore, Google Sheets, Redis, Snowflake, Oracle. Each adapter implements two interfaces: `Integration` (schema introspection) and `IntegrationBase` (runtime query execution). Custom plugins supported in self-hosted mode.

### 3. Context System (backend-core/src/context/)

Uses Node.js `AsyncLocalStorage` for request-scoped context:
- `doInTenant(tenantId, task)` — Tenant context
- `doInWorkspaceContext(workspaceId, task)` — Workspace/app context (sets CouchDB database)
- `doInIdentityContext(identity, task)` — User identity
- `doInAutomationContext(params, task)` — Automation context with snippet loading

Feature flags via PostHog with env var overrides: `USE_ZOD_VALIDATOR`, `AI_RAG_SHAREPOINT`, `AI_AGENT_INSTRUCTIONS`, `AI_TESTS`, `FRONT_COMPANION`, `DEBUG_UI`, `DEV_USE_CLIENT_FROM_STORAGE`

### 4. Builder → Client Pipeline

**App definition**: A `Screen` contains a recursive `Component` tree in `props`. Components are JSON objects with `_component` (type identifier), `_instanceName`, `_styles`, `_children`, `_conditions`, and arbitrary setting keys.

**Builder authoring** (`packages/builder/src/`):
- Svelte + Routify router
- Embeds client runtime as preview iframe (detected via `window["##BUDIBASE_IN_BUILDER##"]`)
- Component settings mapped to UX controls via `componentSettings.js` registry
- Binding engine (`dataBinding.ts`, 2106 lines): resolves readable bindings to runtime-safe format, discovers bindable properties from context tree (data providers, current user, URL params, device, state, roles, embeds)
- History store for undo/redo on all mutations

**Client rendering** (`packages/client/src/`):
- `ClientApp.svelte` → fetches app package → creates context store → renders layout
- `Component.svelte` — single recursive renderer: resolves constructor from `componentStore`, splits settings into static/dynamic, enriches bindings via `processObjectSync()` from string-templates, evaluates conditional UI, caches settings, observes context changes, renders with `<svelte:component this={constructor}>` and `<svelte:self>` for children
- Context is hierarchical and reactive — when any data changes, only components referencing that key re-enrich

**Component definitions**: Stored in `manifest.json` and custom component schemas. Each definition specifies: `component` type, `name`, `settings[]` (setting schema with key/type/section/label/defaultValue), `features`, `legalDirectChildren`, `illegalChildren`, `context` (data contexts exposed).

### 5. String Templates Engine (packages/string-templates/)

**Language**: Handlebars extended with custom helpers and JS execution sandbox.

**Processing pipeline** (three stages):
1. **Preprocessing**: Swap bracket notation to dot notation (`[field]` → `.[field]`), fix function spacing, normalize spaces, wrap expressions in `all` helper
2. **Template execution**: Handlebars compile + execute with custom helpers (object, js, decodeId, all, literal) and external function collections (math, array, number, url, string, comparison, regex, uuid)
3. **Postprocessing**: Convert `%LITERAL% type-value` markers back to typed values

**JS bindings**: `{{ js "base64encoded" }}` syntax. Executes in `isolated-vm` (backend) or `@budibase/vm-browserify` (frontend). Context provides `$` function for data access and `helpers` for JS utility functions.

**Key design**: When context keys match helper names, they're prefixed with `./` so context data takes precedence (e.g., `{ date: "foo" }` → `{{ ./date }}` not the `date` helper).

### 6. Automation Engine (packages/server/src/automations/)

**Architecture**: Bull queues (Redis-backed) + worker-farm thread pool.

- `automationQueue` — main processing queue
- Triggers: `ROW_SAVED`, `ROW_UPDATED`, `ROW_DELETED`, `WEBHOOK`, `APP`, `CRON`, `EMAIL`, `ROW_ACTION`
- 32+ action steps: CRUD rows, execute queries, run scripts, filter, loop, branch, delay, send email, outgoing webhook, Slack, Discord, Make, Zapier, n8n, API request, bash, OpenAI, and AI steps (agent, classify, extract, generate, promptLLM, summarise, translate)
- Thread isolation: queries and automations fork to separate Node processes via `worker-farm` with configurable timeouts (15s default for queries, 120s for automations)

### 7. AI Subsystem (packages/server/src/ai/ + sdk/workspace/ai/)

- **LiteLLM proxy** integration for multi-model LLM access
- **AI Agents**: Tool definitions, instructions, deployment management
- **Chat**: Conversation management, identity links, chat apps
- **RAG**: Retrieval-Augmented Generation with processing queue and knowledge source sync queue
- **Knowledge Bases**: File-based ingestion
- **Agent Tests**: Test suite definitions and execution

### 8. Worker (packages/worker/src/)

**Separate Koa process** (port 4002) handling global/SSO operations:
- Authentication (login/logout, SAML, OIDC, Google SSO)
- User management (CRUD, invitations, permissions)
- Email sending (Nodemailer + Handlebars templates)
- Tenant lifecycle (create, delete, status)
- System configuration, health/status, licensing
- SCIM protocol support
- Role management

**Middleware**: SCIM body handler, koa-body, Redis sessions, correlation IDs, pino logging, IP tracking, CSP, Passport.js authentication

### 9. Security Model

**Authentication**: JWT (cookie `Auth` or `x-budibase-token` header) + API keys (CouchDB view lookup, decrypted, internal flag)

**Session management**: Redis-backed with per-user caps (`MAX_SESSIONS_PER_USER`), TTL, oldest-eviction

**CSRF protection**: Synchronizer Token Pattern — compares `x-csrf-token` header against session-stored token. Exempts GET/HEAD/OPTIONS, JSON content types, internal API keys

**Permissions**: Hierarchical levels (PUBLIC < READ_ONLY < WRITE < POWER < ADMIN) across resource types (TABLE, VIEW, QUERY, USER, WORKSPACE, AUTOMATION, WEBHOOK)

**Tenancy**: Resolved from user, header (`x-budibase-tenant-id`), query param, subdomain, or route param — in priority order

### 10. Threading System (packages/server/src/threads/)

Uses `worker-farm` to fork separate Node.js processes for query execution and automation execution. Can be disabled via `DISABLE_THREADING` env var. Threads run in isolation to prevent memory leaks and crashes from affecting the main server process.

## Key Design Decisions

1. **CouchDB over SQL**: Sacrificed ACID transactions for JSON-native document storage, replication, and the ability to store arbitrary component trees without schema migration. The document ID prefix system (`ta_`, `ro_`, etc.) provides efficient type-based queries via CouchDB's `_all_docs` with `startkey`/`endkey`.

2. **Self-registering routes**: Route files import `endpointGroupList` and call `.group().get()` as side effects. Importing the file registers the route. This makes the route registry implicit but discoverable — you can tell what routes exist by grepping for `endpointGroupList`. Trade-off: tight coupling between import order and route registration.

3. **Component as JSON tree**: Rather than compiling the builder's visual design into code, Budibase stores it as a JSON component tree and interprets it at runtime. This means the builder can manipulate the tree directly (no compilation step), and the client just recursively renders whatever tree it receives. The cost is runtime interpretation overhead.

4. **Separate worker process**: Auth/SSO/user management runs in a separate process (worker) from the app server. This is a clean separation but means two Node.js processes to manage. The server handles per-app operations; the worker handles global operations.

5. **Handlebar templates with JS sandbox**: Rather than inventing a custom expression language, they extended Handlebars with a JS sandbox. The `{{ js "..." }}` pattern lets users write arbitrary JavaScript that runs in `isolated-vm`. This is powerful but opens security concerns — mitigated by sandbox isolation and disabled-helpers configuration.

6. **Thread-per-query isolation**: External datasource queries run in forked Node processes. This prevents a slow/broken query from blocking the event loop and crashing the server. The trade-off is IPC overhead per query.

## Comparison Notes

- **vs Xano**: Xano is a no-code backend (Postgres-based, API generation). Budibase is full-stack: it includes the frontend builder and client runtime. Xano focuses on backend/data; Budibase focuses on the entire app.
- **vs Retool/Appsmith/Tooljet**: These are also internal tool builders, but Budibase is fully open-source (GPL-3.0) with self-hosted as a first-class deployment mode. The CouchDB foundation is unusual — most competitors use Postgres/MySQL.
- **vs n8n**: n8n is workflow automation with 400+ integrations. Budibase has automation too, but it's embedded in the app-building context — automations trigger from app data changes, not just external events.
- **vs Airtable**: Airtable is a spreadsheet-database hybrid. Budibase can do that (tables + views + forms) but adds a full drag-and-drop UI builder and automation engine on top.
- **vs NocoDB/BaseRow**: These turn databases into spreadsheets. Budibase goes the other direction: build the UI first, connect data sources, add automation.

## Innovation Points

1. **Recursive single-component renderer**: `Component.svelte` renders any component type from the same loop — `<svelte:component this={constructor}>` + `<svelte:self>` for children. Most low-code platforms have per-component-type renderers.

2. **Document ID taxonomy**: The `{type}{SEPARATOR}{id}` CouchDB ID scheme enables efficient type-range queries without secondary indexes. Every entity type is queryable via `_all_docs?startkey="ta_"&endkey="ta_￰"`.

3. **Binding-to-context mismatch detection**: Components only re-render when their specific binding keys change, not on any context change. The `handleContextChange(key)` method checks if `key` appears in the component's binding strings.

4. **Three-stage template pipeline**: Preprocessing (bracket→dot, helper wrapping) → Handlebars execution → Postprocessing (literal type markers). This allows Handlebars to handle non-string types via marker tokens that survive the string-based rendering.

5. **Workspace-as-database**: Each app IS a CouchDB database. This makes export/backup trivial (dump the database) and enables per-app replication. It also means every app has the same document structure.

## Scale Notes

- Server: ~177K lines TypeScript
- Builder: ~108K lines Svelte/TypeScript
- Client: ~20K lines Svelte/TypeScript
- Worker: ~16K lines TypeScript
- Total monorepo: well over 350K lines across 14 packages
- 32 automation step types
- 12+ external datasource adapters
- ~40 route modules in server
- 5 built-in permission levels across 7 resource types
