# Aspire

Microsoft's code-first, multi-language toolchain for distributed applications: you describe services, containers, databases, and their connections in one "app host" program, and that single definition runs the whole app locally with OpenTelemetry observability *and* carries into deployment. It's the clearest answer to the local-dev-to-production gap in the .NET world — and its polyglot code-generation layer plus agent-aware CLI make it more interesting than a mere orchestration library.

---

## Architecture

Aspire is a monorepo (~2,800 C# files across ~87 `src/` projects) built around one central idea: **a declarative app model that decouples "what you want" from "how it runs."**

- **The app model** (`src/Aspire.Hosting/ApplicationModel/`) is the heart. `IResource` is deliberately minimal — just `Name` plus an `Annotations` collection — and capability interfaces (`IResourceWithEndpoints`, `IResourceWithConnectionString`) are *derived* from annotations. This is the inversion that matters: instead of a growing class hierarchy (RedisResource → ContainerResource → …), resources are dumb bags of metadata, and behavior is attached via `[Annotation]` objects at build time.
- **The builder** (`DistributedApplicationBuilder`, `DistributedApplication`) wraps a .NET `HostApplicationBuilder`, so the app host gets configuration, DI, and environment for free, then layers on `Resources`, `Eventing`, and a `Pipeline`.
- **Local orchestration via DCP.** At run time, `Dcp/DcpExecutor.cs` translates the model into Kubernetes custom resources — `Container`, `Executable`, `Service`, `Endpoint`, `ContainerNetwork` (`src/Aspire.Hosting/Dcp/Model/`) — and drives the **Developer Control Plane**, a separate Go binary (`microsoft/dcp`), through the standard `k8s` client library. DCP is a *local* Kubernetes-API-compatible orchestrator: Aspire ships a k8s API server on your laptop rather than requiring a cluster.
- **Observability is a first-class side effect.** `docs/open-telemetry-architecture.md` spells it out: the dashboard hosts an in-memory OTLP server, and the app host injects `OTEL_SERVICE_NAME` / `OTEL_RESOURCE_ATTRIBUTES` / `OTEL_EXPORTER_OTLP_ENDPOINT` into every resource. No per-service telemetry wiring.
- **Publishing as a pipeline.** `src/Aspire.Hosting/Pipelines/` reframes deployment as a sequence of `PipelineStep`s. The manifest publisher (`Publishing/ManifestPublisher.cs`) is now marked `[Obsolete]` — superseded by a `publish-manifest` pipeline step — and there are Docker Compose (`Aspire.Hosting.Docker`) and Kubernetes/Helm (`Aspire.Hosting.Kubernetes`) publishers. The manifest JSON schema (`src/Schema/aspire-8.0.json`) is the interchange contract with Azure Developer CLI.

## Key techniques

- **Annotations over inheritance.** The resource model uses `ResourceAnnotationCollection` backed by an `ImmutableArray` with lock-free reads and locked writes — engineered for a workload where LINQ queries vastly outnumber mutations (documented in its own comments). Capability interfaces are just type-safe projections of annotations.
- **ATS — the Aspire Type System.** This is the non-obvious gem. C# methods and types carry `[AspireExport]`, `[AspireDto]`, `[AspireValue]`, `[AspireUnion]` attributes; type IDs are derived as `{AssemblyName}/{TypeName}` and capability IDs as `{AssemblyName}/{camelCaseMethodName}` (see `AspireExportAttribute.cs`). Assembly scanning (`Aspire.TypeSystem`) then generates *guest SDKs* in TypeScript, Python, Go, Java, and Rust. **Handle types** (builder, resource, endpoint) aren't reimplemented per language — they become thin wrappers that hold opaque handles and proxy every call back to the .NET runtime over **JSON-RPC on a Unix-domain socket** (`Backchannel/BackchannelService.cs`, using `StreamJsonRpc`). So there is exactly one engine; polyglot support is code generation, not ports.
- **Env-var injection as the telemetry contract.** Rather than a client SDK, Aspire configures OTel by setting standard `OTEL_*` environment variables, which *any* OTel-aware app (Dapr, Go, whatever) honors. The transport is the standard, not a proprietary agent.
- **Agent-aware CLI.** `Aspire.Cli/Agents/` is a genuine surprise in a dev-tool repo: it scans the user's environment and installs Aspire "skills" bundles for Claude Code, OpenCode, Copilot CLI, VS Code, and Playwright, including a GitHub-artifact attestation verifier for the skill bundles.

## Design decisions

- **One definition, two runtimes.** The trade-off is central: the same model must run *locally* (fast, debuggable, per-developer) and *deploy* (compose/K8s/Azure). Aspire pays for this with two execution paths — DCP for `run`, publishers for `publish` — and keeps them honest by routing both through the same `DistributedApplicationModel`.
- **C# as the source of truth, everything else generated.** This optimizes for correctness (one runtime to fix) over per-language idiomatic freedom. The cost is that non-C# users hit the RPC boundary and its serialization constraints — a real design bet, not a convenience.
- **Correctness-as-structural in the annotation store.** The thread-safe collection exists because multiple pipelines mutate resources concurrently during build/publish; the authors chose an immutable-snapshot store with index-clamping rather than a lock around every read. Reads (LINQ over annotations) stay lock-free; writes are rare and O(n).
- **Convention over configuration, .NET-style.** Telemetry, service discovery, and connection strings are wired by default; opting out is explicit. This is the same bet `asp.net` Core made, applied to distributed apps.

## Comparison notes

- **vs. Docker Compose.** Compose describes *containers*; Aspire describes *an application* — projects and executables alongside containers, with typed references and wait-for dependencies. Compose has no app model to publish; Aspire's model *targets* compose (and K8s) as one publisher among several. The closest spiritual relative is **Dapr**, which shares the "sidecar + building-block" instinct but adds runtime services where Aspire adds build-time code generation.
- **vs. the agent-orchestration tooling in this wiki** ([[Distributed Systems]], [[Agent Orchestration]]): Aspire is about *microservice* composition, but it's the reference point for the observability those systems conspicuously lack — it makes [[The Three Pillars of Observability|logs/metrics/traces]] a default of the app model rather than a bolt-on, and its env-var injection is exactly the "distributed tracing without per-service wiring" the wiki keeps flagging as missing.
- **vs. [[Microservices for the Benefits, Not the Hustle]]:** Aspire is the tooling that lowers the *cost* of Wolf's "changeability" argument — spinning up a boundary is a builder call — but it also makes the nanoservices risk more acute by making new services nearly free.

Tags: #tool #project #dotnet #distributed-systems #observability #devtools

## Related

- [[Distributed Systems]] — the observability gap Aspire fills for microservices
- [[The Three Pillars of Observability]] — the three signals Aspire makes default
- [[Microservices for the Benefits, Not the Hustle]] — the changeability thesis Aspire operationalizes
- [[Semantic Kernel]] — Microsoft's other .NET SDK; same ecosystem, agent middleware rather than app toolchain
- [[WSL Containers]] — sibling Microsoft container tooling

---
*Sources: [[raw/aspire]], [[summary/aspire]]*
*Last updated: 2026-08-26*
