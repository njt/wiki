---
url: https://github.com/justhamade/triadjs
title: "TriadJS — Single-source-of-truth TypeScript API framework"
author: justhamade
date_fetched: 2026-06-11
date_published: unknown
---

# TriadJS — Full Repo Analysis

## Overview

Triad (also called TriadJS) is a TypeScript-first API framework built on the idea that an API's *specification*, *implementation*, *validation*, and *tests* should never drift apart because they are the same thing. It's a monorepo of 21 packages, 4 reference examples, 83+ behavior scenarios, and 1000+ tests.

The central thesis: write TypeScript once using Triad's declarative DSL, and you get runtime validation, static types, OpenAPI 3.1, AsyncAPI 3.0, executable BDD scenarios, automatic adversarial tests, Gherkin feature files, typed frontend hooks (React/Solid/Vue/Svelte Query), typed WebSocket clients, database schemas (via Drizzle), form validators, and breaking-change detection — all generated from the same source of truth.

## Architecture

### Monorepo Structure

```
triad/
  packages/
    core/          — Schema DSL, endpoint(), channel(), router, behaviors, scenario.auto()
    openapi/       — Router → OpenAPI 3.1 document
    asyncapi/      — Router → AsyncAPI 3.0 document
    gherkin/       — Behaviors → .feature files
    test-runner/   — In-process BDD runner + schema-derived auto-scenario generation
    fastify/       — Fastify HTTP + WebSocket adapter
    express/       — Express HTTP adapter
    hono/          — Hono adapter (Node, Deno, Bun, Cloudflare Workers)
    lambda/        — AWS Lambda adapter (API Gateway v1/v2, ALB, Function URL)
    drizzle/       — Triad schemas → Drizzle tables + SQL migrations
    tanstack-query/— Router → typed React Query hooks
    solid-query/   — Router → typed Solid Query hooks
    vue-query/     — Router → typed Vue Query hooks
    svelte-query/  — Router → typed Svelte Query hooks
    forms/         — Router → form validators
    channel-client/— Router → typed WebSocket clients (vanilla TS + framework variants)
    jwt/           — requireJWT BeforeHandler wrapping jose
    otel/          — OpenTelemetry tracing (opt-in router wrapper)
    metrics/       — Prometheus metrics (opt-in router wrapper)
    security-headers/ — Security headers middleware (Fastify, Express, Hono)
    cli/           — triad test, triad fuzz, triad docs, triad new, triad mock, triad db, etc.
  examples/
    petstore/      — Fastify + WebSocket channels + Drizzle + SQLite
    tasktracker/   — Express + auth + pagination + ownership
    bookshelf/     — All features combined (tutorial's final state)
    supabase-edge/ — Hono + Supabase + Deno edge deployment
  plugin/          — Claude Code plugin: 10 skills, 8 slash commands
  docs/            — Tutorial, guides, AI agent reference, internal prompts
```

### Core Package Architecture

The `@triadjs/core` package (~2,400 lines of source across 15 files) is the nucleus. Every other package consumes it:

**Schema DSL** (`packages/core/src/schema/`):
- `SchemaNode<TOutput>` base class (340 lines in types.ts) — immutable builder pattern with `_clone()`, `_validate()`, `_toOpenAPI()`. Every chainable method (`.doc()`, `.example()`, `.optional()`, `.default()`, `.identity()`, `.storage()`) returns a new instance.
- 14 schema kinds: string, int32, int64, float32, float64, boolean, datetime, enum, literal, unknown, empty, file, array, record, tuple, union
- `ModelSchema` (253 lines) — the DDD entity type with pick/omit/partial/required/extend/merge/named/identityField. Self-referential generic constraint (`TShape extends { [K in keyof TShape]: SchemaNode<any> }` instead of `TShape extends ModelShape`) to prevent TypeScript from widening field generics during inference.
- `ValueSchema` — DDD value objects (flattened into host table columns by Drizzle codegen)
- `t` namespace — factory functions + `t.infer<T>` type-level operation via namespace merge

**Endpoint** (`packages/core/src/endpoint.ts`, 191 lines):
- `endpoint(config)` takes a declarative config {name, method, path, request, responses, handler, behaviors, beforeHandler} and returns a runtime `Endpoint`
- Inline `request.params`/`query`/`headers` are normalized into anonymous `ModelSchema` instances via `normalizeRequestPart()`
- The handler receives `HandlerContext<TParams, TQuery, TBody, THeaders, TResponses, TBeforeState>` with full type inference

