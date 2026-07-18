# Celly — Native .NET CEL Implementation

A pure C# implementation of Google's Common Expression Language (CEL) that fills the .NET gap in the CEL ecosystem. Zero dependencies, 100% spec conformance, and faster than the Go reference on comprehension-heavy workloads.

---

CEL is the expression language behind Kubernetes admission policies, Envoy RBAC, Google Cloud IAM conditions, and gRPC protovalidate. Google ships official implementations in Go, C++, Java, and Rust — but not .NET. Celly closes that gap with a from-scratch managed C# implementation that passes every one of the 2,456 cel-spec conformance tests.

## The Conformance Story

> "2,456 / 2,456 (100%)" of the official cel-spec conformance suite, verified in CI on Linux, Windows, and macOS.

This isn't a hand-wavy "compatible with CEL" claim. The author pinned the upstream spec at a specific commit, built a ratcheting skip-list (`known-failures.txt`) that would break CI if a listed test started passing (forcing the list to be updated), and drove that list to zero. That's the kind of rigor that earns trust — and it's rare in the .NET ecosystem, where "good enough" ports are the norm.

The protovalidate package takes the same approach, passing 2,872/2,872 of buf's official conformance suite. Three packages, two independent conformance gauntlets, both at 100%.

## Performance Claims

> "~1.2× faster than Cel.NET and ~6–9× faster than TELUS `Cel`."

The bench is against the two existing .NET CEL packages. More interesting is the comparison to cel-go: within ~1.2× on simple expressions, and *faster* on comprehension-heavy ones. A managed-language implementation beating the reference Go implementation on any workload is a genuine engineering achievement. The secret is likely .NET's JIT + the author's attention to allocation patterns — the README explicitly calls out "far lower memory allocation than competitors."

## Architecture Decisions Worth Noting

**Zero dependencies in the core package.** `Celly` itself has no NuGet dependencies — the lexer, parser, macros, type checker, evaluator, and standard library are all hand-written. Protobuf support is cleanly separated into `Celly.Protobuf`, which depends on `Google.Protobuf`. This means you can use CEL expressions against plain .NET objects without dragging in the protobuf stack.

**Immutable, thread-safe by construction.** `CelEnv` and `CelProgram` are immutable. Once compiled, a program can be evaluated concurrently across threads with no locking. This is the right design for server workloads where expression evaluation happens on the hot path.

**Extension libraries are compile-time-only cost.** All nine extension libraries (Strings, Math, Encoders, Bindings, Block, Two-var comprehensions, Protos, Network, Optionals) can be enabled without per-evaluation overhead — library loading is a one-time compile step.

**Strong-enum mode.** Celly offers a strong-enum mode where enums are distinct named types, something the reference Go implementation doesn't provide. A rare case of a port adding capability rather than just replicating.

**First-class ASTs with round-tripping.** `AstTools` supports lossless conversion to/from canonical `cel.expr` protos, plus a precedence-aware unparser that restores macros to their original syntax. This matters for caching compiled expressions and cross-implementation interop.

## What CEL Actually Is (and Why You'd Want It)

CEL is a small expression language designed for policy and configuration — it's what lets you write `request.auth.claims.group == 'admin'` in a Kubernetes ValidatingAdmissionPolicy and have it evaluated safely against untrusted input. It's not Turing-complete, it has a type checker, and it guarantees termination. These properties make it the standard for policy evaluation in Google's infrastructure stack.

The .NET ecosystem has historically used home-grown expression evaluators or embedded scripting languages (Lua, JavaScript via ClearScript) for these use cases. Celly offers a standards-based alternative that interoperates with the broader CEL tooling ecosystem.

## Critical Assessment

**The good:** Conformance rigor, clean architecture, zero-dependency core, and a thoughtful three-package split. The author clearly understands both CEL semantics and .NET performance engineering. The strong-enum mode and NativeTypeProvider show ambition beyond a straight port.

**The gaps:** Total downloads (174) are tiny — this is brand new, and version 1.0 shipped literally yesterday (2026-07-17). There's no community, no third-party production usage reports, and the author (bsidio) is a single maintainer. The documentation site (bsid.io/celly) is referenced but unverified. The performance claims are self-reported against unnamed benchmark methodology.

**The bet:** If you're building a .NET service that needs CEL — for policy evaluation, feature flags with complex conditions, or protovalidate integration — this is the clear choice. The conformance story alone justifies the dependency over rolling your own or using a less-complete alternative. But you're betting on a solo maintainer's continued investment.

**For agentic systems:** CEL is the expression language that could let coding agents safely generate and evaluate policy rules. An agent writing `request.region in ['us-east', 'eu-west'] && request.tier >= 3` is far safer than one generating arbitrary C# or JavaScript. Celly makes that pattern available in .NET. Combined with the protovalidate package, it's a building block for agent-generated validation rules with deterministic, verifiable semantics.

## Related

- [[CQRS Pattern in C# and Clean Architecture]] — another .NET ecosystem pattern
- [[OpenMono Agent]] — .NET coding agent that could leverage CEL for policy evaluation
- [[Accordant]] — Microsoft's model-based testing for .NET; shares the "spec as executable artifact" philosophy
- [[Security and Sandboxing]] — CEL's termination guarantees are a sandboxing primitive

---
*Sources: [[raw/celly]]*
*Last updated: 2026-07-18*
