# TriadJS

TypeScript API framework where specification, implementation, validation, and tests are the same source of truth. Write a schema once using `t.model()`, and the framework derives static types, runtime validation, OpenAPI 3.1, AsyncAPI 3.0, executable BDD scenarios, automatic adversarial fuzz tests, Gherkin feature files, typed frontend hooks (React/Solid/Vue/Svelte Query), typed WebSocket clients, database schemas (Drizzle/Postgres/SQLite), form validators, and breaking-change detection — all without codegen round-trips or hand-maintained artifact files.

---

## Architecture

Triad is a monorepo: 21 packages, 4 reference examples, 83+ behavior scenarios, 1000+ tests. Pre-1.0, MIT licensed.

**The core** (`@triadjs/core`, ~2,400 lines) is a schema DSL based on the `SchemaNode<TOutput>` base class. Every schema kind (string, int32, enum, model, array, etc.) extends it with immutable chainable methods via `_clone()`. `t.model()` produces a `ModelSchema` with pick/omit/partial/required/extend/merge — the DDD primitives. The `t` namespace doubles as a type-level operation: `t.infer<typeof Pet>` extracts the TypeScript output type.

`endpoint()` takes a declarative config object and returns a runtime `Endpoint`. The handler receives a `HandlerContext` with fully inferred `params`, `query`, `body`, `headers`, and a `respond` map keyed by declared status codes — `ctx.respond[201](pet)` is a compile error if 201 isn't in the responses config. `buildRespondMap()` validates outgoing payloads against declared schemas at runtime.

`channel()` defines WebSocket channels with the same declarative shape: connection params, client messages, server messages, per-message-type handlers, and a typed `ctx.broadcast.*` / `ctx.send.*` surface. Supports three auth strategies (header, first-message, none) and a `withState<T>()` helper for typed per-connection state.

The `Router` groups endpoints and channels into optional DDD bounded contexts. Uses `Symbol.for()` brands instead of `instanceof` for cross-module-graph identity — essential because the CLI loads user routers via `jiti`, which may resolve `@triadjs/core` through a separate module graph.

**Radiating outward from the core are generators** — pure functions that walk the router and produce artifacts:

- `@triadjs/openapi` — Router → OpenAPI 3.1 document. Express-style `:id` converted to `{id}`. Bounded contexts become top-level tags. `t.empty()` correctly omits `content`. `t.file()` emits `multipart/form-data`.
- `@triadjs/asyncapi` — Router → AsyncAPI 3.0 document. Same shared `components/schemas` with the OpenAPI generator.
- `@triadjs/gherkin` — Behaviors → `.feature` files grouped by bounded context.
- `@triadjs/drizzle` — Router → `TableDescriptor[]` via structural walk, then dialect-specific emission (SQLite, Postgres). Dialect-neutral `LogicalColumnType` IR. Value objects flattened into prefixed columns. `primaryKey` storage hint identifies table models.
- `@triadjs/tanstack-query` and frontend variants — Router → typed query/mutation hooks.
- `@triadjs/forms` — Request body schemas → form resolver validators.

**Adapters** mount a Triad router onto real HTTP frameworks: Fastify (+ WebSocket via `@fastify/websocket`), Express, Hono (Node/Deno/Bun/Workers), and AWS Lambda (API Gateway v1/v2, ALB, Function URL). All four produce byte-identical error envelopes.

**The CLI** (`@triadjs/cli`) has 10 commands: `test`, `fuzz`, `docs`, `docs check`, `gherkin`, `db generate`, `frontend generate`, `new`, `mock`, `validate`. `triad docs check --against <ref>` classifies API changes as safe/risky/breaking — a Triad-unique capability since the router is typed source, not generated YAML.

**The test runner** (`@triadjs/test-runner`, ~3,200 lines) executes behaviors in-process (no HTTP server). Per-scenario isolation via `servicesFactory` + `teardown`. Validates request parts against declared schemas, invokes handlers with synthetic `HandlerContext`, validates responses against schemas, and runs assertions. Channel tests use an in-memory `ChannelHarness` that mirrors the Fastify `ChannelHub` semantics.

## Key Techniques

**Immutable schema builders.** Every chainable method (`.doc()`, `.optional()`, `.default()`, `.identity()`, `.storage()`) returns a new instance via `_clone()` with spread-merged metadata. Schemas are safe to share and compose.

**Self-referential generic constraint.** `ModelSchema` uses `TShape extends { [K in keyof TShape]: SchemaNode<any> }` instead of `Record<string, SchemaNode<any>>`. The latter makes TypeScript apply a contextual type that widens field generics — an `EnumSchema<['dog','cat']>` collapses into `SchemaNode<any>`. The self-referential form preserves exact inferred types.

