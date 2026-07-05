---
url: https://gist.github.com/obra/24abe76195f8c0b5b26059ca53345066
title: "Stockyard Design Session (gist)"
author: Jesse Vincent (obra)
date_fetched: 2026-05-15
date_published: 2026-01-16
---

Full transcript of a design session between Jesse Vincent and Claude Code (v2.0.76) for a new project called stockyard. Fresh repo, true greenfield, no commits yet. Working directory: /home/jesse/git/stockyard on a machine called flower-garden with Docker pre-installed.

## Core Concept

A tool to "spin up anywhere between 1 and, if the hardware supported it, hundreds of lightweight containers or micro-containers to run coding agents like Claude Code." Developers would spin up 1–5 in parallel with credentials pre-loaded, running agents in "Dangerously Skip Permissions mode" with GitHub and API credentials ready. Logs, telemetry, and filesystem snapshots are key requirements. Future ambition involves AWS Firecracker, but prototyping is local.

## Key Design Decisions

- Language: Go ("The Firecracker ecosystem has standardized on Go for client tooling.")
- Runtime: Flintlock + Firecracker micro-VMs, not Docker containers. Research showed Ignite is "Dead" (archived Dec 2023), raw Firecracker is "Viable but painful," and Flintlock is the "Best option."
- Secrets: 1Password CLI now, with an abstraction layer for AWS Secrets Manager later. Secrets are namespaced per instance (e.g., op://Stockyard/flower-garden/anthropic-api-key).
- Credentials injection: Via cloud-init at VM startup.
- Workspace persistence: ZFS pool on the host (file-backed initially, moving to a dedicated disk). One ZFS dataset per task/VM.
- Snapshots: High-frequency, near-instant ZFS snapshots triggered from inside the VM via vsock (a "virtual socket for host↔guest communication"), called by agents between tool calls for full audit trails.
- Networking: Each VM joins the tailnet automatically via Tailscale, getting hostnames like stockyard-task-id.
- Image: Fork packnplay's Dockerfile approach, customize it for VM boot (kernel, cloud-init, Tailscale, vsock client).
- CLI commands: run, list, attach, stop, destroy, snapshot, snapshots, restore, logs, cp, configure.
- Architecture: Daemon-centric (stockyardd as control plane with gRPC/REST API, SQLite for state), CLI as thin client, with future web UI in mind.
- Implementation approach: Study packnplay's code for patterns, but implement stockyard "fresh" from scratch (not a fork).

## Workflow Example

stockyard run --repo github.com/org/repo --ref branch -- claude-code --dangerously-skip-permissions -p "do the thing"

Spins up a VM, clones the repo, injects secrets, and runs the agent.

## Design Document

A 484-line design doc was committed as the initial commit (1014ebc) to docs/plans/2026-01-16-stockyard-design.md.
