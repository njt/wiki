# AWS Emulator (Repository Anatomy)

AWS's own local emulator of the AWS APIs: a Rust workspace of 437 per-service crates (~1.16M lines) that serves a faithful AWS endpoint on `127.0.0.1:9297` or runs in-process as an SDK `HttpClient`. Its architecture is a machine for turning **Smithy API models into emulated services** — the model drives codegen of the wire protocol and routing table, and humans/agents write only the behavioral seams — with fidelity managed in public via published fidelity scores and ~6,900 tracked gaps. This is the artifact behind the launch essay in [[AWS Emulator and the Engineer's Job (Sivasubramanian)]], read here from the repository itself.

---

## Architecture

The repo is a Cargo workspace with a strict one-crate-per-service layout (`services/service-com-amazonaws-<name>-<endpoint>/`):

- `smithy-model/<name>.json` — the verbatim Smithy model (DynamoDB's is 948KB). This is the spec-as-source-of-truth.
- `build.rs` — one call into `emulator_core_build_support::generate()`, which emits the service declaration, routing table, and scaffolding from the model.
- `src/service.rs` — the hand-edited claim/wording layer. The generated `ServiceDefinition` (identity, wire protocol, operation routes) is "reached, not edited"; five opt-in seams (`can_handle`, `override_operation`, `override_refusal`, `override_routing_error`, `override_framework_error`) let a service customize routing and error envelopes without touching generated code.
- `src/state.rs` — the service's in-memory store. DynamoDB's is ~5,000 lines: tables, items, capacity accounting, an order-preserving key encoding, and the condition/update expression language.
- `src/ops/<operation>.rs` — one file per operation (DynamoDB has ~70). Each is a struct implementing `smithy_server_runtime::Operation`; the framework drives deserialize → validate → `handle` → serialize, and only `handle` is hand-written.

Core crates: `core/router/` mounts every service via generated `mount_chunk_NN` helpers into a `MultiServiceRouter`; `core/embedded/` (`AwsEmulatorEmbedded`) is the in-process facade. `vendor/` holds the framework (`engine`, `server`, `smithy-server-runtime`, `smithy-macros`, build-support, telemetry) — regenerated on update, never edited.

**Two front doors, one path.** `MultiServiceRouter` implements the emulator `EmulatorService` trait *and* the SDK `HttpConnector` trait, so the standalone server and embedded mode drive the same `dispatch(req)` and cannot diverge in routing behavior. In embedded mode, `emulator.http_client()` drops straight into an SDK config's `.http_client(..)`, and `aws_credentials_provider()` supplies SigV4 credentials minted by the emulator itself — no socket, no certificate, no environment variables. Each embedded instance owns a *fresh* engine; two instances never share state.

**Routing is claim-based, not table-based.** The router asks each service's `can_handle` whether it claims a request (seeded from the model's SigV4 signing name). Zero claimants is 404 `UnknownService`; several is 500 `AmbiguousService` — ambiguity is never resolved by registration order. This matters because many AWS services share signing names (`bedrock` is claimed by six services, `lex` by four); `services/common/src/` holds shared claim rules (`claims_modeled_route`, `shared_path`, `chime_family`) that disambiguate by modelled URI routes, path segments, or ARN ownership, so the same rule text exists in exactly one place.

## Key techniques

