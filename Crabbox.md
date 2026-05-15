# Crabbox

Open-source agent workspace control plane that gives software maintainers and AI agents throwaway remote compute without sharing cloud credentials. "Warm a box, sync the diff, run the suite." The local edit-save-run loop stays unchanged; compute, tests, and review evidence move to remote capacity.

---

## Architecture

Three tiers: a Go CLI on the laptop, a Cloudflare Worker broker (single Durable Object owning provider credentials, lease state, and cost guardrails), and cloud providers. The CLI mints a per-lease SSH key, calls `POST /v1/leases`, the broker provisions a machine, and the CLI syncs the dirty checkout over rsync before streaming the command over SSH.

The key insight: **the broker owns the credentials, not the CLI and not the runner.** Individual machines and CLIs never need provider tokens. This is the same architectural bet as [[OneCLI]] applied to infrastructure provisioning — brokered, ephemeral access through a credential boundary.

## Key Quotes

> "A short-lived box for every *run*."

The atomic unit isn't a project or a branch — it's a single command invocation. This is throwaway infrastructure taken to its logical conclusion. Compare with [[Minions — Stripe's One-Shot Coding Agents]] which uses pre-warmed devboxes for 1,000+ unattended PRs/week. Crabbox goes one step further: the box lives exactly as long as the run.

> "Behind the scenes a small Cloudflare-hosted broker owns cloud provider credentials, lease state, cleanup, usage, and cost guardrails so individual machines and CLIs never need to."

This is the architectural thesis. The broker is a [[Smart Models Dumb Pipes]] pattern applied to infrastructure: the CLI is the smart judgment layer (what to run, where, with what config), the broker is the dumb pipe (serialized lease state, provider API calls). The broker doesn't decide; it enforces caps and executes provisioning.

> "Warm a box, sync the diff, run the suite."

The tagline captures the entire UX in eight words. No clean checkout, no Docker build, no CI YAML — just a warm machine, an rsync of tracked+nonignored files, and your command. Fingerprint skip when nothing changed means sub-second no-op runs.

> "Every coordinator-backed run gets an early `run_...` handle ... durable lifecycle/output events ... retained output after completion."

Run observability is first-class. Every run produces a stream of typed events, persisted beyond the session, queryable with `crabbox events --after <seq>`. This is what makes autonomous agent work reviewable — compare with the ephemeral terminal output that vanishes with most ad-hoc remote execution setups.

## Key Themes

#tool #agent-infrastructure #sandboxing #ci-cd #remote-execution

**Brokered infrastructure.** The credential-boundary pattern is the most interesting architectural decision. It means agents can lease machines without holding AWS keys. This is directly applicable to the problem [[You Dont Want Long-Lived Keys]] describes: the lease lifetime IS the credential lifetime.

**Workspace as a lease, not a server.** No long-running dev servers, no SSH config to maintain. Every box has a TTL. Idle timeout auto-releases. Monthly spend caps prevent runaway costs. This is the operational model that makes agent-driven infrastructure safe — the same constraint-first thinking behind [[yolo-cage]] and [[claude-ctrl]].

**Provider abstraction that earns its keep.** Support for 16 providers (Hetzner, AWS, Azure, GCP, Proxmox, static SSH, Blacksmith, Namespace, Semaphore, Sprites, Daytona, Islo, E2B, Modal, Tensorlake, Cloudflare) with capacity fallback across compatible instance families. The "beast/standard/fast/large" class system maps to provider-native sizes. This isn't a thin wrapper — it handles Spot vs on-demand, image baking, and per-provider bootstrap paths (cloud-init for Linux, Custom Script Extension for Windows).

**Agent-native design.** The OpenClaw plugin (from the same org that built [[acpx]]) exposes Crabbox as agent tools. But run inspection stays CLI-led — the plugin does lifecycle operations; history, events, logs, and results stay in shell commands. This is a clean separation: the agent leases and runs; the human (or a shell-capable agent) inspects.

**Evidence, not just output.** Artifacts bundle screenshots, video, JUnit summaries, logs, and lease metadata for PR evidence. WebVNC streams desktops into browsers. `desktop proof` captures metadata, screenshots, diagnostics, MP4, and contact-sheet PNGs. This is [[Compound Engineering]] applied to test infrastructure: don't just run tests, produce reviewable evidence that they passed.

## Critical Analysis

**What's genuinely novel:** The brokered credential model. Most remote-execution tools either give agents cloud credentials (dangerous) or require pre-provisioned machines (inflexible). Crabbox's broker sits in the middle — it provisions on demand, but the agent never touches a provider key. This is a genuinely useful security primitive that [[Navaris]] and [[OpenSandbox]] don't address — they focus on sandboxing the runtime, not brokering the provisioning.

**What's ambitious:** The provider surface area. 16 providers suggests either deep integration or thin wrapping. From the docs, the major cloud providers (Hetzner, AWS, Azure, GCP) get serious treatment with capacity fallback, image baking, and Windows/WSL2/macOS support. The delegated providers (Daytona, E2B, Modal, etc.) are thinner wrappers. The risk is that maintenance surface area grows with every new provider, and the quality bar varies.

**What's missing:** The docs don't address what happens when the broker itself is a single point of failure. The Cloudflare Worker + single Durable Object architecture means the broker is the availability bottleneck. If the Worker is down, no leases can be created. This is acknowledged implicitly (there's a direct-provider mode for debugging the broker), but the operational story for broker HA isn't surfaced.

**The OpenClaw ecosystem play:** Crabbox, [[acpx]], and the OpenClaw plugin system form a coherent agent infrastructure stack. acpx handles agent-to-agent protocol communication; Crabbox handles agent-to-infrastructure workspace provisioning. Together they solve the two hard problems of multi-agent systems: coordination and execution environment.

**Comparison to Stripe's approach:** [[Minions — Stripe's One-Shot Coding Agents]] uses pre-warmed devboxes with 10s spin-up. Crabbox is general-purpose (not tied to one org's infra) but likely has higher cold-start latency for cloud-provisioned boxes. The warm-reuse pattern (`crabbox warmup` + `--id`) partially addresses this, but Crabbox doesn't claim 10s spin-up times.

**Bottom line:** If you're building an agent system that needs to run code on remote machines, Crabbox solves the credential problem better than any alternative I've seen. If you're a solo developer wanting faster CI, the complexity is probably overkill. The sweet spot is teams that already have multi-provider cloud accounts and want a unified, agent-safe interface to throwaway compute.

---

## Related Pages

- [[acpx]] — same org, agent protocol layer
- [[Security and Sandboxing]] — the broader containment context
- [[Minions — Stripe's One-Shot Coding Agents]] — pre-warmed devboxes at scale
- [[yolo-cage]] — alternative agent sandboxing approach
- [[A Deep Dive on Agent Sandboxes]] — sandbox primitives
- [[OpenSandbox]] — Alibaba's sandbox platform
- [[Navaris]] — unified sandbox control plane
- [[Dev Containers]] — infra-as-code dev environments
- [[Windows in Docker]] — remote Windows execution
- [[OneCLI]] — same credential-brokering pattern for APIs
- [[You Dont Want Long-Lived Keys]] — the principle behind lease-scoped credentials
- [[Smart Models Dumb Pipes]] — broker as dumb pipe
- [[Compound Engineering]] — evidence and verification layers
- [[klaw.sh]] — Kubernetes-style agent lifecycle
- [[Building Agents for Production Systems with MCP]] — agent-to-infrastructure connectivity

---

*Sources: [[raw/crabbox]], https://crabbox.sh/, https://github.com/openclaw/crabbox*
*Last updated: 2026-05-15*