**Handler Context** (`packages/core/src/context.ts`, 175 lines):
- `RespondMap<TResponses>` — type-level map keyed by declared status codes; `ctx.respond[201](pet)` is a compile error if 201 isn't declared
- `buildRespondMap()` validates outgoing payloads against the declared response schema at runtime, catching handler bugs
- `ServiceContainer` — extensible via TypeScript declaration merging

**Router** (`packages/core/src/router.ts`, 249 lines):
- `Router` class with `rootEndpoints`, `rootChannels`, and `contexts` (DDD bounded contexts)
- `Symbol.for('@triadjs/core/Router')` brand for cross-module-graph identity (critical because CLI uses jiti to load user routers, which may resolve `@triadjs/core` through a different module graph)
- `isRouter()` static — structural brand check, not `instanceof`
- `contextOf(route)` — maps any endpoint/channel back to its bounded context

**Behavior** (`packages/core/src/behavior.ts`, 394 lines):
- Fluent BDD builder: `scenario(name).given(desc).when(desc).then(desc).and(desc)`
- `parseAssertion()` — regex-based natural language parser producing structured `Assertion` variants (status, body_matches, body_has, body_is_array, body_is_empty, body_length, body_has_code, channel_receives, channel_not_receives, connection_rejected, channel_message_has, custom)
- Channel assertions support named clients (`"alice receives a message event"`) and wildcards (`"all clients receive a presence event"`)

**Scenario Auto** (`packages/core/src/scenario-auto.ts`, 85 lines):
- `auto()` returns a marker object (`AutoScenarioMarker`) spread into `behaviors: [...scenario.auto()]`
- The marker is NOT expanded at definition time — it's a deferred instruction the test runner resolves at execution time by reading the endpoint's schema

**Channel** (`packages/core/src/channel.ts`, 515 lines):
- `channel(config)` — declarative WebSocket channel definition, same shape as `endpoint()`
- Phantom state witness pattern: `state: {} as ChatRoomState` in the config object avoids TypeScript's partial-inference limitation (providing `<MyState>` explicitly would block inference of `TParams`, `TQuery`, etc.)
- `channel.withState<T>()` — ergonomic alternative that curries the state type
- Three auth strategies: `header`, `first-message` (browser-friendly), `none`
- `Symbol.for('@triadjs/core/Channel')` brand for cross-module identity

**BeforeHandler** (`packages/core/src/before-handler.ts`, 128 lines):
- Singular function (not middleware chain) — deliberate choice for type inference simplicity and legibility
- Returns `{ ok: true, state }` to thread typed state into the handler, or `{ ok: false, response }` to short-circuit
- Runs BEFORE request schema validation — auth code can reject with 401 instead of seeing validation 400s

### Test Runner Architecture

`@triadjs/test-runner` (~3,200 lines across 14 files):

**Runner** (`runner.ts`, 568 lines):
- `runBehaviors(router, options)` — walks every endpoint, expands auto-scenarios, runs each behavior in-process (no HTTP server)
- Per-scenario flow: servicesFactory (fresh container) → setup (seed data) → substitute placeholders → invoke beforeHandler → validate request parts against schemas → invoke handler → validate response against schema → run assertions → teardown
- Auto-scenario handling: `__autoOutcome === 'rejected'` scenarios PASS when validation fails (the endpoint correctly rejects bad input) and FAIL when validation passes (the schema should have rejected)
- beforeHandler short-circuit path flows through the same response validation + assertions pipeline as the normal handler path

**Auto Generators** (`auto-generators.ts`, 406 lines):
- 5 adversarial test generators:
  1. Missing-field — one scenario per required field, with that field removed
  2. Boundary — ±1 at numeric min/max, string minLength/maxLength, array minItems/maxItems
  3. Invalid enum — out-of-range enum value per enum field
  4. Type confusion — wrong JS type per field (number→string, string→number, etc.)
  5. Random valid — N random inputs via fast-check (optional, requires `fast-check` installed)
- `buildBaseline()` produces a minimal valid input by kind: UUIDs get `00000000-...`, emails get `test@example.com`, numbers get min or 0
- `buildArbitrary()` converts `FieldDescriptor[]` into fast-check arbitraries for property-based testing

**Channel Runner** (`channel-runner.ts`, 628 lines):
- `ChannelTestClient` and `ChannelHarness` — in-memory multi-client simulator mirroring the Fastify `ChannelHub` semantics
- `runChannelBehaviors()` — walks channels, interprets `when` descriptions ("alice connects", "alice sends message", "alice disconnects"), executes multi-client `.andWhen()` sequences

