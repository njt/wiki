# OpenSandbox

Alibaba's general-purpose sandbox platform for AI applications. Multi-language SDKs (Python, Java, Go, TypeScript, C#), unified APIs, and Docker/Kubernetes runtimes for coding agents, GUI agents, evaluation, code execution, and RL training. 10.6k GitHub stars and listed in the CNCF Landscape.

---

## Key Themes

#sandboxing #security #agent-architecture

OpenSandbox is the maximalist approach to agent sandboxing: not just process isolation, but a full platform with lifecycle management, network policy controls, browser automation (Chrome, Playwright), desktop environments (VNC, VS Code), and a code interpreter SDK.

The isolation options span the spectrum: gVisor for lightweight sandboxing, Kata Containers for stronger isolation, and Firecracker microVMs for full hardware-level separation. The network policy management (ingress gateway, egress controls) addresses the "agent can make arbitrary network calls" problem that simpler sandboxes ignore.

The MCP server integration is notable -- agents using Claude Code or Cursor can manage sandboxes programmatically through the Model Context Protocol, creating a recursive loop where agents create sandboxes to run other agents or untrusted code.

The CLI (`osb`) covers creation, execution, file operations, and diagnostics. The Sandbox Protocol spec defines lifecycle and execution APIs, which is useful for building compatible implementations.

Contrast with [[Navaris]] (lighter, focused on container vs. microVM abstraction without the full platform), [[OneCLI]] (credential management to pair with sandbox execution), and [[llm-guard]] (prompt-level defense vs. execution-level isolation).

## Critical Analysis

The breadth is both strength and weakness. 10.6k stars and CNCF listing suggest real adoption, but the feature surface is enormous. The multi-language SDK support is genuinely useful -- most sandbox tools are Python-only, which excludes the Java and C# ecosystems. The Docker + Kubernetes runtime support means it fits into existing infrastructure. The main concern is operational complexity -- running OpenSandbox with Firecracker and network policies is significantly more complex than spinning up a container with `--read-only`. For simple "run untrusted code" use cases, this might be overkill; for production agent platforms that need full lifecycle management, it's the most complete option available.

---
*Sources: [[summary/opensandbox]]*
*Last updated: 2026-05-14*
