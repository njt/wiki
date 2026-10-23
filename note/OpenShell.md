# OpenShell — NVIDIA's Policy-Enforced Runtime for Agent Fleets

OpenShell is NVIDIA's open-source Rust runtime for running fleets of autonomous AI agents without giving them unrestricted access to data, secrets or the network. What makes it notable is not sandboxing per se but the pairing of **kernel/proxy enforcement at runtime** with **formal verification (Z3 SMT) of every policy change before it applies** — treating agent permissions as something you prove properties about, not just configure. It is a ~610K-line, 38-crate Rust workspace with a gateway control plane, host supervisors, and capability-free sandbox runtimes, plus SDKs in four languages and installable agent skills.

---

## Architecture

Three planes, each its own crate family:

- **Gateway (`crates/openshell-server`, `openshell-gateway`)** — the control plane: gRPC services, sandbox lifecycle, policy store, mTLS/OIDC auth boundary, protocol multiplexing (gRPC + HTTP on one port). Compute drivers (Docker, Podman, Kubernetes, libkrun VM, Windows MXC AppContainer) are compiled into a registry at startup; runtime selection consults the registry rather than switching on driver names (`crates/openshell-driver-*`).
- **Supervisor (`crates/openshell-supervisor`, `openshell-supervisor-network`, `openshell-supervisor-process`)** — per-host session owner. The network supervisor runs an HTTP CONNECT egress proxy with OPA policy evaluation, binary-identity caching via procfs, TLS interception, and credential injection (`proxy.rs`, `l7/`, `identity.rs`). It deliberately `#![forbid(unsafe_code)]`.
- **Sandbox (`crates/openshell-sandbox`)** — the "capability-free workload launcher": inside the sandbox, processes hold no secrets and no network authority. All I/O is seccomp-mediated through a boundary protocol; the supervisor adds credentials only to requests bound for approved endpoints. A `RuntimeQualification` struct actively verifies (seccomp evidence, Landlock ABI, DNS round-trips) that the admitted runtime actually enforces before protected bootstrap material is consumed — fail-closed startup.

Policies are YAML documents with five sections (filesystem via Landlock LSM, process identity, L4 network rules, L7 rules, middleware), parsed in `openshell-policy-schema` (dependency-light, bounded parsing) and adapted to protobuf in `openshell-policy`. Filesystem/process rules are fixed at sandbox start; network rules hot-reload at runtime.

## Key techniques

- **The Z3 policy prover (`crates/openshell-prover`)** is the signature move. It encodes policy, binary capabilities (e.g. `git-remote-https`, `ssh`, `nc` marked as able to bypass L7 inspection) and credential scopes as SMT constraints and answers four reachability questions: link-local/metadata reach (cloud metadata serves credentials), L7-bypass-plus-credential, credential-reach expansion, and capability expansion. Findings are categorical, not severity-graded; a gateway "finding delta" surfaces only what a proposed change *newly* allows, and auto-approval gates on no-new-findings. The prover is standalone (`openshell-prover` CLI) and explicitly designed for **agents to run**: a parent agent can boundary-check a policy it writes for a subagent against its own maximum policy.
- **L7-aware inspection** in the proxy (`l7/rest`, `graphql.rs`, `jsonrpc.rs`, `mcp.rs`, `websocket.rs`): per-request parsing inside the tunnel, OPA evaluation, forward-or-deny — instead of raw `copy_bidirectional` relaying once the CONNECT tunnel opens.
- **Deny-by-default network with per-binary rules**: each `network_policies` rule binds destinations to specific binaries, so a compromised `curl` doesn't inherit the allowlist of `git`.
- **OCSF v1.8.0 structured security logging** (`crates/openshell-ocsf`) — every denial and policy decision is an OCSF event with builders, not free-text logs; the supervisor's lib.rs even documents which log sites are deliberately *not* OCSF-ified.
- **Agent-first construction**: public `skills/` (policy generation, inference debugging, CLI operation) that work outside a source checkout, contributor skills in `.agents/skills/`, and reviewer sub-agent personas in `.agents/agents/`. The AGENTS.md states plainly: "this is not vibe coding."

## Design decisions

- **Proof at change time, enforcement at run time.** Runtime enforcement (Landlock/seccomp/proxy) catches everything the model covers; the prover exists because the *dangerous* event is a policy change, and those can be checked symbolically before approval. The cost is an honest gap: docs concede the prover's guarantees apply only "to the policy features and behavior represented by its model," and SMT modelling of TLS interception semantics is inherently partial.
- **Credentials never enter the sandbox** — the classic broker pattern taken to the kernel boundary. Trade-off: every egress flows through the supervisor's proxy, so latency and proxy complexity are accepted as the price of credential isolation.
- **Bounded parsing and constant limits everywhere** (max 1,024 rules, 4,096 endpoints, 16,384 L7 rules in `containment.rs`) — policy files are untrusted input, parsed with the same paranoia as wire data.
- **Telemetry is compiled-out-able** with a CI check (`verify-telemetry-compiled-out.sh`), and the supervisor fails the build if telemetry and no-telemetry features are combined — an unusually principled stance for a vendor product.

## Comparison notes

- Like [[SPACE — Perplexity's Firecracker Sandbox Platform]], OpenShell separates a control plane from per-workload isolation and injects credentials at the network proxy so guests never hold secrets — but Perplexity optimizes for create-speed (Firecracker snapshot restore), while OpenShell optimizes for *provability of the permission model* and adds formal verification of policy deltas, which SPACE has no analogue for.
- [[MXC — Microsoft eXecution Container]] shares the "many containment backends behind one contract" idea (AppContainer, Bubblewrap, Seatbelt…); OpenShell's driver registry is the same pattern plus an audit/verification layer and a fleet-scale gateway rather than a per-run SDK.
- [[nono.sh — Brokered Tool Execution for Agent Security]] brokers individual tool calls with human approval gates; OpenShell brokers the whole sandbox's egress and filesystem, and replaces per-action human gates with solver-verified policy diffs — a batch answer to nono's per-call question.
- Against [[CaMeL — Defeating Prompt Injections by Design]], OpenShell doesn't try to stop prompt injection; it shrinks what an injected agent can *do* — kernel confinement plus credential-brokered egress is a blast-radius strategy, complementary to CaMeL's dataflow control.
- [[Best Infrastructure Platforms for Coding Agents in 2026]] surveys sandbox vendors converging on Firecracker micro-VMs for CPU sandboxing; OpenShell is the heavyweight, policy-native counterpoint — a full governance plane (prover, OPA, OCSF, Helm/K8s) where those platforms offer a fast spawn API.

---
*Sources: [[raw/openshell]], [[summary/openshell]]*
*Last updated: 2026-10-24*
