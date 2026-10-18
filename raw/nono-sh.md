---
url: https://nono.sh/
date_fetched: 2026-10-18
---

# Agent runtime security. Kernel-enforced boundaries for AI agents, with zero setup and zero latency.

## Quickstart

The industry standard for software signing, used by PyPI, Homebrew, Maven, Google, GitHub, and NVIDIA.

## Every tool execution. Isolated and scoped.

Brokered Tool Execution

Every tool call gets an ephemeral micro toolbox, scoped to only the files, network routes, arguments, and credentials it needs. Every decision is auditable.

How nono's ephemeral micro toolboxes work: a coding agent runs a tool invocation, for example gh issue view. The nono supervisor resolves and verifies the executable, evaluates argv policy for the caller and arguments, and on approval creates a fresh, invocation-scoped micro sandbox around the tool with only its selected capabilities: read-only workspace, no filesystem writes, no host secrets, bounded stdout and stderr, and network access only through the nono proxy. The sandbox receives a phantom GH_TOKEN; the real credential stays with the supervisor and never enters the child. The proxy validates the phantom token, evaluates HTTP method and path policy, injects the real credential at the boundary, and forwards over TLS to api.github.com. If an action falls outside the provided policy, the policy engine can route it to a human approval decision to approve, deny, or time out. Security-relevant events are hash-chained into an audit record sealed with a SHA-256 Merkle root, and the sandbox is destroyed when the invocation exits. Four scenarios: Allowed read — gh issue view runs, POST /graphql is allowed by the proxy, and output returns to the coding agent. Human approval — gh pr merge matches no argv rule, so the exact invocation pauses for a human decision and, once approved, executes inside a scoped sandbox. Argv denied — gh issue comment is refused at argv authorization; no sandbox or outbound request is ever created. L7 denied — gh api passes the broad argv rule, but the POST to the repository comments endpoint is denied at the proxy; nothing reaches GitHub.

The agent never talks directly to the isolated tool. Requests, credentials, network traffic, stdio, and audit events cross the supervisor boundary.

## Works with your agents

nono sandboxes any terminal agent. Pull a signed profile from the registry and run — no wrappers, no rewrites.

## Engineers at some of the world’s largest tech companies run nono to protect their agents.

It scales from a single laptop to fleets of agents without changing the model — the same composable policy, per-tool sandboxing, credential protection, and fully auditable execution.

### Composable policy

Scalable JSON profiles that compose across teams and version-control alongside your code.

### Tool-specific sandboxing

A sandbox tuned to each agent and tool — least privilege by default, not one blunt boundary.

### Credential protection

Secrets are injected at the boundary and never exposed to the agent. Zeroised on exit.

### Fully auditable

Every action lands in a cryptographic, tamper-evident audit trail you can verify.
