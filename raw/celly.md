---
url: https://www.nuget.org/packages/Celly
title: Celly — Native C#/.NET Implementation of Common Expression Language (CEL)
author: bsidio
date_fetched: 2026-07-18
date_published: 2026-07-17
---

# Celly

A native C#/.NET implementation of Google's Common Expression Language (CEL). Written from scratch in pure managed C# — no WASM shims, no Go-compiled artifacts, no native library bindings.

**Version:** 1.2.0 (2026-07-18) | **License:** Apache-2.0 | **Target:** .NET 8.0+
**Repository:** https://github.com/bsidio/celly
**Documentation:** https://bsid.io/celly/

## Status

1.0 stable — public API with snapshot testing, 100% conformance, and fuzz-hardened.

## Conformance

Passes **2,456 / 2,456 (100%)** of the official cel-spec conformance suite, spanning 30 test data files pinned at commit `59505c1`, verified in CI across Linux, Windows, and macOS. A ratcheting skip-list (`known-failures.txt`) is now empty.

The conformance suite also runs `strong_*` enum sections with Celly's strong-enum mode.

## Performance

Described as "the fastest .NET CEL implementation":
- ~1.2× faster than Cel.NET
- ~6–9× faster than TELUS's `Cel` package
- Far lower memory allocation than competitors
- Within ~1.2× of the reference Go implementation (cel-go) on simple expressions
- **Faster** than cel-go on comprehension-heavy expressions

## Package Ecosystem (3 packages)

| Package | Contents | Key Dependency |
|---------|----------|---------------|
| **Celly** | Core: lexer, parser, macros, type checker, evaluator, standard + extension libraries | **None** |
| **Celly.Protobuf** | Protobuf types, well-known types, enums, proto2 extensions | `Celly`, `Google.Protobuf` |
| **Celly.Protovalidate** | protovalidate for .NET — validates messages against `buf.validate` rules | `Celly`, `Celly.Protobuf`, `Google.Protobuf` |

Celly.Protovalidate passes **2,872 / 2,872 (100%)** of buf's official protovalidate conformance suite.

## Architecture

Zero-dependency core containing the lexer, parser, macros, type checker, evaluator, standard library, and all extension libraries.

## Feature Checklist

- **Full standard library**: CEL-semantics operators (checked int64/uint64 arithmetic, cross-type numeric comparisons, IEEE doubles), `size`/`in`/indexing, all type conversions with Go-compatible formatting, RE2-semantics `matches()` via .NET's linear-time NonBacktracking engine, nanosecond-precision timestamps/durations with IANA timezone accessors
- **Macros**: `has`, `all`, `exists`, `exists_one`, `map`, `filter` plus two-variable comprehensions
- **Type checker**: Gradual typing with `dyn`, parameterized-type unification, container (`C.name`) resolution, comprehension-variable shadowing per spec
- **Optionals**: Full optional chaining with `optional.of/ofNonZeroValue/none`, `a.?b`, `a[?b]`, `[?e]`, `{?k: v}`, `orValue`, `optMap`/`optFlatMap`
- **Extension libraries (opt-in)**: Strings, Math, Encoders, Bindings, Block, Two-var comprehensions, Protos, Network
- **Protobuf integration**: Message construction, WKT collapse, Any pack/unpack, strong-enum mode
- **Plain .NET types** via `NativeTypeProvider`
- **First-class ASTs** with traversal/inspection tools
- **Untrusted-input safety**: Runtime evaluation budget (`EvalLimits`) and static cost estimation (`CelEnv.EstimateCost`)
- **Thread safety**: `CelEnv` and `CelProgram` are immutable; programs are safe for concurrent evaluation

## Usage Example

```csharp
var env = CelEnv.Create(new CelEnvSettings
{
    Declarations =
    [
        new VariableDecl("request", CelType.Map(CelType.String, CelType.Dyn)),
    ],
});
var program = env.Compile("request.user.startsWith('admin-') && size(request.groups) > 0");
var result = program.Eval(new Dictionary<string, object?>
{
    ["request"] = new Dictionary<string, object?>
    {
        ["user"] = "admin-alice",
        ["groups"] = new[] { "ops" },
    },
});
```

## Why Celly

CEL is the expression language behind Kubernetes admission policies (ValidatingAdmissionPolicy), Envoy RBAC, Google Cloud IAM conditions, gRPC protovalidate, and more. Official implementations exist for Go, C++, Java, and Rust but not .NET — which Celly fills.

## Total Downloads: 174 (across 6 versions)

Versions: 0.1.0 (37), 0.2.0 (35), 0.3.0 (37), 1.0.0 (31), 1.1.0 (34), 1.2.0 (0)

## NuGet Packages Depending on Celly
1. Celly.Protobuf (145 downloads)
2. Celly.Protovalidate (0 downloads)
