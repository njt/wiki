---
url: https://github.com/microsoft/durabletask-dotnet
title: "Durable Task .NET Client SDK"
author: Microsoft
date_fetched: 2026-10-14
date_published:
topics:
  - distributed-systems
  - developer-tools
---

Microsoft's Durable Task .NET SDK is a standalone .NET library for writing **durable orchestrations, activities, and entities** — workflows that survive process crashes by replaying from a persisted event history. It is the client/worker SDK behind Durable Functions for the .NET isolated worker and connects to a backend over gRPC: either a local "sidecar" process (the Azure Functions host) or the managed **Durable Task Scheduler** (DTS).

The repo is split into NuGet packages with deliberate layering: `Microsoft.DurableTask.Abstractions` (the core types — `TaskOrchestrator`, `TaskActivity`, `TaskEntity`, `TaskOrchestrationContext`, `DurableTaskRegistry`), separate `Client` and `Worker` cores, pluggable transports (`Client.Grpc`, `Worker.Grpc`, `Worker.AzureManaged`), and a Roslyn **source generator** that turns class-based `[DurableTask]` orchestrators/activities into registrations plus type-safe `CallXxxAsync` extension methods. The wire protocol is a vendored `orchestrator_service.proto` (~30 RPCs including streaming `GetWorkItems`, `CompleteOrchestratorTask`, entity signal/query, and history streaming) shared across the durabletask family.

Programmer-facing model: orchestrators are deterministic, replayable functions that only call `context` APIs (activities, durable timers, external events, sub-orchestrations, `ContinueAsNew`); activities do arbitrary I/O with at-least-once semantics; entities are stateful single-threaded virtual actors. The README's more novel material covers **orchestration versioning** — multiple `[DurableTask(Version = "...")]` classes under one name, worker-level match strategies, and `ContinueAsNew(NewVersion)` to migrate long-running instances — and detailed Azure Government token-audience handling for DTS authentication.
