---
url: https://www.nuget.org/packages/Celly
title: "Celly — Native C#/.NET Implementation of Common Expression Language (CEL)"
author: bsidio
date_fetched: 2026-07-18
date_published: 2026-07-17
---

Celly is a pure managed C# implementation of Google's Common Expression
Language (CEL), written from scratch with no WASM shims, Go-compiled
artifacts, or native bindings. It targets .NET 8.0+ and is released under
Apache-2.0 at version 1.2.0.

It achieves 100% conformance with the official cel-spec suite (2,456/2,456
tests), verified in CI on Linux, Windows, and macOS. Benchmarking places it as
the fastest .NET CEL implementation — ~1.2× faster than Cel.NET, ~6–9× faster
than TELUS's Cel package, and within ~1.2× of the reference Go implementation
(and faster than cel-go on comprehension-heavy expressions).

The package ecosystem spans three packages: the zero-dependency core (Celly),
Protobuf integration (Celly.Protobuf), and protovalidate for .NET
(Celly.Protovalidate), which itself passes 100% of buf's protovalidate
conformance suite. The core includes a full standard library with CEL-semantic
arithmetic, RE2 matches via .NET's NonBacktracking engine, macros, gradual
typing with dyn, optional chaining, extension libraries, first-class ASTs, and
untrusted-input safety through evaluation budgets and static cost estimation.
CelEnv and CelProgram are immutable and safe for concurrent evaluation.

CEL is the expression language behind Kubernetes ValidatingAdmissionPolicy,
Envoy RBAC, Google Cloud IAM conditions, and gRPC protovalidate. Official
implementations exist for Go, C++, Java, and Rust; Celly fills the .NET gap.
