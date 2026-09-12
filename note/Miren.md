# Miren

Miren is a self-hosted deployment platform — a PaaS you point at your own Linux servers instead of a managed cloud — that treats AI coding agents as first-class deploy users. It ships agent skills and predictable docs, declares deploy/databases/auth/previews in config, and gives every sandbox a verifiable OIDC workload identity. The homepage is thin marketing copy, but the positioning is a real signal: "your servers, your cloud, your call" plus an agent-native deploy surface.

---

## Key Quotes

> "Miren deploys your apps to any Linux server — bare metal or cloud VM. Miren Cloud connects your clusters into one control plane."

The sovereignty pitch, stated plainly. Most modern PaaS products (Render, Fly, Railway, Cloud Run) ask you to hand them the runtime; Miren's differentiation is that it deploys *to your* infra. It's the self-hosted pole of a market that has spent a decade drifting managed.

> "Miren ships with agent skills and predictable docs so your AI assistant can manage your entire deploy lifecycle."

The line that makes this wiki-relevant. "Predictable docs" is doing quiet work here — it means docs written so an LLM can navigate them deterministically, the same instinct behind [[10 Principles for Agent-Native CLIs]]. Miren isn't bolting AI on after the fact; it's designing the control surface for agents from the start.

> "Declare an addon, get a running database on deploy. Postgres, MySQL, Valkey, Memcached, RabbitMQ."

The addon model, straight from the Render/Fly playbook. Databases as declared config rather than provisioned infrastructure. Not novel, but it's the table stakes for any "one tool, every target" claim.

> "Every sandbox gets a verifiable OIDC token — services prove who they are without shared secrets."

The most technically interesting claim on the page. Shared-secret env vars are the default identity mechanism of every PaaS, and they're exactly the thing that leaks and rots. Per-sandbox OIDC workload identity is the direction the industry is already moving — see [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]] and [[NuGet API Key Lifetime Reduction]]'s push toward OIDC-based trusted publishing — and Miren making it a headline feature rather than a footnote is notable.

> "Expose internal admin endpoints without making them public. Built in, not bolted on."

The "admin API" pitch. A smaller point, but it signals the product's real audience: small teams running apps on their own boxes who still need internal-only operational endpoints.

---

## Key Themes

- **#tool** — A deployment PaaS in a crowded category (Render/Fly/Railway/Coolify), differentiated by self-hosting and agent-first UX.
- **#concept** — Workload identity: per-sandbox OIDC tokens as the replacement for shared secrets between services.
- **#pattern** — Declarative addons: databases and services materialized from config at deploy time.
- **#agent** — Agent-native deploy: skills + predictable docs so a coding agent owns the deploy lifecycle end to end.
- **#platform** — Self-hosted PaaS: "your servers, your cloud, your call" as a deliberate stance against managed-runtime lock-in.

---

## Critical Analysis

**This is marketing, not evidence.** The homepage is four screens of positioning copy with no pricing, no benchmarks, no code, no author, and no date. Every claim — "one tool, every target," "built in, not bolted on," the Cloud Run comparison — is a promise, not a demonstration. The three blog links suggest real engineering is happening (a custom RPC system, distributed runners, SQLite support), but the homepage itself is the weakest form of source: a product's own sales page.

**The AI-first framing is the genuine contribution — if it holds.** The wiki has been tracking the "agents need to deploy" thesis from two directions: Cloudflare making a managed platform *work for* agents via temporary accounts ([[Cloudflare Temporary Accounts for Agents]]), and Nubase/InsForge building backends *for* agents via MCP. Miren is a third position: a self-hosted PaaS that ships skills and docs as its agent integration surface. The bet is that an agent that can read predictable docs and invoke skills needs no special protocol — which is lighter than MCP but heavier than a `--temporary` flag.

**Workload identity is the sleeper differentiator.** "Verifiable OIDC token, no shared secrets" is one bullet on a marketing page, but it's a genuinely hard infrastructure problem that most PaaS products solve with a shrug and a `.env`. If Miren actually ships per-sandbox identity that services use to prove who they are to each other — not just to authenticate to the platform — that's a real architectural stance, and it dovetails with the broader industry turn away from long-lived shared secrets. The risk is that "workload identity" here means the platform signs a token and the rest is aspirational.

**The scope is ambitious for a deploy tool.** A custom RPC system, distributed runners across a cluster, maintenance windows, secrets, tasks, SQLite, plus databases/auth/previews — this is a lot of surface area for a product whose homepage doesn't even have a pricing page. The "one tool, every target" pitch is seductive, but the history of deploy tools ([[Building the deployment tool I wish I had]]) suggests the tools that stay small and honest about scope outlast the ones that try to be everything.

**The Cloud Run comparison is a positioning feint.** Comparing to a managed serverless runtime lets Miren claim "self-hosted, so no lock-in," but it sidesteps the products Miren actually competes with — Coolify, CapRover, Dokku — which already do "deploy to your own server." The real test is whether Miren's agent skills and workload identity justify existing over those mature, free alternatives.

---

## Cross-Links

- [[Nubase]] — the same "deploy platform an agent drives" thesis, from the managed/monolith direction rather than self-hosted/addons
- [[Cloudflare Temporary Accounts for Agents]] — the "an agent needs to deploy" thesis, solved with throwaway accounts instead of agent skills + self-hosting
- [[Building the deployment tool I wish I had]] — the minimalist pole of self-hosted deployment that Miren's "one tool, every target" maximalism stands against
- [[Platform Engineering as the AI Control Plane]] — Miren productizes the deploy slice of the control plane: skills, docs, and workload identity as the platform's agent-facing surface
- [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]] — the identity framework that per-sandbox OIDC tokens are an instance of
- [[InsForge]] — a BaaS for agents; Miren is the deploy-layer counterpart, bundling databases as addons rather than as a full managed backend

---

*Sources: [[raw/miren-dev]], [[summary/miren-dev]]*
*Last updated: 2026-09-04*