### Generator Architecture

**OpenAPI** (`packages/openapi/src/generator.ts`, 381 lines):
- Pure function: `generateOpenAPI(router)` → `OpenAPIDocument` (plain JS object)
- Express-style `:id` → OpenAPI `{id}` via `convertPath()`
- Bounded contexts → top-level `tags[]` with descriptions
- `t.file()` → `multipart/form-data` content type with `format: binary`
- `t.empty()` (204/205/304) → omits `content` (per HTTP spec)
- Serialization (YAML/JSON) in separate `serialize.ts`

**Drizzle Codegen** (`packages/drizzle/src/codegen/walker.ts`, 366 lines):
- `walkRouter()` → `TableDescriptor[]` — structural walk (kind-based, not instanceof)
- Table detection heuristic: a model becomes a table if ANY field has `.storage({ primaryKey: true })`. Derived schemas (CreatePet, UpdatePet), input DTOs, and error shapes are automatically excluded.
- Value objects (Money) are flattened into prefixed columns (`adoptionFee` + `amount`/`currency` → `adoption_fee_amount`, `adoption_fee_currency`)
- Dialect-neutral `LogicalColumnType` IR (`string | uuid | datetime | integer | bigint | float | double | boolean | enum | json`) — emitters map to dialect-specific helpers
- Nested `ModelSchema` fields throw `CodegenError` pointing at the fix
- `Object.hasOwn()` guard in model validation was added after property tests discovered the `valueOf`/`toString`/`constructor` prototype pollution bug (Phase 25)

## Key Techniques

### 1. Immutable Schema Builder Pattern
Every schema node extends `SchemaNode<TOutput>` which has `_clone(metadata, isOptional, isNullable): this`. Every chainable method creates a new instance via spread-merging metadata. This means schemas are safe to share and compose without mutation side effects.

### 2. Cross-Module-Graph Brand Identity
Both `Router` and `Channel` use `Symbol.for('@triadjs/core/...')` brands rather than `instanceof` checks. This is essential because the CLI uses `jiti` to load user routers, which may resolve `@triadjs/core` through a different module graph than the CLI's own copy. `Symbol.for()` is global across the process, so the brand check works even when two copies of the class exist.

### 3. Self-Referential Generic Constraint
`ModelSchema` uses `TShape extends { [K in keyof TShape]: SchemaNode<any> }` instead of `TShape extends Record<string, SchemaNode<any>>`. The latter makes TypeScript apply it as a contextual type, widening each field's generic (an `EnumSchema<['dog','cat']>` collapses into `SchemaNode<any>`). The self-referential form preserves exact inferred types per field.

### 4. Phantom State Witness Pattern
Channel's `TState` is inferred from a phantom `state` field in the config (`state: {} as ChatRoomState`) rather than requiring `<ChatRoomState>` as a type argument. This sidesteps TypeScript's partial-inference limitation: providing `<MyState>` explicitly would block inference of `TParams`, `TQuery`, and friends.

### 5. Deferred Auto-Scenario Expansion
`scenario.auto()` returns a marker object (`AutoScenarioMarker`), not concrete scenarios. The test runner calls `expandAutoMarker()` at execution time, reading the endpoint's schema descriptors and generating concrete scenarios. This means the auto-scenarios always reflect the current schema — no stale generated code.

