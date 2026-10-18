# MXC — Microsoft eXecution Container

MXC is Microsoft's MIT-licensed sandboxed code execution system for running untrusted code (model output, plugins, tools) on Windows, Linux, and macOS. Unlike cloud sandbox products, it is a **library, not a service**: an SDK (Rust, .NET, Node) you link into your own application, which launches the workload in a platform-appropriate containment backend selected from a lattice of nine backends behind one versioned containment contract. It matters because it is Microsoft productizing, in the open, exactly the containment layer every agent harness has been hand-rolling — and because its policy-authoring tooling (audit mode) attacks the real bottleneck, which is not enforcement but figuring out what policy to write.

---

## Architecture

A Rust workspace (`src/Cargo.toml`) with three layers:

- **`src/mxc-sdk`** — the public library. `core/mxc_engine/` holds the dispatch brain: `dispatch.rs` maps a `ContainmentBackend` enum to a `spawn_runner`/`spawn_pty_runner` call; `backend_registry.rs` is a static table of ten `BackendRegistration` entries marking which backends are experimental and whether host detection covers them — with an invariant test forcing every new backend to declare its probe coverage before it can reach the enum. `run.rs` logs a `mxc.PolicyHash` audit event *before* any backend engages, anchoring each run to the canonical hash of its effective policy.
- **`src/mxc-sdk/src/backends/*`** — one module per backend: `process_container` (Windows), `wslc`, `windows_sandbox`, `isolation_session` (all Windows), `bubblewrap`, `lxc`, `nanvix` (Linux micro-VM), `seatbelt` (macOS), plus `hyperlight`. The macOS module is instructive: `profile_builder.rs` and `seatbelt_policy.rs` are deliberately platform-agnostic pure string/policy generation "so the rules are unit-tested everywhere even though only macOS can execute them."
- **`src/tools/wxc`** — the standalone executor CLI (`wxc-exec.exe`, 3,191 lines in `main.rs`) accepting JSON configs; **`src/host/plm`** — the Windows-only Permissive Learning Mode WPR-trace helper behind `--audit`.

Node and .NET SDKs (`sdk/node`, `sdk/dotnet`) wrap the Rust core via FFI (`src/ffi/mxc_ffi`); the Rust crate compiles the engine and selected backends into the consuming application. Versioned JSON schemas under `schemas/stable/` (0.4.0-alpha through 1.0.0) define the wire contract, with conformance tests (`mxc_contract_v1_0_0.rs`, `mxc_contract_v0_9_0_alpha.rs`) pinning behavior per version.

## Key techniques

- **Three-tier Windows fallback.** `backends/process_container/common/dispatcher.rs` documents the fallback ladder: Tier 1 BaseContainer (PSEC-enforced `deniedPaths`), Tier 2 AppContainer + BFS, Tier 3 AppContainer + DACL. Because BFS cannot express deny for paths inside an allowed parent, Tier 2 gets deny ACEs stamped on the host filesystem by a `DaclManager` whose `Drop` replays the ACE removals — and the docs warn that dropping it early yanks filesystem access mid-run. Cost is honestly quantified: tens of milliseconds at the typical 6–12 paths.
- **Deny-by-default Seatbelt profiles.** The generated profile is `(deny default)`, then baseline allows needed for any process to start, then policy-derived allows, then policy-derived denies **last so they win** — an explicit precedence ordering for `deniedPaths` over broader read grants.
- **Policy discovery over policy authoring.** `policy.rs` ships helpers that enumerate the host (tool/SDK env vars like `PYTHONPATH`, `CARGO_HOME`, `DOTNET_ROOT`, user profile, temp dirs) into filesystem-policy fragments, so a policy can be *assembled* from discovery rather than typed from memory.
- **Audit mode = learning mode.** `plm` runs a UAC-elevated retained child that captures WPR ETL events for every access *would have been denied*, decodes them through a shared `EtlDenialAnalyzer`, and emits `denials.json` plus an adjusted config — closing the loop from "my app won't run in the sandbox" to a working policy.
- **State-aware lifecycle.** `core/mxc_engine/state_aware.rs` drives provision/start/execute/stop/deprovision phases with per-phase telemetry correlation vectors minted at provision and recalled thereafter.
- **Nanvix micro-VM backend** (`backends/nanvix/runner.rs`): CPython 3.12 in a micro-VM with a trimmed FAT32 stdlib; host networking hard-off unless the full allow posture is present, and mixed postures rejected before execution rather than partially enforced.

## Design decisions

The central bet is **one contract, many enforcers**: a single `ContainerRequest` schema maps onto radically different OS mechanisms (AppContainer SIDs, Seatbelt profiles, Bubblewrap flags, LXC, micro-VMs), sacrificing per-backend expressiveness for portability — the README's backend table shows macOS getting only Seatbelt, and several backends returning `unsupported_containment` for streaming or PTY. Odd capabilities are handled honestly (WSLC has no stdin API, so `take_stdin` returns `None`). The second bet is that the hard problem is **policy tuning, not enforcement** — hence audit mode, deny-capture diagnostics, and policy discovery, all of which assume developers will iterate from "access denied" to a tuned policy. The third is API stability as a product feature: contract-versioned tests and schema versioning treat the sandbox request format like a protocol, not a config file.

## Comparison notes

- The backend lattice makes MXC the industrial-strength counterpart to [[Sandboxing with Minimal Effort (Inko)]] — Peterse's argument that a security feature's value is ease of use is exactly the design pressure behind MXC's SDK-first, audit-mode UX, but at Microsoft scale with three SDK languages.
- Its Windows ProcessContainer tiering is a production-grade answer to the same problem [[WSLC Architecture Deep Dive]] covers from the VM side — MXC unifies both: WSLC is just another selectable backend behind the same request schema.
- Compared to [[How We Contain Claude]], Anthropic's three-pattern postmortem (gVisor, OS sandbox, sealed VM) chosen per use case, MXC generalizes that menu into one dispatch layer — and its policy-hash audit record addresses the observability gap Anthropic's incidents exposed.
- The Nanvix micro-VM backend is a sibling of [[NVX Micro-VM Sandbox]], Microsoft's OpenVMM-based one-VM-per-container design; MXC wraps a Nanvix runtime (CPython in a micro-VM) as one option in the lattice rather than a standalone product.
- Against [[The Rule of 2 (Chromium Security)]], MXC is the "sandboxing" escape hatch applied to agent code: when you must run untrusted, unsafe-language input, MXC supplies the high-privilege containment so two of the three legs never co-occur.

---
*Sources: [[raw/mxc]], [[summary/mxc]]*
*Last updated: 2026-10-18*
