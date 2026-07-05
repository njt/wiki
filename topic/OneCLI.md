# OneCLI

An open-source credential vault designed for AI agents. Instead of embedding API keys in agent configurations, OneCLI acts as an HTTP gateway that intercepts outbound requests and transparently injects the real credentials. Agents use placeholder keys and never touch the real secrets.

---

## Key Themes

#security #agent-architecture #guardrails

OneCLI's architecture is elegant in its simplicity. Agents make standard HTTP calls with fake credentials. The Rust gateway intercepts requests, matches them against stored patterns (host + path), swaps in real credentials, and forwards the authenticated request. The agent never sees, stores, or logs the real key.

Key design decisions:

- **AES-256-GCM encryption at rest** -- secrets decrypted only during request processing
- **Host & path matching** -- credentials scoped to specific API endpoints, not global
- **Multi-agent support** -- each agent gets a scoped access token with defined permissions
- **Bitwarden integration** -- pull credentials from existing password managers

This is "like 1Password but for agents" -- the annotation captures it perfectly. The credential surface area shrinks from "every agent config file" to "one encrypted store behind a gateway."

Connects to [[You Dont Want Long-Lived Keys]] (the credentials OneCLI manages should still be ephemeral where possible), [[Navaris]] (agents in sandboxes need credentials), [[OpenSandbox]] (same), and [[llm-guard]] (complementary -- OneCLI handles credential security, llm-guard handles prompt/output security).

## Critical Analysis

The proxy-based approach is the right architecture for agent credential management. It solves the "don't put secrets in environment variables" problem without requiring agents to understand a credential API. The HTTPS interception capability raises questions about certificate management and trust, though. The TypeScript (61.4%) + Rust (36.8%) split suggests the gateway is high-performance while the dashboard is pragmatically built with web tech. The main limitation is that it only works for HTTP-based APIs -- agents that need credentials for other protocols (database connections, SSH) need a different solution. Apache-2.0 license is good for adoption.

---
*Sources: [[summary/onecli]]*
*Last updated: 2026-05-14*