- **Model-generated types with Smithy-derived serde.** `smithy-macros` implements `SmithyStruct`/`SmithyUnion` derives: the Rust field type only says `Option`/container; all real type information (recursively through lists, maps, unions) comes from the Smithy model, so wire deserialization is generated, not hand-matched.
- **Shared stores in a type-map engine.** `InMemoryEngine` is a `HashMap<TypeId, Box<dyn Any>>`; each service reaches its state via `engine.state::<MyState>()`, getting a clone-cheap `ServiceState<T>` wrapping `Arc<Mutex<T>>`. Hand-written `Clone` avoids a spurious `T: Clone` bound. Lock poisoning is deliberately *recovered from* rather than propagated — a long-lived emulator must survive one panicking handler.
- **Anti-fabrication doctrine, written into the engine docs.** The `engine` crate's module docs forbid echoing input as output, pattern-matching source text, or hand-writing evaluators for a subset of a language ("answers every other input wrongly while reporting success"). Where the engine can't be faithful, it must return the real engine's error or a 501 — never `todo!()`/`panic!()`, which would render as a 500 the real AWS never sends.
- **In-process native listeners.** Services needing a real protocol (PostgreSQL, MySQL, a message broker) don't bind sockets: they hand a connection handler to `engine.listeners().tcp(handler)`, and the *core* owns the socket, port, and TLS upgrade, returning `listener.url()` for the control-plane endpoint fields. This keeps a wildcard/IPv6 bind from producing a wrong advertised endpoint.
- **Real request authentication.** The vendored `server` crate verifies SigV4 by *re-signing* the request with the SDK's own `aws-sigv4` signer against minted credentials, including presigned-URL validation capped at AWS's 7-day maximum. Credential failures are deliberately reworded per-service: the framework produces an authorization-style denial, but AWS answers a credential failure with an authentication code, and `service_common::authentication_refusal` owns that correction once for all services.
- **Compile-time house rules.** `#![forbid(unsafe_code)]` everywhere. `build-support` contains a drift-guard that *refuses to compile* a crate whose tests are one cargo target per file — because N test targets statically link N copies of the whole runtime; one `tests/all.rs` target with `#[path]` modules pays the dependency closure once. That's the invisible-decay class of problem turned into a build error.
- **Stack discipline at 437 services.** Mounting hundreds of generated services costs stack proportional to the deepest generated deserializer. `main.rs` forbids `#[tokio::main]`, runs on a 64MB explicit thread, sizes tokio workers explicitly, and splits `build_router` into `#[inline(never)]` chunk helpers so no single frame holds the whole workspace — the docs warn that removing any of these aborts the process before either listener binds.

## Design decisions

- **Faithfulness over convenience, with honest refusal.** The 501-until-implemented default and the no-fabrication rule trade "works a bit" for "wrong never pretends to be right." A test passing against the emulator is evidence, not accident.
- **Everything shared lives once.** `services/common/` centralizes the one IAM policy evaluator, the one ARN parser, the one RFC 3339 reader, the one `X-Amz-Target` reader, ten shared stores — so two services asked the same question cannot answer differently. It costs a fat common crate but buys consistency across 437 services, which for an emulator is the product feature.
- **No isolation boundary, so bound and contain.** In-process execution of user code (Lambda handlers, expression languages) runs under wall-clock timeouts plus step budgets (a timeout can't preempt a CPU-bound loop on the request thread) with no outbound network. The emulator is explicitly not for production or security testing.
- **The spec is the schema, the tests are the fidelity report.** Published per-service fidelity scores and the `issues/things-that-do-not-work/` → `things-that-now-work/` pipeline make incompleteness a tracked, public artifact instead of a README caveat. The build cost is stated honestly: ~100GB of `target/` for a full debug build.
- **CC0 plus no-PRs.** The code is maximally open but the contribution surface is issues only — Amazon keeps the agent-driven pipeline as the sole writer.

## Comparison notes

Against [[floci]] — the community's replacement for LocalStack Community — the contrast is sharp. Floci is 47 services in 24ms and 13 MiB, mixing in-process stateless services with real Docker containers for Lambda/RDS/EC2. AWS Emulator is the opposite trade: hundreds of services, a 100GB build, and *in-process interpreters* rather than containers, justified by the engine doctrine that a real interpreter answers any input correctly where a container-per-service design answers only what's exercised. Floci optimizes for developer-cycle speed; AWS Emulator optimizes for behavioral fidelity and agent-workability (its quick start begins with installing an agent skill).

It is also the concrete existence proof for [[AWS Emulator and the Engineer's Job (Sivasubramanian)]]: the essay's "specs, tests, and merge gates" are visible here as Smithy models checked verbatim into each crate, per-service fidelity scores, and a scaffold that makes every unimplemented operation fail loudly. Where the essay argues agents need checkable environments, this repo shows the environment itself compiled out of 437 specifications — and, notably, the claim-rule and error-wording code reads like the output of a careful spec process rather than a code dump, which is the essay's thesis made inspectable.

For [[Guardrails and Feedback Loops]] it adds a mechanism the essay only gestured at: deterministic refusal (501) instead of plausible fabrication, compile-time drift guards against known decay patterns, and a public issues ledger of ~6,900 gaps that functions as a continuously updated fidelity contract between the emulator and real AWS.

---
*Sources: [[raw/aws-emulator]], [[summary/aws-emulator]]*
*Last updated: 2026-11-13*
