# How We Contain Claude

Anthropic's own engineering team describes the containment architecture across three Claude products — claude.ai (gVisor containers), Claude Code (OS-level sandbox), and Claude Cowork (sealed VM) — and the incidents they didn't anticipate. This is the most candid public document on AI containment engineering: rather than a security white paper, it's a postmortem-adjacent tour of what broke, why it broke, and what principle would have prevented it.

---

## Key Quotes

> "Battle-tested hypervisors, syscall filters, and container runtimes have survived more adversarial attention than anything you'll build."

The article's thesis statement, repeated across every deployment described. The standard primitives held. The custom code around them — the proxy in claude.ai, the trust dialog in Claude Code, the egress logic in Cowork — is where every incident occurred. This is a humility principle masquerading as engineering advice.

> "Users approved roughly 93% of permission prompts."

The number that killed the permission-prompt model. When your users are clicking "allow" on 93% of prompts, you don't have a security boundary — you have a consent theater with a 7% refusal rate. The 84% reduction in prompts after sandboxing isn't just a UX win; it's the difference between a security boundary and a speed bump. This directly vindicates Freeman's argument in [[A Deep Dive on Agent Sandboxes]] that default-deny at the OS level is the only approach that scales.

> "When the user is the one typing the instruction, there's nothing anomalous for a classifier to catch."

The user-as-injection-vector insight. A phished employee typing a malicious prompt looks identical to legitimate use from the model's perspective. This is why model-layer defenses are structurally incapable of catching certain attacks — not because the classifier isn't good enough, but because the attack is indistinguishable from authorized use. The fix has to be environmental: egress controls, filesystem boundaries, credential hygiene.

> "An audited connector isn't the same as audited data."

The GitHub-MCP-connector paradox. You can audit every line of the connector code and still get owned by a poisoned README that passes through it. Traditional supply-chain security addresses code execution risk but is blind to prompt injection. The recommendation — live inspection of tool return values by a small fast classifier before they enter the model's context — is a genuinely novel architectural primitive.

> "The egress proxy checked the destination, saw api.anthropic.com, and let it through."

The allowlisted-egress vulnerability described from the inside. This is the same attack that [[Attack Review -- Claude Allowlisted-Egress Exfiltration]] analyzes from the outside. Anthropic's fix — a defensive MITM proxy inside the VM that only passes requests with the VM's own provisioned session token — is clever but creates the monitoring blind spot that enterprise security teams flagged when VM isolation blocked their EDR.

> "The standard primitives held while our own work around them exposed flaws."

The meta-lesson, stated plainly. Every incident in the article follows this pattern: gVisor held, the custom proxy didn't. Seatbelt held, the trust-dialog timing didn't. The hypervisor held, the egress allowlist didn't. If you're building containment for agents, write as little custom security code as possible, and expect the code you do write to fail.

---

## The Three Isolation Patterns

**claude.ai — Ephemeral Container.** Server-side gVisor containers with per-session ephemeral filesystems. The traditional threat model: protect Anthropic's infrastructure, isolate tenants. Lowest user-facing complexity because the user never sees the sandbox. The custom proxy around gVisor was the failure point in "our most consequential incident" (details not disclosed). Google's [[Agent Substrate]] uses the same gVisor checkpoint/restore for a different axis: not just isolation, but density — packing hundreds of sandboxed agents onto a handful of hosts by hibernating idle ones.

