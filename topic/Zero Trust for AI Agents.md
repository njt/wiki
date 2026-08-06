# Zero Trust for AI Agents

Anthropic's definitive security framework for deploying autonomous AI agents in the enterprise, published May 2026. A three-tier maturity model (Foundation/Enterprise/Advanced) across eight security capability domains, grounded in the Zero Trust principles of never-trust-always-verify, assume-breach, and least privilege. The document's real contribution is operationalizing the "impossible vs. tedious" design test: controls that rely on friction (rate limits, extra hops, non-standard ports) degrade to zero against agentic attackers with unlimited patience and near-zero per-attempt cost. Only hard barriers — cryptographic identity, expiring tokens, network paths that don't exist — survive.

---

## The "Impossible vs. Tedious" Test

This is the sharpest idea in the document and the lens through which every tier recommendation is evaluated:

> "When you evaluate any control in this document, ask a single question: does this make the attack impossible, or just tedious? Mitigations whose value comes from friction rather than a hard barrier — including extra pivot hops, rate limits, non-standard ports, and SMS-based MFA — degrade significantly against an adversary that can grind through tedious steps at scale. Agentic attackers have unlimited patience and near-zero per-attempt cost."

This reframes the entire security conversation. Rate limiting isn't defense — it's a stopwatch. The countermeasures that survive the test all share a pattern: hardware-bound credentials, expiring tokens, cryptographic identity, and network paths that do not exist rather than paths that are merely inconvenient. It's the security equivalent of "linters beat prompts" ([[Guardrails and Feedback Loops]]) — deterministic architectural constraints beat probabilistic behavioral ones.

> "If you are running API keys with rotation policies today, treat it as a known gap rather than a legitimate Foundation posture. Rotating a credential that can be grepped out of a lockfile does not raise the cost to an AI-assisted attacker meaningfully."

## Least Agency

OWASP's new term, which the document adopts as foundational:

> "Least agency goes further, restricting what each agent tool can do, how often, and where. In practice: a database tool gets read-only queries, an email summarizer gets no send/delete rights, an API gets minimal CRUD operations."

This matters because traditional access controls are blind to tool-level semantics. An agent with read-only database access and an email tool with send permission can be tricked into chaining them — reading customer data and emailing it out — using only legitimate, individually-authorized operations. Host-centric monitoring sees no malware because every command executes through trusted binaries under valid credentials. Least Agency means constraining at the tool-action level, not the credential level. [[Cloudflare OS]] operationalizes this at the platform level: its Gatekeepers record every resource an agent observes, attach those observations to every downstream artifact, and re-check viewer permissions when anyone tries to access the agent's output — closing the chained-access gap that credential-level controls alone cannot address.

## Three-Tier Framework

The tier structure is unusually honest about its own assumptions:

| Tier | Posture | Who It's For |
|------|---------|-------------|
| Foundation | Short-lived tokens, crypto identity, identity-based isolation, automated first-pass triage | Small teams, initial deployments |
| Enterprise | mTLS, ABAC, statistical anomaly detection, sandboxed execution, immutable audit | Most organizations with significant deployments |
| Advanced | Hardware-bound credentials, continuous auth, ML behavioral analysis, self-healing, confidential computing | Regulated industries, national security, high-stakes |

> "Expect the Advanced tier to become Enterprise standard as the space evolves, and Enterprise to become Foundation."

The Foundation floor has been raised — friction-only controls no longer qualify. Static API keys and shared service-account passwords are "among the first things an attacker with model-assisted code analysis will find; they are no longer a legitimate entry point, not even at Foundation."

## Supply Chain Reality

The most actionable section for practitioners:

> "Most large codebases accumulate multiple libraries doing the same job (several HTTP clients, several JSON parsers), each adding an attack surface for no functional gain. A one-hour dependency-tree audit — pointing a frontier model at your lockfile and asking which dependencies overlap and what migration would look like — often surfaces consolidation worth doing."

And the killer stat: **250 malicious documents** can backdoor LLMs from 600M to 13B parameters, and those backdoors **persist through safety training including supervised fine-tuning and RLHF**. This directly connects to the [[Supply Chain Security for Software Developers]] and [[You Should Not Update Your Dependencies in 2026]] arguments — the threat isn't theoretical, and the economics now favor model-assisted discovery of pre-existing vulnerabilities at scale.

> "For small dependencies that score poorly on Scorecards and are not actively maintained, having a frontier model reimplement the subset of functionality you actually use is often safer than continuing to depend on them."

## Prompt Injection: Direct vs. Indirect

The document draws a clean distinction that most security discussions blur:

- **Direct injection**: Attacker crafts input that overrides system instructions. Algorithmic approaches achieve 100% success with prompts that transfer across model families.
- **Indirect injection**: The more insidious threat. Attackers embed malicious instructions in external data (web pages, emails) that agents process. "The user never sees the malicious payload, and the agent executes it as if it were a legitimate request."