### 6. Kind-Based Structural Discrimination
All codegen tools (OpenAPI generator, Drizzle walker, test runner's schema reader) dispatch on `schema.kind` (string discriminant) rather than `instanceof`. This works across duplicate module graphs and doesn't require importing schema subclass constructors.

### 7. Respond Map as Type-Level Contract
`buildRespondMap()` creates a runtime object where each status code key is a function that validates its argument against the declared schema. Combined with the `RespondMap<TResponses>` type, this means `ctx.respond[201](malformedData)` is both a compile error (if 201 isn't declared) and a runtime error (if the data doesn't match the schema). The response safety net also catches handlers that bypass `ctx.respond` — the test runner validates the returned `HandlerResponse` against the declared schema.

### 8. Regex-Based Natural Language Parser
`parseAssertion()` in behavior.ts uses a series of regex patterns to parse BDD `then` step descriptions into structured `Assertion` objects. The pattern order matters — longer patterns must be checked before shorter ones (e.g., `"response body has code \"NOT_FOUND\""` is checked before the generic `"response body has <path> \"<value>\""` pattern). Channel assertions use similar regex parsing for multi-client interaction descriptions.

### 9. Dialect-Neutral Codegen IR
The Drizzle walker maps Triad schema kinds to a `LogicalColumnType` enum (`string | uuid | datetime | integer | bigint | float | double | boolean | enum | json`) that is dialect-agnostic. SQLite and Postgres emitters each map these to dialect-specific helpers. Adding MySQL requires ~15 lines of column-helper mapping in the emitter.

## Design Decisions

**Single source of truth over flexibility.** The entire framework bets that deriving everything from one TypeScript definition is better than composing separate libraries (Zod + zod-to-openapi + Cucumber + Drizzle + hand-written fetch wrappers). The value isn't fewer dependencies — it's that a schema change is impossible to forget to propagate because there's nothing to propagate to.

**Singular beforeHandler over middleware chain.** Deliberate choice documented in `before-handler.ts`. A single function keeps the request lifecycle legible at a glance. Users who need composition write plain functions. Middleware stacks make type inference harder and the `TBeforeState` approach only works with a single function return type.

**Structural discrimination over `instanceof`.** Every codegen tool walks schemas via `kind` string discriminants. This is explicitly designed for the CLI's `jiti`-based router loading, where `instanceof` checks would fail across duplicate module graphs.

**In-process testing over HTTP.** The test runner invokes handlers directly with synthetic `HandlerContext`, not through HTTP. This makes tests fast (no server startup), deterministic (no port conflicts), and framework-agnostic. Each of the 4 examples also has e2e HTTP/WebSocket tests for wire-level confidence.

**ESM-only.** No CommonJS support. Build output is ESM via `tsup`. Adapters pull in Node built-ins only where needed (Fastify, Express, Lambda). The Hono adapter works on Deno/Bun/Cloudflare Workers unchanged.

**Pre-1.0 with API stability caveats.** 26 phases shipped, APIs may still shift. Users are told to pin exact versions if adopting early.

**Explicit primary key over identity promotion.** The Drizzle walker uses `.storage({ primaryKey: true })` to identify table models, not `.identity()`. Identity is a domain concept; primary key is a storage concept. This prevents surprising generated output.

**Flat wiki-style docs over generated reference.** Documentation is hand-written markdown organized by task (pick your stack, learn by building, work with AI). The AI Agent Guide is a canonical source-grounded reference specifically designed for LLM coding assistants.

## Comparison Notes

**vs. tRPC:** tRPC shares the "types from server to client" philosophy but is RPC-style (procedures, not resources). Triad is REST-first with OpenAPI as a first-class output. tRPC generates a typed client; Triad generates typed hooks for 4 query libraries plus typed WebSocket clients plus form validators.

**vs. Zod + zod-to-openapi + ... :** The "stitch it yourself" approach. Triad's thesis is that stitching introduces drift — your Zod schemas, OpenAPI YAML, test fixtures, and DB schema definitions are separate files that fall out of sync. Triad makes them the same file.

**vs. FastAPI (Python):** FastAPI uses Pydantic for the same "schema → OpenAPI + validation" trick, but doesn't generate BDD tests, DB schemas, frontend hooks, or WebSocket clients. Triad's scope is wider.

**vs. NestJS:** NestJS is a full MVC framework with decorators, modules, and dependency injection. Triad is lighter — a schema DSL, a router, and generators. No decorators, no module system, no opinion on project structure beyond the router.

**vs. Hono:** Hono is a web framework with built-in RPC and Zod integration. Triad's Hono adapter lets you use Hono as the HTTP layer while getting Triad's codegen pipeline.

**vs. Swamp Club:** Both are agent-first workflow frameworks with typed models. Swamp Club uses Zod + DAG execution + encrypted vaults. Triad uses its own schema DSL + singular endpoint definitions + multi-target codegen.

**Relation to AI coding agents:** Triad's north star is that an AI coding assistant should understand an entire API by reading one place. The Claude Code plugin ships 10 skills and 8 slash commands that write idiomatic TriadJS code against a documented phrase table. The AI Agent Guide is structured as canonical grounding for LLMs, specifically designed to prevent hallucination.

## Property Testing Discovery (Phase 25)

Property-based fuzzing with `fast-check` discovered a real bug: `ModelSchema._validate` read fields via plain member access (`input[fieldName]`), which resolved `Object.prototype.valueOf` when a field was named `valueOf` (and similarly for `toString`, `constructor`, `hasOwnProperty`). Fixed by adding `Object.hasOwn(input, fieldName)` guard. This is a concrete example of the value of property testing on a schema validation engine.

---

*Analysis by Claude Code, 2026-06-11. Full repo clone (shallow) from https://github.com/justhamade/triadjs.*
