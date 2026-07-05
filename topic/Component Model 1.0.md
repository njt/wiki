# Component Model 1.0

The Bytecode Alliance's roadmap for taking WebAssembly's Component Model from its current preview state (P3 nearing release) to a stable, formally specified 1.0. Luke Wagner and Alex Crichton laid out five work areas at the February 2026 Plumbers Summit: ABI modernization, native browser implementation, spec simplification, ecosystem growth, and WIT expressivity gaps. The through-line is making the Component Model as easy to implement as WASI P1 while keeping the backwards-compatibility story that's held since P1.

---

## Key Quotes

> "We can start using this stuff now."
> — Luke Wagner

The Component Model and WASI are already in production. P1 modules and P2 components still work. The stability guarantees aren't aspirational — they're maintained via semver, side-by-side implementations, and Wasm-to-Wasm adapters.

> The current ABI creates friction: heap fragmentation over time, difficulty handling large allocation failures gracefully, one host-to-guest call per value in a list, and trouble using custom memory allocators.

The `cabi_realloc`-based ABI is the biggest compatibility burden. The lazy ABI flip — callee returns opaque handles, caller places values via static inlinable built-ins — is the kind of design move that only becomes obvious after years of production use. It also enables zero-copy forwarding and dropping unused values, which are genuinely new capabilities, not just fixes.

> The Component Model can't formally reach 1.0 without native implementation in at least two browser engines.

This is the hard political constraint. `jco`'s transpile-to-JS-glue works today, but the `"use components"` telemetry strategy (borrowed directly from asm.js → WebAssembly) is the long game. Mozilla's showing perf numbers; Chrome V8 opened an eval issue. Neither is a commitment, but both are more signal than WebAssembly had at this stage.

> synchronous adapters allocate a lightweight task on the stack that compilers like Cranelift can largely optimize away

The 3.5x overhead on synchronous cross-component calls is a real papercut. The fix — stack-allocated lightweight tasks that Cranelift can see through — is the right shape of solution: make the zero-cost path actually zero-cost by giving the compiler enough information to elide the abstraction.

> The Component Model 1.0 spec will only include the "good parts" of the P3 spec, rather than being a superset of what exists right now.

Refreshingly honest. Most spec efforts add; this one explicitly subtracts. The `cabi_realloc` ABI, the recursion check (already removed), and other accumulated complexity get dropped. Existing P3 components keep working through adapters.

---

## Key Themes

- **#concept** The microkernel analogy: Component Model as always-present kernel, WASI as optional OS services. Explains why browsers can implement the Component Model natively (no I/O) while WASI needs polyfills.
- **#pattern** The `"use asm"` → `"use components"` telemetry strategy. Ship a no-op string, let browser vendors collect usage data, build the case for native implementation with evidence rather than advocacy.
- **#pattern** Lazy ABI with static inlinable built-ins. Control-flow inversion that a compiler can see through — the same trick behind zero-cost futures in Rust.
- **#tool** `lower-components` as the "component smasher": flatten a compound component into a single core Wasm module using multi-memory. Makes implementing the Component Model roughly as hard as implementing WASI P1.
- **#tool** Record/replay debugging via WAVE format. Instrumentation components wrap imports/exports, record at WIT level, replay without the original host. Editable, human-writable traces. Only possible because of shared-nothing component isolation.
- **#person** Luke Wagner, Alex Crichton, Nick Fitzgerald, Ryan Hunt, Sy Brand, Yan Chen — the core Bytecode Alliance technical leadership.

---

## Critical Analysis

**The ABI transition is the right kind of hard.** Most projects would ship 1.0 with the existing ABI and call it done. The Bytecode Alliance is instead doing the messy work of changing the calling convention in a minor release (0.3.x), shipping both old and new side-by-side, then flipping the default at 1.0. This is SQLite-level conservatism about compatibility, and it's the right call for something that wants to be infrastructure.

**The browser requirement is both constraint and forcing function.** Requiring two browser implementations before 1.0 means the spec can't just be Wasmtime's internal API dressed up as a standard. It forces the spec to be implementable by independent teams with different constraints — which is exactly what made WebAssembly itself succeed. The `jco` telemetry play is smart, but the timeline is still measured in years, not months.

**The "good parts" approach to the 1.0 spec is underrated.** Most standards efforts suffer from the "everything we built must ship" pathology. Explicitly dropping features that didn't earn their complexity — and providing adapter tools for backwards compat — is the sign of a team that's been burned by spec bloat before.

**WIT expressivity gaps reveal a language design problem masquerading as a feature list.** Optional imports, callbacks, resource subtyping, function subtyping — these are things every interface definition language eventually needs. WIT is young, and these gaps are normal, but the sequencing decisions (what makes 1.0 vs 1.x) will shape what kinds of components people can build for years.

**Cooperative threads below WASI is architecturally clean but coordination-heavy.** Putting threads at the Component Model level rather than WASI means every language's `std::thread::spawn` just works. But the toolchain dependency chain (wasi-libc → LLVM → Wasmtime → wit-bindgen → language ecosystems) means a feature can be "done" at one level and still months from reaching developers. LLVM's 6-month release cycle is the long pole.

---

*Sources: [[summary/the-road-to-component-model-1-0]]*
*Last updated: 2026-06-11*
