---
url: https://www.agentsh.org/docs/
title: agentsh — Execution-Layer Security for AI Agents
author: Canyon Road (Eran Sandler)
date_fetched: 2026-05-15
date_published: unknown
source_urls:
  - https://www.agentsh.org/docs/
  - https://raw.githubusercontent.com/canyonroad/agentsh/main/README.md
topics:
  - security-and-sandboxing
---

# agentsh

agentsh is a policy-enforced execution gateway that sits beneath AI agents and tooling, intercepting file, network, process, and signal activity — including entire subprocess trees. It enforces user-defined policies in real time and produces structured audit events.

Repository: github.com/canyonroad/agentsh

## Architecture

Client-server model:

- **Server**: Runs the policy engine and an underlying enforcement layer (FUSE on Linux, ESF+NE on macOS)
- **Session model**: Users create sessions pinned to a policy; commands are executed within sessions
- **Shim**: A shell shim that can replace /bin/sh or /bin/bash, automatically routing all shell invocations through agentsh
- **Autostart**: The first `agentsh exec` (or any shimmed shell) launches a local server automatically using the configured YAML policy file
- **Sidecar pattern**: Recommended for Docker — run agentsh as PID 1 or a sidecar sharing a workspace volume

## Core Features

1. **Per-operation policy decisions**: allow, deny, approve (human OK), audit, soft_delete, and redirect
2. **Full I/O visibility** across file operations, network connects + DNS, process start/exit, PTY activity, signal send/block, LLM API requests, and declared HTTP services
3. **Two output modes**: human-friendly shell output and compact JSON for agents/tools
4. **Session Reports**: Markdown summaries with decision breakdowns, findings detection, and activity timelines
5. **Workspace Checkpoints**: Snapshots for recovery before risky operations, with diff preview, dry-run rollback, and auto-checkpointing
6. **LLM Proxy with DLP**: Embedded proxy intercepting LLM requests, supporting automatic routing via environment variable injection, PII redaction, custom DLP patterns, token usage tracking, and full audit trails
7. **Policy Generation**: Profile-then-lock workflow that generates restrictive policies from observed session behavior, supporting CI/CD lockdown and agent sandboxing
8. **MCP Security**: Tool whitelisting, version pinning (rug pull detection), cross-server exfiltration blocking, and token bucket rate limiting

## Redirect & Steering

The redirect capability is a distinguishing feature. Rather than simply denying an action (which may cause agents to brute-force workarounds), agentsh can **swap the command or path** transparently.

- **Command redirect**: When a tool like `curl` or `wget` is invoked, policy can swap it for an audited wrapper (e.g., `agentsh-fetch --audit`). The agent perceives success, not an error.
- **File path redirect**: Writes outside an approved workspace can be silently redirected to an alternate directory (e.g., `/workspace/.scratch`). The agent sees a normal write operation, but the data lands where policy dictates.
- **Signal redirect**: A `SIGKILL` to child processes can be transparently converted to `SIGTERM` for graceful shutdown.
- **DNS redirect**: Intercept DNS queries and return configured IPs, with visibility settings (audit_only, warn, silent) and failure modes (fail_closed, fail_open, retry_original)
- **Connect redirect**: Reroute TCP connections to different destinations with TLS options (passthrough or SNI rewrite)

This steering approach keeps agents on desired paths and "reducing wasted retries" by avoiding denial errors that prompt alternative attempts.

## Policy Engine

**Evaluation**: First matching rule wins. Rules live in named policies; sessions select a policy by name.

**Scopes covered**: file operations, commands, environment variables, network (DNS + connect), PTY/session settings, and declared HTTP services.

**Environment policy**: Agents can enforce allowlists and denylists on environment variables, with byte/key count limits, secret stripping, and an `env_block_iteration` flag that prevents enumeration via LD_PRELOAD injection. Operator-trusted `env_inject` values bypass all filtering.

**Integrity**: Optional `policies.manifest_path` can specify a SHA256 manifest to verify policy files at load time.

**Policy packs**: Three opinionated configurations:
- `dev-safe.yaml` — local development with approvals on deletes
- `ci-strict.yaml` — CI runners, denies outbound network except registries
- `agent-sandbox.yaml` — default-deny + explicit allowlists for untrusted code

## Platform Support

- **Linux**: Full enforcement — "100% security score" via FUSE + eBPF + seccomp
- **macOS**: ESF+NE mode — "90% score" in Alpha. ESF provides kernel-level allow/deny but doesn't support transparent file interception. Actions like redirect and soft_delete are implemented as "deny + guidance"
- **Windows**: WSL2 gives "100% score" (Linux-equivalent). Native Windows via minifilter driver + AppContainer ("85% score") is pending driver signing

## Security Modes

Five modes with varying protection levels:

| Mode | Mechanism | Protection |
|------|-----------|------------|
| full | seccomp + eBPF + FUSE | 100% |
| landlock | Landlock + FUSE | ~85% |
| landlock-only | Landlock | ~80% |
| minimal | (none) | ~50% |
| ptrace | SYS_PTRACE | ~95% |

## Authentication & Approval

Multiple auth methods: api_key, oidc (enterprise SSO), hybrid. Human-in-the-loop approval via local_tty prompts, totp authenticator codes, webauthn hardware keys, or a remote API endpoint.

## MCP Security

Tool whitelisting, version pinning (rug pull detection), cross-server exfiltration blocking, and token bucket rate limiting. This is a novel addition — MCP-native security controls that most sandboxing solutions don't address.

## Integration Patterns

- **Docker**: Install release package, activate shell shim, point at server via AGENTSH_SERVER env var
- **AI coding assistants**: Pre-built configuration snippets for Claude Code (CLAUDE.md), Cursor (rules), and generic AGENTS.md
- **CI/CD**: Session reports and policy generation enable pipeline integration

## Scaling

Canyon Road offers two platform products:
- **Beacon**: Monitors AI desktop tools (Claude, Cursor, ChatGPT) on endpoints
- **Watchtower**: Control plane with centralized policies, approval routing (Slack/email/SMS), SIEM export, kill switch
