---
url: https://github.com/NVIDIA/OpenShell
title: "OpenShell — Safe, Private Runtime for Fleets of Autonomous AI Agents"
author: NVIDIA
date_fetched: 2026-10-24
date_published: 2026
topics:
  - security-and-sandboxing
  - coding-agents-and-frameworks
---

OpenShell is NVIDIA's open-source (Apache 2.0, Rust) runtime for running fleets of autonomous AI agents safely: agents get file access, package installs, APIs and credentials, but every file access, syscall and network connection is governed by a declared policy enforced at the kernel and proxy level.

Two mechanisms carry the security story. First, **kernel-level enforcement**: each agent runs in an isolated sandbox (Docker, Podman, libkrun VMs, Kubernetes pods, or Windows AppContainers) where Landlock confines the filesystem, seccomp mediates syscalls, and all egress flows through a policy-checking proxy — agents never hold real credentials, which are injected only into requests bound for approved endpoints. Second, **formally verified policy changes**: a Z3-based prover encodes policies, binary capabilities and credential scopes as SMT constraints and flags risky new access (new credentialed hosts, L7-bypassing protocols, new HTTP methods) so those changes wait for human review.

The architecture is a gateway (control plane: sandbox lifecycle, policy store, mTLS/OIDC auth boundary), a supervisor per host (egress proxy with OPA policy evaluation, L7 inspection of REST/GraphQL/JSON-RPC/MCP/WebSocket traffic, credential injection), and the capability-free sandbox runtime itself. A 38-crate Rust workspace (~610K lines) ships CLI, four SDKs (Python/TS/Go/Rust), Helm deployment, OCSF security-event logging, and installable agent skills — the project is explicitly "built agent-first" with its own AGENTS.md and contributor skills.
