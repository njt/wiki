---
url: https://bytecodealliance.org/articles/the-road-to-component-model-1-0
title: "The Road to Component Model 1.0"
author: Eric Gregory
date_fetched: 2026-06-11
date_published: 2026-06-08
---

# The Road to Component Model 1.0

Eric Gregory, Bytecode Alliance, June 8, 2026.

## Summary

WASI P3 is nearly here, bringing native async to WASI and the Component Model. The article looks ahead to the next major milestone: a stable, formally specified Component Model 1.0. At the Bytecode Alliance Plumbers Summit in February, Luke Wagner and Alex Crichton previewed the path to a stable 1.0, and Luke expanded on that vision at Wasm I/O 2026 in Barcelona in March.

## The Component Model and WASI

The Component Model is the specification layered atop core WebAssembly that defines how Wasm binaries bundle, link, and communicate — covering the type system, binary format, IDL, and calling conventions for passing typed values across component isolation boundaries. WASI is a standardized set of APIs for accessing system resources (files, sockets, clocks, randomness) in a portable, capability-secure way.

Luke analogized the relationship to a microkernel architecture: the Component Model is the always-present microkernel providing foundational primitives that run across any host, while WASI layers on top like OS services (networking, storage, graphics) that run as processes on top of the microkernel and may or may not be present on a given device.

Both WASI and the Component Model are already heavily used in production. P1 modules and P2 components still work. Stability has been maintained since P1 using semantic versioning, side-by-side implementations, and Wasm-to-Wasm adapters.

## Five Areas of Work

### 1. ABI Improvements

The current ABI relies on `cabi_realloc`. A planned lazy ABI inverts control flow: callee returns lazy value handles (opaque i32 indices), and the caller calls static built-in functions to place values into memory. Benefits: zero-copy forwarding, unused values can be dropped, and string transcoding logic moves from engine to guest code. Non-breaking transition via opt-in in 0.3.x.

Also: multivalue returns (blocked on LLVM C ABI support), error context values, and a GC ABI option (Nick Fitzgerald's pre-proposal in Component Model issue 525).

Related performance goal: zero overhead on synchronous calls between components. Currently ~3.5x overhead on sync paths due to async task infrastructure. Plan: refactor task state so sync adapters allocate lightweight tasks on the stack that Cranelift can optimize away.

### 2. The Browser Path

Can't reach 1.0 without native implementation in at least two browser engines. jco's transpile command already converts components to core Wasm + JS glue. jco now emits a `"use components"` identifier (jco 1.16.8) for browser feature-usage telemetry — same strategy as asm.js's `"use asm"` that led to WebAssembly.

Mozilla presented perf results at a recent CG meeting (Ryan Hunt's blog post on Component Model work). Chrome V8 opened an issue for evaluating Component Model implementation. Not commitments, but meaningful signals.

DOM mutation-heavy Wasm VDOM reconciliation can get close to 2x speedup from direct Wasm-to-browser API calls.

### 3. Making the Component Model Easier to Implement

Two parts:
- **Simplify the spec**: 1.0 spec only includes "good parts" of P3, not a superset. ABI change means implementers don't need to support cabi_realloc.
- **C ABIs**: Guest C-ABI lets toolchains target Component Model by compiling to core Wasm + calling imports via generated C headers, then wrapping with `wasm-component-ld`. Host C-ABI lets runtimes implement Component Model support without re-implementing the full spec. Proposed `lower-components` tool would "smash" compound components into single core Wasm modules using multi-memory.

Goal: implementing WASI + Component Model should be about as easy as implementing WASI P1.

### 4. Growing the Ecosystem

- More documentation (Component Model book, Danilo Chiarlone's Server-Side WebAssembly)
- Stable P3 support in Rust, Tokio, LLVM, CPython (tracked in awesome-wasm-components)
- Cross-language tooling: jco, wac, wkg
- Record/replay debugging: Yan Chen demoed instrumentation components recording WIT-level calls in WAVE format, replayable without original host. Traces are editable and human-writable.

### 5. Closing Expressivity Gaps in WIT

- **Optional imports**: declare capability as optional (standard libraries against minimal host world)
- **Callbacks**: needed for DOM APIs like addEventListener
- **Resource subtyping**: e.g. DOM Node extends EventTarget
- **Function subtyping**: add params/fields/variants without breaking changes
- **Enhanced import/export names**: multiple of same WIT interface with different identifiers
- **Getters and setters**: syntactic sugar for idiomatic bindings
- **Map type**: idiomatic objects/dictionaries across languages (Yordis Prieto working on this)
- **Runtime instantiation**: instantiate components at runtime for lazy loading, subcommands

Not all will land before 1.0; some in 1.x.

## Cooperative Threads and Stream Splicing

Sy Brand presented cooperative threads at the Plumbers Summit. Implemented at Component Model level below WASI. Rust `std::thread::spawn` gets cooperative thread support with no WIT changes. wasi-libc pthreads largely done, LLVM patches landed (next release), Wasmtime implementation under feature flag. Ships in follow-up WASI P3 minor release.

Stream splicing also shipping in early P3 follow-up: splice one stream directly into another without intermediate copy.

## The Toolchain View

Alex Crichton's pipeline: Component Model spec → wasm-tools (binary format, validation) → Wasmtime (runtime semantics) → wit-bindgen (guest language bindings) → guest libraries and toolchains. Each stage provides feedback refining the previous one.

Bytecode Alliance controls much but not all: Rust (~9 weeks to stable), LLVM (6-month releases), CPython have their own cadences.

## Getting Involved

- Spec discussion: Component Model repo
- Implementation: wasm-tools, Wasmtime, wit-bindgen
- Language toolchains: componentize-go, componentize-py, componentize-js
- Browser: use jco-transpiled components (telemetry signal)
- Discussion: Bytecode Alliance Zulip
