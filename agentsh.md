# agentsh

Execution-layer security for AI agents: a policy-enforced gateway that sits *under* the agent, intercepting every file operation, network connection, process spawn, and signal — then deciding allow/deny/approve/redirect in real time. Built by Eran Sandler (@erans), the same person behind lunaroute and pgsqlite.

---

## The Big Idea

Most agent sandboxes work at the OS boundary ([[A Deep Dive on Agent Sandboxes]], [[yolo-cage]], [[OpenSandbox]]). agentsh works at the *syscall* boundary, which means it sees not just the top-level command the agent runs but everything its subprocesses do. The docs call this "Execution-Layer Security" (ELS) — a runtime enforcement model that governs the hidden work done by child processes.

## Redirect, Don't Deny

This is the feature that makes agentsh interesting. Most security tools follow a binary allow/deny model. Deny an action and the agent sees an error, then tries workarounds. agentsh supports `redirect` as a first-class policy decision:

> "Swap commands or file paths to keep agents on the paved road."

- **Command redirect**: Agent calls `curl`, policy silently swaps it for `agentsh-fetch --audit`. Agent sees success.
- **File path redirect**: Agent writes outside workspace, policy redirects to `/workspace/.scratch`. Agent sees normal I/O.
- **Signal redirect**: `SIGKILL` gets converted to `SIGTERM` for graceful shutdown.
- **DNS redirect**: Intercept queries, return configured IPs, with tunable visibility (audit_only/warn/silent).
- **TCP connect redirect**: Reroute connections to different destinations with TLS SNI rewriting.

This matters because agents brute-force around denials. A denied `curl` becomes `wget` becomes `python -c "import urllib"`. Redirect eliminates the retry loop — the agent thinks it succeeded and moves on.

## Full Feature Set

Nine policy decision types: `allow`, `deny`, `approve` (human-in-loop), `audit`, `soft_delete`, and `redirect`. Full I/O visibility across files, network, processes, PTYs, signals, LLM API requests. Two output modes (human shell, agent JSON). Session reports in Markdown. Workspace checkpoints with diff preview and dry-run rollback.

The **LLM Proxy with DLP** is an embedded proxy that intercepts LLM API requests, applies PII redaction and custom DLP patterns, tracks token usage, and produces audit trails. It can auto-route via environment variable injection — so the agent never sees real API keys.

**MCP Security** is a novel addition: tool whitelisting, version pinning to detect rug pulls, cross-server exfiltration blocking, and token bucket rate limiting. Nobody else is doing MCP-native security controls.

**Policy generation** has a profile-then-lock workflow: observe what the agent does, then generate a restrictive policy from that behavior. Useful for CI/CD lockdown and agent sandboxing.

## Platform Reality Check

Linux gets the full stack: FUSE + eBPF + seccomp, 100% of features. macOS is a second-class citizen: ESF provides kernel-level allow/deny but *doesn't support transparent file interception*. On macOS, `redirect` and `soft_delete` are implemented as "deny + guidance" — blocked at the ESF level with retry instructions, which is semantically different from true transparent redirection. The docs rate macOS at 90% but that overstates it for redirect-heavy use cases.

Windows is WSL2 (full Linux stack) or native via minifilter driver + AppContainer (pending driver signing, 85%).

Five security modes scale from `full` (seccomp+eBPF+FUSE) down to `minimal` (no kernel mechanisms), with Landlock and ptrace intermediates. The `landlock` mode at ~85% is the practical sweet spot for systems that can't run eBPF.

## Architecture

Client-server with a shell shim that replaces `/bin/sh` or `/bin/bash`. Commands route through the shim to the policy engine. Sessions are pinned to named policies. Docker integration follows a sidecar pattern (agentsh as PID 1 or adjacent container sharing a volume). Policy evaluation is "first matching rule wins" across file ops, commands, env vars, network, PTY settings, and declared HTTP services.

Authentication supports api_key, OIDC/SSO, and hybrid modes. Approval can use local TTY prompts, TOTP, WebAuthn hardware keys, or a remote API endpoint.

## Canyon Road Platform

The commercial layer: **Beacon** monitors AI desktop tools (Claude, Cursor, ChatGPT) on endpoints. **Watchtower** is the control plane with centralized policies, approval routing to Slack/email/SMS, SIEM export, and a kill switch. The open-source agentsh is the reference implementation; the platform is the enterprise scaling story.

## Critical Analysis

**The redirect primitive is genuinely novel and underappreciated.** Every other sandbox is binary. The insight that denial creates retry loops — and that transparent redirection short-circuits them — is correct and important. This should be table stakes for agent security tooling.

**But macOS is a real problem.** If you're building for a world where developers use Macs (which they do), the gap between "transparent redirect on Linux" and "deny + guidance on macOS" is significant. The macOS ESF limitation means the flagship feature literally doesn't work the same way on the most common developer platform. This isn't agentsh's fault — it's an Apple API constraint — but it limits the addressable market.

**The LLM proxy is both smart and risky.** Intercepting LLM API calls for DLP and audit is valuable, but the proxy itself becomes a single point of interception that an attacker who compromises agentsh can control. The defense-in-depth question: what monitors the monitor?

**MCP security is an underexploited niche.** Tool whitelisting and version pinning for MCP servers is something nobody else is doing, and it addresses real risks (rug-pull package updates, cross-server data exfiltration). This alone could justify agentsh for teams that depend heavily on MCP.

**The policy-generation workflow is clever.** Profile-then-lock is the right approach for CI/CD — observe what the agent actually does, then generate the minimal policy that allows exactly that. Beats hand-writing policies from scratch.

**Comparison to peers:** [[yolo-cage]] is architecturally simpler (Vagrant + egress proxy, defer to PR review) but can't do transparent redirect. [[claude-ctrl]] enforces via hooks and SQLite — same "don't just prompt" philosophy, but at the application layer rather than the kernel layer. [[VTcode]] does tree-sitter command parsing which is deeper than agentsh on the command-validation axis, but shallower on network/filesystem control. agentsh is trying to be the most complete single solution, which is both its strength and its complexity risk.

## What It Fills

agentsh directly addresses the "Network-level agent firewalls" gap identified in [[Security and Sandboxing]] — its DNS and TCP connect redirect are exactly the semantic network control that OS sandboxing can't provide. The MCP security features address another unserved need. The workspace checkpoints and session reports fill part of the "Audit and forensics" gap.

---

*Sources: [[raw/agentsh]]*
*Last updated: 2026-05-15*