> "Microsoft's Spotlighting technique reduces indirect injection attack success from over 50% to under 2% by clearly delimiting untrusted content."

Anthropic's own constitutional classifiers blocked 95% of jailbreak attempts with minimal over-refusal. The combination of spotlighting (input-level) + constitutional classifiers (model-level) is the closest thing to a working defense stack against prompt injection.

## Defensive Operations at AI Speed

> "The answer is not to remove humans from the loop — it is to move humans off the bookkeeping and onto the decisions."

This is the right framing. Automate evidence collection, enrichment, correlation, and documentation. Keep humans on containment calls, disclosure calls, and customer-comms calls. Human decision speed during an incident should never be rate-limited on evidence collection or write-ups.

> "Put a model at the front of your alert queue. Every inbound alert should get an automated first-pass investigation before a human sees it."

Two metrics to instrument before anything else: **dwell time** (anomaly occurrence to human awareness) and **coverage** (fraction of alerts actually investigated). These are the two metrics AI automation has the greatest leverage to move.

## Critical Analysis

**What's genuinely new**: The "impossible vs. tedious" test is a clean, portable heuristic that cuts through most security theater. The three-tier structure with explicit capability tables gives CISOs a roadmap they can hand to engineering. The eight-phase implementation workflow (Part IV) is concrete enough to follow without a consultant. Least Agency as an explicit extension of Least Privilege names a real gap in existing security models.

**What's not addressed**: The document is silent on cost. Deploying hardware-bound credentials, continuous authorization, and confidential computing across an agent fleet isn't free — and for organizations that can't afford Advanced, the implicit message is "accept the risk." There's no discussion of what a reasonable interim posture looks like for cash-constrained teams.

**The Claude Code problem**: Every section includes a "Pro-tip" callout showing how Claude Code supports the capability. Some of these are legitimate (sandboxing, OAuth for MCP, session isolation). Others feel like product marketing masquerading as security guidance — particularly when the "pro-tip" amounts to "configure settings.json." The document is stronger when it stays vendor-neutral; the Claude Code examples are useful but should probably have been a separate appendix.

**What it gets right**: The insistence that Foundation means *cryptographic* identity, not just labeled identity. The recognition that rotating API keys is security theater. The framing of supply chain risk as *runtime composition* (agents loading tools dynamically) rather than static dependency analysis. The emphasis on dwell time and coverage as the two metrics that matter. The acknowledgment that defensive agents need the same Zero Trust treatment.

**The gap**: No treatment of cost, no threat modeling methodology, no discussion of what "good enough" looks like for teams that can't reach Enterprise tier. The framework assumes organizational infrastructure (SIEM, SOAR, PKI, HSMs) that many teams don't have. The "just use hardware-backed credentials" advice is correct but requires infrastructure most organizations haven't built.

---

## Key Themes

#zero-trust #agent-security #least-agency #supply-chain #prompt-injection #credential-management #sandboxing #observability #behavioral-monitoring #framework

---

## Related

- [[Security and Sandboxing]] — Synthesis page covering the same domain from the tooling side
- [[Supply Chain Security for Software Developers]] — The 7-day rule and layered defenses
- [[You Should Not Update Your Dependencies in 2026]] — Gambier's case that every dep update is untrusted
- [[yolo-cage]] — Practical sandbox implementation with egress proxy
- [[OneCLI]] — Credential vault with transparent proxy injection
- [[claude-ctrl]] — "An instruction in context is not a constraint"
- [[A Deep Dive on Agent Sandboxes]] — Codex sandbox architecture (Seatbelt, Landlock, seccomp)
- [[You Dont Want Long-Lived Keys]] — Ephemeral credentials sidestep rotation entirely
- [[LLM Guard]] — 35 scanners for prompt injection and data leakage
- [[HackAPrompt Dataset]] — 100K+ real prompt injection attempts
- [[agentsh]] — Execution-layer security gateway with redirect primitives
- [[Clawpatrol]] — Deno's transparent L3 firewall for agents
- [[Guardrails and Feedback Loops]] — Linters beat prompts; deterministic enforcement
- [[Cybersecurity Is Proof of Work Now]] — Security as compute economics
- [[AI Cybersecurity After Mythos — The Jagged Frontier]] — The moat is the scaffold, not the model
- [[Project Glasswing — Mythos at Cloudflare]] — Nine-stage vuln discovery harness

---

*Source: [Anthropic — Zero Trust for AI Agents](https://cdn.prod.website-files.com/6889473510b50328dbb70ae6/6a1611a04085d7cd3dadc924_Claude-eBook-Zero-Trust-for-AI-Agents-05182026.pdf), published 2026-05-18. Fetched 2026-05-31.*
