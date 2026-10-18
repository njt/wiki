---
url: https://nono.sh/
title: "nono.sh — Agent runtime security"
author: nono (Nucleus/Enkrypt?) — vendor landing page
date_fetched: 2026-10-18
date_published: not stated
topics:
  - security-and-sandboxing
  - coding-agents-and-frameworks
---

nono is a commercial "agent runtime security" product: kernel-enforced sandboxing for AI agents' tool executions, pitched as zero-setup and zero-latency. Every tool call an agent makes — `gh issue view`, `gh pr merge`, anything on stdio — is wrapped in an ephemeral, invocation-scoped "micro toolbox" provisioned by a supervisor.

The core mechanism is brokered tool execution. The agent never runs a tool directly; the supervisor resolves and verifies the executable, evaluates argv policy for the caller and arguments, and on approval spins up a fresh sandbox with only the capabilities that invocation needs: read-only workspace, no filesystem writes, no host secrets, bounded stdout/stderr, and network access only through a nono proxy. Credentials never enter the child — the sandbox receives a phantom token (e.g. `GH_TOKEN`), and the proxy validates the phantom, checks HTTP method and path policy, then injects the real credential at the boundary over TLS.

Policy failures degrade gracefully across layers: an allowed read flows through; a `gh pr merge` matching no argv rule pauses for a human approve/deny/timeout decision; a denied argv refuses before any sandbox or request exists; and a command that passes argv but makes a forbidden HTTP call is blocked at the L7 proxy. Every decision lands in a hash-chained audit record sealed with a SHA-256 Merkle root, and the sandbox is destroyed when the invocation exits.

Positioning: it "sandboxes any terminal agent" via signed profiles pulled from a registry — no wrappers or rewrites — and claims engineers at large tech companies run it, scaling from a laptop to fleets with the same composable JSON policy, per-tool sandboxing, credential protection (secrets zeroised on exit), and auditable execution.
