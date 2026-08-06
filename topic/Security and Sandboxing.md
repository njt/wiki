# Security and Sandboxing

The core problem: you want agents powerful enough to be useful but constrained enough to be safe. Permission prompts fail because "a tired user" clicks allow on everything ([[yolo-cage]]). Prompt-level instructions degrade under context pressure ([[claude-ctrl]]). The only reliable approach is architectural constraint -- sandboxes that limit what the agent can do regardless of what it's been told to do, credential systems that never expose real secrets, and defense-in-depth that assumes every individual layer will be bypassed. The good news is that the OS-level primitives exist (Seatbelt, Landlock, seccomp, Firecracker). The bad news is that the agent-specific tooling is still immature and fragmented.

---

## The Landscape

### Execution Sandboxing

**OS-level primitives.** [[A Deep Dive on Agent Sandboxes]] reverse-engineers how Codex sandboxes agents: macOS Seatbelt for coarse-grained control, Linux Landlock + seccomp for granular access. The "Auto" default -- workspace edits allowed, network blocked -- is the sweet spot. Environment variable clearing prevents accidental credential exposure. Limitation: all-or-nothing network control means you can't allow npm but block everything else at the OS level.

**Container sandboxing.** [[yolo-cage]] runs Claude Code in Vagrant VMs with a dispatcher (enforces branch isolation, blocks dangerous git) and egress proxy (scans outbound traffic for secrets via TruffleHog). The "defer security to PR review" philosophy: don't try to make the agent safe in real time, just prevent it from pushing anything dangerous and let humans review at merge time. Heavy requirement: 8GB RAM, 4 CPUs minimum.

