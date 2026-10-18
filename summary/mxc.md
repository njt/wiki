---
url: https://github.com/microsoft/mxc
title: "Microsoft eXecution Container (MXC)"
author: Microsoft
date_fetched: 2026-10-18
date_published: 2026-10-18
topics:
  - security-and-sandboxing
  - coding-agents-and-frameworks
---

MXC is Microsoft's open-source (MIT) **sandboxed code execution system** for running untrusted code — model output, plugins, tools — on Windows, Linux, and macOS. It ships as an embeddable SDK dependency (Rust, .NET, Node) rather than a standalone service: your application supplies a JSON `ContainerRequest` (command, filesystem/network/UI policy, timeout), and MXC validates it, selects a platform-appropriate containment backend, and launches the workload.

The interesting part is the backend lattice behind one unified containment model: ProcessContainer (Windows AppContainer/BaseContainer with a three-tier fallback), Windows Sandbox, WSLC, IsolationSession, Bubblewrap, LXC, macOS Seatbelt, plus experimental MicroVM (Nanvix) and Hyperlight backends. The same versioned JSON config schema (`schemas/stable/`, now at 1.0.0) drives all of them, and standalone executor binaries like `wxc-exec.exe` accept the same requests for SDK-less use.

Two developer-experience ideas stand out: **audit mode** (`wxc-exec --audit`), which deliberately turns sandboxing off for a trusted workload, records every access the OS *would* have denied via ETW tracing, and emits an adjusted policy — policy authoring by observation rather than guesswork; and a policy-hash audit record (`mxc.PolicyHash`) logged before any backend engages, so every run is anchored to the canonical identity of the policy that governed it.

Implementation is a Rust workspace (`mxc-sdk` crate with `backends/*` and `core/mxc_engine`, the `wxc` executor CLI, the Windows-only `plm` Permissive Learning Mode trace helper) with Node and .NET SDKs wrapping the Rust core via FFI. Versioned contract tests (`mxc_contract_v1_0_0.rs` etc.) pin the public API surface across schema versions.
