---
url: https://crabbox.sh/
title: Crabbox
author: OpenClaw
date_fetched: 2026-05-15
date_published: unknown
topics:
  - security-and-sandboxing
---

Crabbox is an open-source agent workspace control plane for software maintainers and AI agents. It lets you lease managed cloud capacity, point at an existing SSH host, or use an agent sandbox provider, then sync your dirty checkout, run commands remotely, stream output, collect evidence, and release. The tagline: "Warm a box, sync the diff, run the suite."

Architecture: Go CLI on the user's laptop → Cloudflare Worker broker (owns provider credentials, lease state, cost guardrails) → cloud provider (Hetzner, AWS, Azure, GCP, Proxmox, static SSH, or delegated sandbox providers like Daytona, E2B, Modal, Tensorlake, Blacksmith, Namespace, Semaphore, Sprites, Islo, Cloudflare).

Key design decisions:
- Brokered leases: the CLI carries only a bearer token; the Worker holds provider credentials
- TTL-bounded machines with monthly spend caps and per-user/org/provider usage tracking
- Warm reuse via `crabbox warmup` + `--id` for repeated runs
- Actions hydration: reuse GitHub Actions setup steps so local runs land in the same hydrated workspace
- WebVNC for streaming Linux, macOS, or Windows desktops into a browser
- Artifacts bundle screenshots, video, JUnit summaries, logs, and lease metadata for PR evidence
- OpenClaw plugin exposing agent tools: crabbox_run, crabbox_warmup, crabbox_status, crabbox_list, crabbox_stop
- Multi-provider with fallback across compatible instance families when capacity is constrained

Machine classes: standard, fast, large, beast — with provider-specific instance type mappings and Spot preference by default. Azure supports Linux, native Windows, and Windows WSL2. AWS supports Linux, Windows, WSL2, and EC2 Mac. Static SSH supports existing Linux, macOS, Windows, and WSL2 hosts.

Install: `brew install openclaw/tap/crabbox`. MIT licensed. Docs at https://openclaw.github.io/crabbox/.