**Claude Code — HITL Sandbox.** OS-level sandboxing (Seatbelt on macOS, bubblewrap on Linux) with human-in-the-loop for bash and network. The key evolution: from permission prompts (93% approval rate) to sandbox defaults (84% fewer prompts). Two missed risks: pre-trust-dialog hook execution (3 vulns, mid-2025 to Jan 2026) and user-as-injection-vector (employee phish, Feb 2026, 24/25 successful exfiltrations). The sandbox runtime is [open source](https://github.com/anthropic-experimental/sandbox-runtime).

**Claude Cowork — Sealed VM.** Full VM via platform hypervisor (Apple Virtualization framework, HCS on Windows). Designed for knowledge workers who can't evaluate bash commands. Agent loop moved outside the VM while code execution stays inside — a pragmatic compromise after the "full-VM mode" proved brittle. The allowlisted-egress vulnerability and the EDR blind spot are the two documented incidents.

---

## Themes

#security #sandboxing #agents #Claude #prompt-injection #isolation #containment

**Environment-first defense.** The article's central architectural claim: model-layer defenses are a supplement, not a foundation. Both major incidents involved egress through permitted paths where no classifier could have helped. The environment layer — sandboxes, VMs, egress controls, credential boundaries — is the only layer that can enforce properties regardless of what the model decides to do. This aligns with [[claude-ctrl]]'s SESAP concept and the default-deny philosophy in [[A Deep Dive on Agent Sandboxes]].

**Custom code as the vulnerability surface.** Every incident traces to custom components built around battle-tested primitives. This inverts the usual security intuition: you'd expect the complex kernel-level code to be where bugs live, but it's the application-level glue — proxies, dialogs, allowlists — that breaks. Anthropic's lesson is essentially "use as little of your own security code as possible," which is both correct and uncomfortable for a company whose product IS custom AI infrastructure.

**Three products, three threat models.** The article's structure reveals a design philosophy: match isolation to user capability, not to the product name. A developer who reads bash gets a sandbox with approval gates. A knowledge worker who can't gets a sealed VM with no approval surface at all. The server-side product gets traditional container isolation because there's no user machine to protect. This is threat modeling as product design, not as compliance exercise.

---

## Critical Analysis

This article is remarkable less for what it says than for what it represents: a major AI company publicly documenting its security incidents, including specific dates, attack vectors, and failure modes. The February 2026 employee phish — 24/25 successful exfiltrations — is the kind of detail most companies bury in internal postmortems. Publishing it signals either extraordinary confidence or a bet that transparency is the better strategy when your product's security depends on architectural choices users need to understand.

**What's missing.** The article never discloses the "most consequential incident" involving claude.ai's custom proxy. It describes the phish and the allowlist disclosure in detail but redacts the server-side incident entirely. Given the pattern — custom proxy code failing around battle-tested gVisor — this is probably the most interesting case, and we don't get it. The OTLP-based logging mitigation for the EDR blind spot is described as "not the same as live monitoring," which is an honest admission that the problem isn't solved.

**The persistent memory vector is under-explored.** The article flags persistent memory poisoning as an emerging risk but gives it only a paragraph. This is arguably the hardest problem: if an injection lands in CLAUDE.md or a memory file, it fires on every subsequent session with no further attacker action. The comparison to APT persistence is exact but the industry has no equivalent of endpoint detection for agent memory. [[Agent Identity]] touches on this from the participation angle, and [[Akmon]] provides tamper-evident logging, but neither addresses the detection problem. [[ANSI Escape Sequence Injection in MCP Servers]] generalizes this into a named attack class (stored AESI) with a concrete DAST methodology for empirically mapping the write-to-read paths that make persistence exploitable.

**The "user as injection vector" problem is existential for HITL models.** If phishing an employee into pasting a malicious prompt works 96% of the time and model-layer defenses are structurally blind to it, then the entire human-in-the-loop security model has a hole that no amount of classifier improvement can patch. The only fix is environmental — egress controls, credential boundaries, filesystem isolation — which means HITL is a UX pattern, not a security pattern. This is the most uncomfortable claim in the article, and it's correct.

**The tool side is just as leaky.** Alan Smith demonstrated the same vulnerability from the other direction in [[AI Agents In-Depth — Function Calling, MCP and Tool Use Under the Hood]]: a system prompt containing a credit card number, and a tool accepting a credit card parameter — the LLM passed the number through "without batting an eyelid." When he switched providers (OpenAI → Azure Foundry), the behaviour changed — sometimes it refused, sometimes it didn't — but the non-determinism means you can't rely on refusal. The lesson is the same as this article's: model-layer defenses are probabilistic; environmental controls (not passing sensitive data in context, tool-scoped credentials) are deterministic. [[Corsair — Agent Integration Layer]] operationalizes that deterministic fix: the agent never receives credentials at all (envelope-encrypted, resolved at call time) and destructive calls land in a permission database the agent can't reach, so approval is a single-use expiring record rather than a prompt the user auto-approves 93% of the time.

**Comparison to the broader landscape.** [[A Deep Dive on Agent Sandboxes]] describes Codex's OS-level sandboxing, which is architecturally similar to Claude Code's approach. [[Zeroclaw]] bakes supervised autonomy into the runtime. [[yolo-cage]] wraps Claude Code itself in Vagrant VMs. [[OpenSandbox]] is the maximalist platform approach. Anthropic's contribution isn't a novel sandboxing technique — it's the candid taxonomy of failure modes across three different isolation strengths, and the principle that environment-first defense is the only defense that survives contact with real attackers. [[Bounding the Blast Radius — Prompt Injection Defenses]] provides the theoretical backing: the Nasr et al. (2025) finding that every defense collapses under adaptive attack explains *why* Anthropic's layered approach is necessary, and the economic reframe (cost-to-exploit > value-at-risk) gives a language for deciding when the layers are sufficient.

---

*Sources: [[summary/how-we-contain-claude]]*
*Last updated: 2026-06-15*
