---
url: https://github.com/microsoft/aspire
title: Aspire
author: Microsoft
site: github.com/microsoft/aspire
date_fetched: 2026-08-26
date_published: 2024-05-21
---

# Aspire

Microsoft's code-first, multi-language toolchain for building, running, and deploying distributed applications. The repo is the .NET Aspire monorepo: the AppHost SDK, the new `aspire` CLI, the observability dashboard, service-discovery infrastructure, project templates, integrations, and a VS Code extension. Version in `eng/Versions.props` is **13.6.0**, built against the .NET 10 SDK (`global.json` → `10.0.400`).

## Core Premise

You describe how services, frontends, containers, databases, caches, and their connections fit together in code — a single "app host" program — and the same definition runs the whole app locally *and* carries into deployment. The canonical language is C#, but the README shows the identical definition in TypeScript, and the `Aspire.Hosting.CodeGeneration.*` projects emit Go, Java, Python, and Rust SDKs too.

## How it works

- **App model.** The `DistributedApplicationBuilder` fluent API composes resources (`AddRedis`, `AddProject`, `AddNodeApp`). Resources are thin named entities — `IResource` is just `Name` + an `Annotations` collection — and capability interfaces (`IResourceWithEndpoints`, `IResourceWithConnectionString`, …) are derived from annotations rather than a class hierarchy.
- **Local orchestration.** At run time the app host translates the model into Kubernetes-style custom resources (`Container`, `Executable`, `Service`, `Endpoint`, …) and drives the **Developer Control Plane (DCP)** — a separate Go binary (`microsoft/dcp`) that exposes a Kubernetes API — via the standard `k8s` client.
- **Observability by default.** The dashboard hosts an in-memory OTLP server; the app host injects `OTEL_SERVICE_NAME`, `OTEL_RESOURCE_ATTRIBUTES`, and `OTEL_EXPORTER_OTLP_ENDPOINT` into every resource, so traces/metrics/logs "just work" locally (documented in `docs/open-telemetry-architecture.md`).
- **Deployment.** Publishing is a pipeline of steps (`DistributedApplicationPipeline` / `PipelineStep`). The manifest JSON (`src/Schema/aspire-8.0.json`) is the interchange format with Azure Developer CLI, and there are Docker Compose and Kubernetes/Helm publishers.
- **Polyglot via code generation.** The Aspire Type System ("ATS") marks C# capabilities with `[AspireExport]` / `[AspireDto]` / `[AspireValue]` attributes, scans assemblies, and generates guest SDKs. Handle types proxy back to the .NET runtime over JSON-RPC on a Unix-domain socket ("Backchannel"), so the C# runtime stays the single engine.
- **CLI.** `Aspire.Cli` is a large System.CommandLine + Spectre.Console CLI with a TUI, scaffolding, acquisition, secrets, migrations, and — notably for agent workflows — a whole `Agents/` subsystem that installs Aspire "skills" bundles and integrates with Claude Code, OpenCode, Copilot CLI, VS Code, and Playwright.
