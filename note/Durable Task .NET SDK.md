# Durable Task .NET SDK

Microsoft's durable-execution SDK for .NET: the library behind Durable Functions for the .NET isolated worker and the front end to the managed Durable Task Scheduler. Orchestrations are ordinary `async` C# methods that survive crashes by deterministically replaying from a persisted event history, with the heavy state kept in a backend reached over gRPC. It matters to this wiki because durable execution is the same primitive the agent-harness world keeps reinventing — and because it shows what a mature, version-aware control plane for long-running async work looks like.

---

## Architecture

~40K lines of C# across ten NuGet packages with strict layering (in `src/`):

- **Abstractions** (`src/Abstractions/`) — zero-dependency core types: `TaskOrchestrator<TInput,TOutput>`, `TaskActivity<TInput,TOutput>`, `TaskEntity<TState>`, the abstract `TaskOrchestrationContext`, and `DurableTaskRegistry` (split across `DurableTaskRegistry.Orchestrators.cs` / `.Activities.cs` / `.Entities.cs`) which maps `(name, version)` keys to factories.
- **Client.Core / Worker.Core** — the client (`DurableTaskClient` for starting, querying, suspending, purging instances) and the worker (`DurableTaskWorkerOptions`, `IDurableTaskFactory`, hosting glue that runs orchestration processing inside an `IHostedService`).
- **Transports** — `Client.Grpc`, `Worker.Grpc` (sidecar/DTS over gRPC) and `*.AzureManaged` packages for the Durable Task Scheduler. The worker consumes a server-streamed `GetWorkItems` RPC (`GrpcDurableTaskWorker.Processor.cs`) with reconnect/backoff logic in `WorkItemStreamConsumer.cs` and `GrpcBackoff.cs`.
- **Generators** (`src/Generators/DurableTaskSourceGenerator.cs`, ~1,300 lines) — a Roslyn source generator that scans `[DurableTask]`-attributed classes and emits registrations plus type-safe `CallSayHelloTypedAsync(...)` extension methods, replacing string-named activity invocation.

The wire protocol (`src/Grpc/orchestrator_service.proto`) is vendored from `durabletask-protobuf` and defines ~30 RPCs — start/wait/suspend/terminate/rewind instances, query/purge, entity signal/query, `StreamInstanceHistory`, and three "abandon" RPCs for redelivering work items.

## Key techniques

- **Replay-based durability.** Orchestrator code is deterministic and re-executed from the stored history; `TaskOrchestrationContext` exposes `IsReplaying` and a `CurrentUtcDateTime` that returns the value recorded in the history rather than the wall clock. Non-obvious craft: `ReplaySafeLoggerFactoryImpl` wraps `ILoggerFactory` so logs are suppressed during replay, with an unwrap-depth guard against wrapper-context double-wrapping (`TaskOrchestrationContext.cs`).
- **Deterministic GUIDs.** `NewGuid()` derives a name-based UUIDv5 from instance ID + orchestration time + sequence number — replay-safe randomness without persisting each value.
- **Version-aware dispatch.** `DurableTaskRegistry` keys orchestrators by `TaskVersionKey(name, version)`; worker-level `Versioning.MatchStrategy` gates which versions are accepted off the wire, and `ContinueAsNewOptions.NewVersion` migrates eternal orchestrations to a new implementation at a replay-safe boundary. The source generator emits diagnostics (`DURABLE3003/3004`) instead of generating colliding registrations.
- **Shim layer.** `Worker/Core/Shims/` adapts the typed SDK surface to the lower-level durabletask runtime interface (`TaskOrchestrationShim`, `TaskActivityShim`, `TaskEntityShim`, `JsonDataConverterShim`), letting the same user code run against the gRPC sidecar, the standalone scheduler, or an in-process test host (`src/InProcessTestHost`).
- **Extended sessions.** `ExtendedSessionsCache`/`ExtendedSessionState` keep orchestration state resident across work items so replay cost is amortized instead of paid per activity.

## Design decisions

The dominant trade-off is **determinism over expressiveness**: orchestrators may not do I/O, use the clock, or touch randomness — every side effect is pushed into activities, in exchange for crash-resilient arbitrary control flow (loops, fan-out, human-in-the-loop waits via `WaitForExternalEvent` with durable timers as timeouts). This is the event-sourcing renaissance approach — replay a deterministic function over a log — rather than checkpointing continuations.

The second trade-off is **backend as a service over embedded storage**: unlike its predecessor the Durable Task Framework (Azure Storage / MSSQL backends in-process), this SDK holds no storage provider — state lives in the sidecar or DTS. Simpler SDK, but the durability story is now someone else's deployment.

Finally, the payload advice is telling: inputs/outputs are serialized into the history, so the docs push blob-reference indirection for large data — the same trade-off every event-sourced system makes, and the source of the SDK's dedicated large-payload purge and tombstone machinery in `Client/Core`.

## Comparison notes

- [[All Your Agents Are Going Async]] argues agent sessions outlive HTTP connections and need durable state plus resumable operations; this SDK is the canonical .NET implementation of exactly that substrate — deterministic replays, durable timers, external events, and a streaming work queue — built a decade before agents made it fashionable.
- [[Cloud Agent Lessons from Cursor]] describes Cursor running 50M+ daily actions on Temporal for durable execution; Durable Task is Microsoft's answer to the same problem, differing mainly in that replay is priced into user code (determinism rules) rather than hidden behind a closed workflow engine.
- [[Agent Orchestration]] collects multi-agent topologies and schedulers; this repo supplies the single-orchestration building block — sub-orchestrations with independent histories, entities as single-threaded actors — that such control planes assume.
- [[Open Source Agent Toolkit 2026]] treats orchestration as an independent layer of the agent stack; durabletask-dotnet is a production-grade instance of that layer whose versioning and history-truncation (`ContinueAsNew`) features answer problems long-horizon agent harnesses (e.g. [[MiMo Code]]) are only now hitting.

---
*Sources: [[raw/durabletask-dotnet]], [[summary/durabletask-dotnet]]*
*Last updated: 2026-10-14*