**Platform sandboxing.** [[OpenSandbox]] (Alibaba, 10.6k stars, CNCF Landscape) is the maximalist approach: Docker, Kubernetes, gVisor, Kata Containers, Firecracker microVMs, network policies, browser automation, desktop environments. Multi-language SDKs (Python, Java, Go, TypeScript, C#). The complete platform for production agent sandboxing.

**Unified control plane.** [[Navaris]] abstracts over Incus containers (~1s startup) and Firecracker microVMs (2-3s startup, hardware isolation) through a single API. Pick the isolation level you need without changing your code. Runtime resource resizing and time-bounded CPU/memory boosts are operationally useful features.

**Agent-native sandboxing.** [[VTcode]] (Rust) adds tree-sitter-based command validation on top of OS-level sandboxing -- parsing bash commands syntactically before deciding whether to allow execution. Deeper than pure filesystem/network controls because it understands what the command does, not just where it executes. [[Zeroclaw]] (31.3k stars, Rust) bakes supervised autonomy into the runtime: medium-risk = approval required, high-risk = blocked, with cryptographic tool receipts documenting every action.

**Execution-layer (syscall-level) enforcement.** [[agentsh]] sits *under* the agent at the syscall boundary (FUSE + eBPF + seccomp on Linux), governing not just the top-level command but every subprocess. Its distinguishing feature is `redirect` as a policy primitive: instead of denying and triggering retry loops, it transparently swaps commands, file paths, signals, DNS responses, and TCP connections. Includes an embedded LLM proxy with DLP, MCP-native security controls (tool whitelisting, version pinning, cross-server exfiltration blocking), and a profile-then-lock policy generation workflow. macOS support degrades because ESF doesn't support transparent file interception — redirect becomes "deny + guidance" which is semantically different.

### Credential Management

[[OneCLI]] is the cleanest solution: an HTTP gateway that intercepts outbound requests and transparently injects real credentials. Agents use placeholder keys and never touch secrets. AES-256-GCM at rest, host + path scoped, multi-agent support. "Like 1Password but for agents."

[[You Dont Want Long-Lived Keys]] provides the principle: ephemeral credentials sidestep the rotation problem entirely. SSH via EC2 Instance Connect, package publishing via trusted publishers, authentication via SSO. If you're building a new system, there's no excuse for long-lived credentials. This applies doubly to agents running in sandboxes -- the credential lifetime should match the agent session lifetime.

### Prompt-Level Defense

[[Bounding the Blast Radius — Prompt Injection Defenses]] is the definitive 2026 survey: a four-layer taxonomy (prompt formatting, model training, input filtering, trajectory monitoring) with the brutal finding that every layer collapses under adaptive attack. The only honest answer is economic — compose layers until cost-to-exploit exceeds value-at-risk.

[[LLM Guard]] sits between application and LLM with 15 input scanners and 20 output scanners: PII anonymization, prompt injection detection, secret scanning, bias detection, factual consistency checks. Defense in depth at the prompt/response boundary rather than the execution boundary. Complementary to sandboxing, not a substitute.

[[HackAPrompt Dataset]] provides 100K+ real prompt injection attempts from a global competition. The fundamental attack patterns (ignore previous instructions, context manipulation, role-playing exploits) remain durable across model generations. The practical value: train your own prompt injection detectors on real adversarial data.

### Enforcement via Hooks

[[claude-code-config (Trail of Bits)]] implements security as hooks: PreToolUse blocks dangerous commands, PostToolUse audits. Deny rules cover SSH keys, cloud credentials, package tokens, git credentials, crypto wallets. Devcontainer option for complete isolation.

[[claude-ctrl]] goes further with SQLite-backed policy evaluation and a "first-deny-wins" engine. The SESAP concept: probabilistic systems constrained to deterministically produce desired outcome ranges. At 1,566 commits, either deep conviction or endless yak-shaving.

## Key Tensions

**Usability vs. security.** Every security layer adds friction. [[yolo-cage]]'s 8GB RAM requirement pushes teams toward "just yolo it." [[A Deep Dive on Agent Sandboxes]]'s session-scoped trust lists require human judgment about what to trust. The tradeoff is real: overly restrictive sandboxes make agents useless, overly permissive ones make them dangerous. [[Cloudflare OS]] attacks this tension with simulated actions — Gatekeepers return fake results so the agent can keep working while side-effecting actions queue for async human approval, making the secure path the convenient path instead of the blocking one. The problem extends beyond agents: [[Security Is Hard, Y'all]] shows how even expert users cannot distinguish legitimate OAuth consent screens from phishing attacks when the signals are structurally identical — when security UX failures make real products indistinguishable from attacks, the usability-vs-security tradeoff has failed on both axes.

**Trust via permission prompts vs. trust via architecture.** [[yolo-cage]] says permission prompts fail because tired users click allow. [[claude-ctrl]] says instructions in context are not constraints. Both point to the same answer: move trust decisions from the human (unreliable under fatigue) to the architecture (reliable regardless). But architectural trust requires upfront investment that most teams skip.

**OS-level vs. application-level sandboxing.** OS primitives (Seatbelt, Landlock) are battle-tested but coarse. Application-level validation ([[VTcode]]'s tree-sitter parsing) is more precise but less proven. The right approach is both: OS as the outer boundary, application logic as the inner filter.

**Credential exposure vs. agent utility.** Agents need API keys to be useful. [[OneCLI]]'s proxy injection solves this for HTTP APIs. But database connections, SSH, and non-HTTP protocols still require direct credential access. [[You Dont Want Long-Lived Keys]] says make credentials ephemeral, but ephemeral credential infrastructure is its own engineering project.

**Emergent exploitation.** [[Benchmark Exploitation]] shows agents independently discovering privilege escalation exploits. [[HackAPrompt Dataset]] documents the attack patterns. As agents get more capable, they'll discover bypass strategies that weren't anticipated. The security model must assume adversarial capability even from "friendly" agents -- because capability produces exploitation as an emergent behavior, not an intentional one.

## What's Missing

**Access control models for agents.** The existing tooling focuses on sandboxing execution and managing credentials, but lacks a coherent authorization architecture for the agentic era. Cloudflare's [[The Agent Access Model]] proposes one: task-scoped credentials (RFC 8693 + DPoP), dual-boundary mediation (harness + network), and a Trust Ratchet that narrows the capability ceiling in-flight when protected data is accessed. Its Grant Review Loop addresses the operational problem of keeping task templates at true least privilege across many runs. AAM explicitly declines to solve the multiplayer case (agents serving multiple humans with different permissions), naming it an open systems problem.

**Network-level agent firewalls.** OS sandboxing gives all-or-nothing network control. [[agentsh]] partially addresses this with DNS and TCP connect redirect — transparently rerouting network traffic at the syscall level — but it's Linux-first and the feature set degrades on macOS.

**Cross-sandbox agent protocols.** If Agent A in Sandbox 1 needs to share a result with Agent B in Sandbox 2, how do they communicate securely? [[Navaris]] and [[OpenSandbox]] manage individual sandboxes but don't address inter-sandbox communication.

**Audit and forensics tooling.** [[VTcode]]'s cryptographic tool receipts and [[Zeroclaw]]'s action logging are steps in the right direction, but nobody has built the forensics toolkit for "what did the agent do during the 3 hours I was away?" Agent session forensics needs to be as mature as server access logging.

**Security evaluation frameworks.** [[Demystifying Evals for AI Agents]] covers functional evals. Security-specific evals -- can the agent be tricked into exfiltrating secrets? Can it be prompted to bypass its own constraints? -- are ad hoc at best. [[HackAPrompt Dataset]] is a starting point but targets the model, not the agent system.

**Observation-tracking security.** [[Cloudflare OS]] introduces a model where the platform records every resource an agent observes, attaches those observations to every artifact the agent produces (dashboards, documents, shared workspaces), and re-checks viewer permissions against the original resources at access time. This closes the chained-access gap: an agent with read-only DB access and a file-sharing tool can't exfiltrate data through legitimate operations if the platform tracks what was observed and enforces policy on downstream artifacts.

**MCP-native security.** [[agentsh]] is the first tool with MCP-specific security controls: tool whitelisting, version pinning for rug-pull detection, cross-server exfiltration blocking, and token bucket rate limiting. This addresses a gap that most sandboxing solutions don't even acknowledge, since MCP servers run with ambient trust.

**ANSI escape sequence injection (AESI).** [[ANSI Escape Sequence Injection in MCP Servers]] identifies a new MCP-specific injection class: ANSI escape codes that terminals render invisible are consumed as raw bytes by LLMs reading model-consumable fields. Stored AESI is particularly dangerous — a poisoned record persists across sessions and users, exploding the blast radius from one conversation to the entire knowledge graph. The detection challenge is that no static analysis can answer "what bytes does the live server actually return?", making DAST the only viable approach.

**Per-session Kubernetes isolation.** Ai2's Mothership platform provisions a dedicated Kubernetes deployment for each user session, with JWT injection at provision time and session-scoped files that are never shared. Designed for multi-tenant government use across 70+ countries where data isolation failures are career-ending. [[Building Shippy — Agent Architecture for High-Stakes Domains]]

**Nested sandbox defense in depth.** [[Building Agents That Don't Break Themselves]] argues that even an agent running inside a sandbox should dispatch commands to a *separate* sandbox — verified by Fly.io Sprite ID mismatches on return — so no agent can compromise its own execution environment even if it tries. The corollary: copy-on-write checkpointing before every risky step turns catastrophic self-destruction into a nine-second rollback.

## Key Themes

#sandboxing #credentials #prompt-injection #defense-in-depth #enforcement

## Pages

- [[matrix3]] — Tavis Ormandy's uMatrix successor for MV3: declarative content policy via CSP, proving the linter-not-prompts pattern in browser security
- [[A Deep Dive on Agent Sandboxes]] — How Codex sandboxes agent execution: Seatbelt, Landlock, seccomp
- [[yolo-cage]] — Agents that can't exfiltrate secrets or merge their own PRs. Vagrant + egress proxy
- [[Attack Review -- Claude Allowlisted-Egress Exfiltration]] — lhl's shisad framework vs. Claude Pirate and Cowork: four-stage kill chain analysis proving security is architecture, not patches
- [[OpenSandbox]] — Alibaba's sandbox platform for AI: Docker, Kubernetes, gVisor, Firecracker
- [[Navaris]] — Unified sandbox control plane: containers or microVMs, one API
- [[Crabbox]] — Agent workspace control plane: lease throwaway cloud machines without sharing provider credentials. Brokered provisioning, 16 providers, warm reuse, PR evidence artifacts
- [[Resident (ESP32 Sandbox)]] — Lua sandbox runtime for ESP32 devices built for AI agents. Inverts the sandbox metaphor: sandbox as host, not cage. Hermit crab architecture for agents inhabiting physical hardware
- [[LLM Guard]] — 35 scanners for prompt injection, data leakage, and toxicity
- [[OneCLI]] — Credential vault for agents: transparent proxy injection, no real keys exposed
- [[Stockyard]] — Jesse Vincent's Firecracker micro-VM orchestrator for coding agents: ZFS snapshots via vsock, Tailscale networking, 1Password-backed secrets
- [[HackAPrompt Dataset]] — 100K+ prompt injection attempts from a global hacking competition
- [[You Dont Want Long-Lived Keys]] — Ephemeral credentials sidestep the rotation problem entirely
- [[VTcode]] — Rust coding agent with OS-native sandboxing and comprehensive audit trails
- [[Clawpatrol]] — Deno's transparent L3 firewall for agents: WireGuard/Tailscale tunneling, protocol-aware CEL rules (SQL/K8s/SSH/HTTPS), credential injection at the wire, human+LLM approval chains. Single Go binary, SQLite-backed
- [[agentsh]] — Execution-layer security gateway: redirect instead of deny, FUSE+eBPF+seccomp, MCP-native security controls
- [[Project Glasswing — Mythos at Cloudflare]] — Cloudflare's nine-stage vuln discovery harness on 50+ repos: exploit chain stitching, adversarial review, and why patching faster is a trap
- [[Cloudflare Security Audit Skill]] — Cloudflare's open-source Claude Code skill for security auditing: the six-phase pipeline (recon → hunt → adversarial validation → report → structured output → independent verify) that seeded their internal Glasswing vulnerability harness. Pure prompt engineering; no static analysis
- [[audit (evilsocket)]] — Runnable MIT-licensed implementation of Cloudflare's Glasswing pipeline using Claude Code Agent SDK: 8 stages, 8 prompts, 9 schemas, SQLite state, concurrent agents
- [[AI Cybersecurity After Mythos — The Jagged Frontier]] — Fort tests Mythos's claims against cheap open-weights models: 8/8 detect the flagship exploit. The moat is the scaffold, not the model
- [[Cybersecurity Is Proof of Work Now]] — Security is a compute economics problem: outspend your attacker or stay vulnerable. Breunig's proof-of-work framing for the Mythos era
- [[An Illustrated Guide to OAuth]] — Visual explainer of the authorization code flow: every piece of OAuth's complexity closes a specific attack vector
- [[Hostnames and Usernames to Reserve]] — Which names to block on any user-registration platform: hostnames, emails, and URL paths that break protocol trust assumptions
- [[Supply Chain Security for Software Developers]] — The 7-day rule and layered defenses against package attacks after TeamPCP's March-April 2026 campaign
- [[You Should Not Update Your Dependencies in 2026]] — Olivier Gambier's case that dependency updates are now untrusted code contributions, Dependabot is an attack vector, and AI-in-CI is the only viable reviewer at scale
- [[Zero Trust for AI Agents]] — Anthropic's definitive security framework: three-tier maturity model across eight capability domains. The "impossible vs. tedious" design test, Least Agency, and why rotating API keys is security theater
- [[AI Agents Are Installing Packages No One Owns]] — Hallucinated package names spreading through agent skill files into 237+ repos; the accountability gap when AI agents install deps no human approved
- [[Tom Barraclough — Sovereign AI Policy]] — confidential computing as personal sovereignty: Apple Private Cloud Compute and Tinfoil offer hardware-verifiable privacy through cryptographic attestation, extending trust from "we promise" to "the silicon proves it"
