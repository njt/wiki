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

### Credential Management

[[OneCLI]] is the cleanest solution: an HTTP gateway that intercepts outbound requests and transparently injects real credentials. Agents use placeholder keys and never touch secrets. AES-256-GCM at rest, host + path scoped, multi-agent support. "Like 1Password but for agents."

[[You Dont Want Long-Lived Keys]] provides the principle: ephemeral credentials sidestep the rotation problem entirely. SSH via EC2 Instance Connect, package publishing via trusted publishers, authentication via SSO. If you're building a new system, there's no excuse for long-lived credentials. This applies doubly to agents running in sandboxes -- the credential lifetime should match the agent session lifetime.

### Prompt-Level Defense

[[LLM Guard]] sits between application and LLM with 15 input scanners and 20 output scanners: PII anonymization, prompt injection detection, secret scanning, bias detection, factual consistency checks. Defense in depth at the prompt/response boundary rather than the execution boundary. Complementary to sandboxing, not a substitute.

[[HackAPrompt Dataset]] provides 100K+ real prompt injection attempts from a global competition. The fundamental attack patterns (ignore previous instructions, context manipulation, role-playing exploits) remain durable across model generations. The practical value: train your own prompt injection detectors on real adversarial data.

### Enforcement via Hooks

[[claude-code-config (Trail of Bits)]] implements security as hooks: PreToolUse blocks dangerous commands, PostToolUse audits. Deny rules cover SSH keys, cloud credentials, package tokens, git credentials, crypto wallets. Devcontainer option for complete isolation.

[[claude-ctrl]] goes further with SQLite-backed policy evaluation and a "first-deny-wins" engine. The SESAP concept: probabilistic systems constrained to deterministically produce desired outcome ranges. At 1,566 commits, either deep conviction or endless yak-shaving.

## Key Tensions

**Usability vs. security.** Every security layer adds friction. [[yolo-cage]]'s 8GB RAM requirement pushes teams toward "just yolo it." [[A Deep Dive on Agent Sandboxes]]'s session-scoped trust lists require human judgment about what to trust. The tradeoff is real: overly restrictive sandboxes make agents useless, overly permissive ones make them dangerous.

**Trust via permission prompts vs. trust via architecture.** [[yolo-cage]] says permission prompts fail because tired users click allow. [[claude-ctrl]] says instructions in context are not constraints. Both point to the same answer: move trust decisions from the human (unreliable under fatigue) to the architecture (reliable regardless). But architectural trust requires upfront investment that most teams skip.

**OS-level vs. application-level sandboxing.** OS primitives (Seatbelt, Landlock) are battle-tested but coarse. Application-level validation ([[VTcode]]'s tree-sitter parsing) is more precise but less proven. The right approach is both: OS as the outer boundary, application logic as the inner filter.

**Credential exposure vs. agent utility.** Agents need API keys to be useful. [[OneCLI]]'s proxy injection solves this for HTTP APIs. But database connections, SSH, and non-HTTP protocols still require direct credential access. [[You Dont Want Long-Lived Keys]] says make credentials ephemeral, but ephemeral credential infrastructure is its own engineering project.

**Emergent exploitation.** [[Benchmark Exploitation]] shows agents independently discovering privilege escalation exploits. [[HackAPrompt Dataset]] documents the attack patterns. As agents get more capable, they'll discover bypass strategies that weren't anticipated. The security model must assume adversarial capability even from "friendly" agents -- because capability produces exploitation as an emergent behavior, not an intentional one.

## What's Missing

**Network-level agent firewalls.** OS sandboxing gives all-or-nothing network control. Nobody has built the agent-aware network proxy that allows specific API endpoints (npm registry, GitHub) while blocking everything else, with semantic understanding of what the agent is trying to do.

**Cross-sandbox agent protocols.** If Agent A in Sandbox 1 needs to share a result with Agent B in Sandbox 2, how do they communicate securely? [[Navaris]] and [[OpenSandbox]] manage individual sandboxes but don't address inter-sandbox communication.

**Audit and forensics tooling.** [[VTcode]]'s cryptographic tool receipts and [[Zeroclaw]]'s action logging are steps in the right direction, but nobody has built the forensics toolkit for "what did the agent do during the 3 hours I was away?" Agent session forensics needs to be as mature as server access logging.

**Security evaluation frameworks.** [[Demystifying Evals for AI Agents]] covers functional evals. Security-specific evals -- can the agent be tricked into exfiltrating secrets? Can it be prompted to bypass its own constraints? -- are ad hoc at best. [[HackAPrompt Dataset]] is a starting point but targets the model, not the agent system.

## Key Themes

#sandboxing #credentials #prompt-injection #defense-in-depth #enforcement

---
*Synthesis of: [[A Deep Dive on Agent Sandboxes]], [[yolo-cage]], [[OpenSandbox]], [[Navaris]], [[LLM Guard]], [[OneCLI]], [[HackAPrompt Dataset]], [[You Dont Want Long-Lived Keys]], [[VTcode]], [[claude-code-config (Trail of Bits)]], [[claude-ctrl]], [[Zeroclaw]], [[Benchmark Exploitation]]*
*Last updated: 2026-05-14*
