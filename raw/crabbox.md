---
url: https://crabbox.sh/
date_fetched: 2026-07-05
backfilled: true
---

### Local loop, remote box

Keep your editor and git workflow. Crabbox rsyncs your dirty checkout to a leased remote box and streams the run back.

OpenClaw - remote testbox

Crabbox gives maintainers and agents a fast local loop on shared cloud capacity: lease, sync, run, release. The CLI keeps the developer story simple; a Cloudflare-hosted broker keeps the fleet safe.

Keep your editor and git workflow. Crabbox rsyncs your dirty checkout to a leased remote box and streams the run back.

A Cloudflare Worker holds provider credentials and serializes lease state. Your CLI only carries a bearer token.

TTL-bounded machines, monthly spend caps, and per-user / per-org / per-provider usage from the broker.

`crabbox warmup` keeps a box hot. Reuse it with `--id` across runs, SSH, and CI hydration.

Brokered Hetzner / AWS / Azure, delegated E2B / Daytona / Blacksmith / Semaphore, or static SSH targets - Linux, Windows, and macOS.

`actions hydrate` reuses your repository's GitHub Actions setup steps so local runs land in the same hydrated workspace.

`crabbox webvnc` streams a Linux, macOS, or Windows desktop into the browser. Drive UI tests, capture screenshots, hand the live session to a teammate - no VPN.

`crabbox artifacts collect` bundles screenshots, video, JUnit summaries, logs, and lease metadata. Drop it on a PR as before/after evidence instead of scraping log output.

**Warm a box, sync the diff, run the suite.**

Crabbox is a generic remote software testing and execution control plane. It keeps the local developer story unchanged: edit, save, run. The difference is where the command executes. Crabbox moves tests, builds, browser checks, platform validation, and review evidence onto owned or provider-backed remote capacity, then streams the result back to the caller.

It is for maintainers, contributors, and automation that need a repeatable way to run repository commands on a machine other than the local laptop. A `crabbox run` leases a brokered cloud machine, reuses a static SSH host, or delegates to a sandbox provider; syncs your tracked, non-ignored local files; executes the command remotely; streams stdout and stderr back; records evidence; and then releases or unclaims the target.

Use Crabbox when local compute is too small or slow, when a workflow needs a fresh disposable runner, when the target platform is remote, or when an AI agent or reviewer needs auditable command output from the exact environment that ran. It is not a CI service, a package manager, a production deployment platform, a hostile multi-tenant sandbox, or a tool that automatically sanitizes secrets from command output and artifacts.

An optional coordinator owns cloud provider credentials, lease state, cleanup, usage accounting, and cost guardrails so individual machines and CLIs never hold those. The coordinator runs either on Cloudflare Workers with a Durable Object or on Node.js with PostgreSQL.

```
your laptop                 coordinator runtime              cloud provider
-------------               -------------------              --------------
crabbox CLI    -- HTTPS --> Cloudflare + Durable Object  --> Hetzner / AWS / Azure / GCP
   |                      or Node.js + PostgreSQL              |
   |                                                           |
   +------------- SSH + rsync to leased runner <---------------+
```
The CLI is a Go binary (`cmd/crabbox`, `internal/cli`). Shared coordinator behavior lives in `worker/src`; Cloudflare and Node/PostgreSQL provide runtime adapters. Lease lifecycle calls go through the coordinator over HTTPS, but the data plane — SSH, rsync, and command execution — goes **directly from the CLI to the runner host**. Runners hold no coordinator credentials; they are leaf nodes.

Crabbox selects one of three execution modes per provider:

`aws`, `azure`, `gcp`, and `hetzner` when a broker URL isconfigured (`CRABBOX_COORDINATOR`). The coordinator provisions and tracks leases; the CLI still drives sync and command execution over SSH.

hosts (`provider: ssh`) and self-hosted/local providers. The CLI talks to the cloud or host API itself.

Firecracker providers) that own sync and run end to end; there is no SSH lease.

Brokered Linux runners are vanilla Ubuntu boxes prepared by cloud-init with SSH, Git, rsync, and `/work/crabbox`. AWS and Azure can also broker Windows (normal and WSL2) and, on AWS, EC2 Mac desktop targets. Project runtimes come from Actions hydration or repo-owned setup.

`POST /v1/leases` on thebroker (brokered mode) or provisions directly (direct mode).

worst-case TTL cost, provisions a server with region/market fallback, and returns host / port / user / workdir / expiry / slug.

`crabbox-ready` marker, seeds remote Git when possible,rsyncs the Git file-list manifest, runs sync guardrails, and hydrates the configured base ref.

sends heartbeats.

`--keep` is set. Kept leases stillauto-release after the idle timeout, and the broker frees reserved cost when the lease closes.

See How Crabbox Works for the full picture, including warm-box reuse and the brokered-vs-direct paths. See the Source Map to trace any documented behavior back to code.

`brew install openclaw/tap/crabbox`Verify with `crabbox --version`.

```
# log in once per machine — stores a broker token in user config
crabbox login --url https://broker.example.com
# one-shot run on a fresh leased box
crabbox run -- pnpm test
# keep a warm box around for repeated runs; output includes an id and a slug
crabbox warmup
crabbox run --id swift-crab -- pnpm test:changed
crabbox ssh --id swift-crab
crabbox stop swift-crab
```
Each lease has a canonical id (`cbx_<12 hex>`) and a friendly slug (`<adjective>-<noun>`); most commands accept either via `--id`. Run `crabbox doctor` to validate local config, broker/provider reachability, and SSH key availability before a long workflow, and `crabbox usage` to summarize recent spend by user, org, provider, and server type.

Pick whichever matches your intent:

How Crabbox Works, Concepts and glossary.

Architecture, Orchestrator, Runtime adapter stack, Broker auth and routing, Coordinator, Bring your own infrastructure, Slurm academic sandboxes.

Portable coordinator, Operations, Security.

Command reference, Feature reference, Configuration, Jobs, Pond, Actions hydration, Capsules, Checkpoints, Browser portal, Runtime adapter stack, Capabilities, Interactive desktop and VNC, Telemetry, Sync.

Providers feature overview, Provider selection, Provider landscape, Provider live smoke, Provider authoring, Provider backends, Capacity fallback, Slurm academic sandboxes, Network, Tailscale. Per-provider: AWS, Azure, Azure Dynamic Sessions, Google Cloud, Hetzner, DigitalOcean, Linode, Vultr, Proxmox, XCP-ng, Incus, Parallels, Local Container, Multipass, Static SSH, Railway, RunPod, Blacksmith Testbox, KubeVirt, External, Namespace Devbox, Namespace Compute Instance, Semaphore, Sprites, Tenki, Coder, Daytona, Islo, E2B, Modal, Agent Sandbox, OpenComputer, Freestyle, Anthropic Sandbox Runtime, Tensorlake, Upstash Box, Weights & Biases, Cloudflare.

Observability, Troubleshooting, Performance, Cost and usage, Lifecycle and cleanup.

Portable coordinator, Security, Auth and admin, Repository onboarding, SSH keys, Source Map.

Markdown in this directory is the user-facing documentation source. Implementation truth stays in code; the Source Map lists the files behind each documented behavior. The GitHub Pages site at <https://openclaw.github.io/crabbox/> is generated from these Markdown files by `scripts/build-docs-site.mjs` and deployed by `.github/workflows/pages.yml`. Pages must be enabled on the repository or organization for the workflow to publish.

Build and check the docs site locally:

```
scripts/check-docs.sh
open dist/docs-site/index.html
```