**Phantom state witness.** Channel's `TState` is inferred from `state: {} as ChatRoomState` in the config rather than `<ChatRoomState>` as a type argument. This sidesteps TypeScript's partial-inference limitation where providing one explicit generic blocks inference of all others.

**Deferred auto-scenario expansion.** `scenario.auto()` returns an `AutoScenarioMarker` — not concrete scenarios. The test runner expands it at execution time by reading the endpoint's schema constraints. Generated scenarios always reflect the current schema; no stale codegen.

**Kind-based structural discrimination.** All generators and the validator walk schemas via `schema.kind` string discriminant rather than `instanceof`. Works across duplicate module graphs — a hard requirement because the CLI's jiti-based router loading creates separate `@triadjs/core` instances.

**Five adversarial test generators** fire from a single `scenario.auto()` call: missing-field (remove each required field), boundary (±1 at numeric/string/array limits), invalid enum, type confusion (number ↔ string, etc.), and random valid (optional fast-check property testing).

**Regex-based BDD parser.** `parseAssertion()` in behavior.ts parses natural-language `then` descriptions into structured `Assertion` objects. Pattern order matters — longer patterns checked before shorter. Supports 12 assertion types including channel variants (`"alice receives a message event"`, `"connection is rejected with code 4401"`).

**Dialect-neutral DB codegen IR.** The Drizzle walker maps schema kinds to `LogicalColumnType` (string/uuid/datetime/integer/bigint/float/double/boolean/enum/json). SQLite and Postgres emitters map to dialect-specific column helpers. Adding MySQL is ~15 lines.

**`Object.hasOwn()` guard against prototype pollution.** Property-based fuzzing (Phase 25) discovered that model validation used plain member access (`input[fieldName]`), which resolved `Object.prototype.valueOf` when a field was named `valueOf`. Fixed with `Object.hasOwn(input, fieldName)`.

## Design Decisions

**Single source of truth over composable libraries.** The bet: deriving everything from one TypeScript definition is better than stitching Zod + zod-to-openapi + Cucumber + Drizzle + hand-written fetch wrappers. The win isn't fewer dependencies — it's that a schema change is impossible to forget to propagate.

**Singular beforeHandler over middleware chain.** Documented rationale: one function keeps the lifecycle legible, type inference for `TBeforeState` only works with a single return type, and users who need composition write plain function calls. Middleware is easy to add; hard to remove.

**In-process testing over HTTP.** Handlers are invoked directly with synthetic `HandlerContext`. No server startup, no port conflicts, framework-agnostic. Each example also has e2e HTTP/WebSocket tests for wire-level confidence.

**Explicit `primaryKey` over `identity()` promotion.** The Drizzle walker uses `.storage({ primaryKey: true })` to detect table models, not `.identity()`. Identity is domain; primary key is storage. Being explicit prevents surprising generated output.

**ESM-only.** No CommonJS. Adapters pull in Node built-ins only where needed. The Hono adapter works on Deno/Bun/Cloudflare Workers unchanged.

**AI-first design.** Triad's north star is that an AI coding assistant should understand an entire API by reading one place. The Claude Code plugin ships 10 skills and 8 slash commands that write idiomatic TriadJS code against a documented phrase table. The AI Agent Guide is structured as canonical grounding specifically to prevent LLM hallucination.

## Comparison Notes

**vs. tRPC:** Both share types from server to client. tRPC is RPC-style; Triad is REST-first with OpenAPI as first-class output. tRPC generates one typed client; Triad generates hooks for 4 query libraries + WebSocket clients + form validators.

**vs. Zod stack:** The standard approach (Zod + zod-to-openapi + Cucumber + Drizzle + hand-written fetch) works but introduces drift — schemas live in multiple files that fall out of sync. Triad's thesis is that a single definition prevents this.

**vs. FastAPI (Python):** FastAPI's Pydantic → OpenAPI pattern is the closest analogue, but doesn't extend to BDD tests, DB schemas, frontend hooks, or WebSocket clients.

**vs. Hono:** Hono is a web framework. Triad's Hono adapter lets you use Hono as the HTTP layer while getting Triad's codegen pipeline on top.

**vs. [[Swamp Club]]:** Both are typed-model frameworks targeting agent workflows. Triad uses its own schema DSL + endpoint definitions + multi-target codegen. Swamp Club uses Zod + DAG execution + encrypted vaults.

**vs. [[DAB]] (Microsoft Data API Builder):** DAB generates REST/GraphQL/MCP from a database. Triad generates a database from REST/WebSocket definitions. The direction of the single source of truth is inverted.

---

#tool #project #api #typescript #codegen #framework

*Source: [[raw/triadjs]] — repo analysis from https://github.com/justhamade/triadjs*
*Last updated: 2026-06-11*
