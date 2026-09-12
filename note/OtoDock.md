# OtoDock

OtoDock is a self-hosted "agentic company OS" that turns Claude Code and Codex into a multi-user platform of persistent, sandboxed digital employees. Where [[QM (Multiplayer Agent Harness)]] builds the same category as MIT-licensed infrastructure with a harness-swapping core, OtoDock is the productized version: departments, per-agent roles, four workspace-sharing modes, scheduled/webhook automation, phone numbers, and a kernel-sandbox security story, all running on *your* Anthropic or OpenAI subscription. It occupies the middle ground between the single-user [[Personal Agents]] movement and the cloud-hosted, org-wide [[Cloudflare OS]].

---

## Key quotes

> "The brains of your company, built on Claude Code & Codex, working on your Anthropic and OpenAI subscriptions."

The positioning is deliberate: OtoDock does not resell tokens. It is a shell around engines you already pay for, which sidesteps the per-seat LLM-cost problem that sinks most agent startups and makes the Fair Source "self-host free up to 5 users" economics actually work — the expensive part is your subscription, not their software.

> "Agents are powerful, so OtoDock assumes they can't be trusted."

This is the sharpest line on the page and the whole security thesis in one sentence. It is the inverse of the ambient-trust default in most agent harnesses, and it lands squarely inside [[Security and Sandboxing]]'s central argument that the boundary belongs below the model.

> "Each session runs in its own mount and process namespace. Folders are mounted automatically from each user's role per agent."

The sandbox is not a Docker container or a VM — it is Linux namespaces (mount + process) per session, with role-derived folder mounts. Lighter than a microVM, heavier than a bare shell, and interestingly close to how [[How We Contain Claude]] thinks about isolation tiers.

> "Credentials are encrypted at rest and injected only per session. Agents can use them, but never see them."

The credential-injection pattern from the network-layer tools in [[Security and Sandboxing]] ([[Clawpatrol]], [[OneCLI]], [[Corsair — Agent Integration Layer]]) — but shipped as a default feature of a self-hosted product, not as infrastructure you assemble yourself.

> "This entire video was directed, captured and edited by an OtoDock agent."

A small thing, but it is the product eating its own dog food in the most legible way possible, and a telling signal that the "digital employee" framing is meant literally rather than as metaphor.

## Key themes

- **#tool** — OtoDock: a self-hosted, multi-user platform wrapping Claude Code and Codex as interchangeable engines on your own subscriptions
- **#concept** — "Agentic company OS": departments of agents with delegation rules, meetings, and a two-level role system (platform: Admin/Creator/Member; per-agent: Manager/Editor/Viewer)
- **#pattern** — Locked-down-by-default security: kernel namespaces + network isolation + per-session credential injection, granted one service at a time
- **#pattern** — Four workspace-sharing modes (personal only / personal + shared / shared + personal / shared only) as the answer to "one agent, many people"
- **#concept** — Fair Source licensing: public source, free self-host under 5 users, each release converts to Apache 2.0 after two years

## Analysis

**The real novelty is the middle position, not any single feature.** Individually, every piece exists elsewhere: [[QM (Multiplayer Agent Harness)]] has scoped memory/files/keychain/crons/sandbox and the same "company infrastructure, not personal assistant" framing; [[Cloudflare OS]] has per-user sandboxes and default-deny capability access; [[Personal Agents]] catalogs a dozen self-hosted assistants with persistent memory. What OtoDock adds is the *combination* — a productized, self-hosted, multi-user agent platform with a genuine kernel-sandbox story, BYO-subscription economics, and a company-organizational model (departments, delegation, roles) that the personal-agent tools have never attempted. It is the "missing middle" that [[Personal Agents]] itself flags under *What's Missing*: "multi-user personal agents" need shared context with private compartments, delegated authority, and audit trails — OtoDock is a direct attempt to build exactly that.

**The subscription passthrough is the strategic insight, and the strategic risk.** By running on the customer's own Claude Pro/Max or ChatGPT subscription (or their API key), OtoDock avoids the hardest problem in agent-infrastructure economics: per-user inference costs that erase margins ([[Unit Economics of AI Software]] names this as the structural squeeze on SaaS). But it also means the platform's ceiling is set by what a consumer or pro subscription permits — rate limits, usage caps, and terms of service that were never written for a company running departments of autonomous agents. "Pick per agent, switch per chat" is genuinely nice for avoiding vendor lock-in, but it quietly makes the whole platform hostage to two vendors' consumer-plan rules.

**The sandbox claim deserves skepticism, not dismissal.** Mount + process namespaces with network isolation is a real isolation boundary — it is the same primitive family as [[A Deep Dive on Agent Sandboxes]] documents for Codex CLI — but namespaces are not a security boundary against a motivated kernel-exploit attacker the way a microVM is, and the page doesn't say what happens on the "full access" remote machines (laptop/workstation/PC), where the sandbox is presumably off by default. The honest reading: OtoDock ships a meaningful-but-not-absolute isolation layer for server agents, and leans on "your hardware, your data" for the rest. That is a reasonable trade for a self-hosted tool; it is not the same guarantee as [[SmolVM]]-class hardware isolation.

**Fair Source is a licensing move worth watching.** "Public source, converts to Apache 2.0 after two years" is a maturation play — the Functional Source License pattern — that lets OtoDock sell seats to growing teams now while promising the code eventually becomes truly open. It is a clever hedge against the [[Zero-Cost Fallacy of Open Source]] (charge for the software, not the inference) while still letting a solo user self-host for free. Whether the "growing teams license by seats" line survives contact with the five-free-users floor is the open question.

**What's missing from the page.** No mention of how agents are *evaluated* — the same gap [[Personal Agents]] names for the whole category — and no evidence that a department of autonomous agents actually works beyond the demo video. The "agent meeting" concept is a provocative idea (multi-agent deliberation as a management metaphor) but the page offers nothing on what actually happens in one. Treat the whole thing as a well-articulated vision with a credible security posture and an unproven operational record.

## Related pages

- [[QM (Multiplayer Agent Harness)]] — the closest sibling: same self-hosted, multi-user, per-scope memory/files/keychain/crons/sandbox shape, built as MIT infrastructure rather than a product
- [[Personal Agents]] — the single-user movement OtoDock extends; it directly addresses that hub's flagged "multi-user personal agents" gap
- [[Cloudflare OS]] — the cloud-hosted, org-wide "agentic OS" at the other end of the spectrum; OtoDock is the self-hosted, BYO-model counterpart
- [[Security and Sandboxing]] — the kernel-namespace, network-isolation, credential-injection story is a productized instance of that hub's core thesis
- [[Munder Difflin — Clones of You, Not a Shared Bot]] — the opposite pole: distributed per-user clones on your own hardware versus OtoDock's centralized self-hosted server

---

*Sources: [[raw/otodock-io]], [[summary/otodock-io]]*
*Last updated: 2026-09-11*
