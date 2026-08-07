# SmolForge

A fully-realized GitHub clone built entirely on Cloudflare infrastructure — Workers, D1, R2, and Durable Objects — that treats AI coding agents as first-class platform citizens. It provides Git hosting, issues, pull requests, CI/CD, deploy previews, repository agents, AI transcript storage, and a comprehensive REST API, all running on serverless primitives. Built by swyx (Shawn Wang) and self-hosted on its own platform at `forge.smol.ai/swyx/forge`.

---

## Key Quotes

> "SmolForge is a GitHub clone built entirely on Cloudflare infrastructure (Workers, D1, R2, Durable Objects)."

This isn't a toy. The entire surface area of GitHub — repos, commits, branches, issues, PRs, CI/CD, gists, wikis, orgs, webhooks — is implemented on serverless primitives. The architecture is the argument: Cloudflare's platform can host a competitive code forge. The source is self-hosted on the platform it provides, which is both meta and a genuine integration test.

> "Every Forge repository has its own durable, multi-turn agent authority."

This is the feature that distinguishes SmolForge from every other Git host. Repository agents aren't bolted on — they're a platform primitive. Every repo gets its own agent. The Phase 1 "Instant" profile is read-only and bounded (one exact SHA, no shell, no network, no secrets), but the architecture is designed for escalation to Workspace agents that can write branches and open PRs. This is code review infrastructure, not a chatbot feature.

> "Forge Deploy adds a Git-native release control plane above them: one exact pushed Git SHA identifies the source."

The deploy model is the strongest technical argument in the docs. Cloudflare provides runtime, storage, and networking. Forge adds immutable previews, SHA-gated activation, pointer-based rollback, and a deployment evidence chain that joins source, configuration, build, provider version, preview, and activation outcome. This is what production deployment should look like — every release is a frozen, traceable artifact.

> "For every new JavaScript or TypeScript project, Forge strongly recommends pnpm as the default package manager."

Opinionated infrastructure is good infrastructure. Forge takes a clear stance on package managers, with `cache: auto` that detects the declared manager and restores only that manager's store. This is the opposite of "works with anything" — it's "works best when you follow the paved path, still works otherwise."

> "Browser login authenticates the web app only. It does not configure credentials for local git pull or git push."

The auth model is carefully scoped: browser JWTs are for the web UI, PATs are for Git and API access, and the two never mix. The docs repeatedly warn against putting tokens in remote URLs, inspecting browser storage for credentials, or using session tokens as Git passwords. This is security hygiene as documentation.

> "Forge evaluates forgeBuild.ts as static data without executing repository code."

The build configuration is a data file, not executable code. This is a hard security boundary — Forge reads the config to understand what to build, but never runs arbitrary repository code during configuration evaluation. Compare with GitHub Actions, where workflow YAML can trigger arbitrary code execution from the first push.

## Key Themes

- **#platform** — SmolForge as a full-stack code forge on Cloudflare serverless infrastructure
- **#tool** — The `sf` CLI and `@smolai/forge` npm package as the agent-native interface
- **#pattern** — Immutable previews + SHA-gated activation as the deploy model
- **#concept** — Repository agents as a platform primitive: every repo gets its own durable agent
- **#security** — Scoped PATs, `credential.useHttpPath`, never tokens in URLs, static config evaluation
- **#concept** — AI transcripts as first-class commit artifacts linked via git trailers

## Critical Analysis

**The Cloudflare bet.** SmolForge is the most ambitious proof of concept for Cloudflare's platform primitives. If you can build GitHub on Workers + D1 + R2 + Durable Objects, what can't you build? This is the same thesis that animates [[Cloudflare OS]] — that Cloudflare's infrastructure composes into full applications, not just edge functions. The difference is that Cloudflare OS is a platform for building *organizational* tools; SmolForge is a platform for building *software*. Both prove the same point from different angles.

**Repository agents as code review infrastructure.** The agent system is the most forward-looking feature. Every repo gets a durable, multi-turn agent that can read code at an exact SHA and return validated citations. Phase 1 is read-only, but the architecture is designed for write-capable agents. This isn't "ChatGPT for your repo" — it's infrastructure for automated code review, spec validation, and eventually automated PRs. Compare with [[Agentic Code Review]], where Cloudflare already runs AI review at scale — SmolForge could be the platform that hosts those review agents alongside the code they review.

**The deploy model is underrated.** Forge Deploy's immutable preview + SHA-gated activation + pointer-based rollback is a better deploy model than what most teams build themselves. The fact that Forge uses this same model for its own API and web app is the strongest possible endorsement. [[Cloud Software Factories]] describes centralized SDLC automation; SmolForge Deploy is a concrete implementation of the deploy-and-verify stage, with the added property that every deployment is traceable to an exact source SHA.

**Transcript storage is a wedge.** Storing AI coding agent transcripts alongside commits is a feature GitHub can't easily replicate without changing its data model. It turns agent sessions from ephemeral chat logs into permanent, queryable artifacts linked to the code they produced. The `AI-Session` git trailer convention and the multi-agent support (Claude Code, Codex, Cursor, Copilot, Factory Droid, Devin, OpenCode) make this a cross-harness standard rather than a SmolForge-proprietary format. This is the kind of feature that could pull developers from GitHub — not because SmolForge hosts Git better, but because it treats agent provenance as a first-class concern.

**What's missing or unproven.** The docs are comprehensive but aspirational in places. Merge commits, squash, and rebase are "not yet supported" — fast-forward only. The Workspace agent profile returns `backend_unavailable`. Forge AI is alpha with significant constraints (no streaming, no browser-direct, no provider selection). LFS adoption API is "implemented and tested in source, but is not live." The architecture is sound, but a production-grade GitHub replacement needs these gaps closed.

**The pnpm bet is interesting.** Forge's strong recommendation of pnpm as the default package manager is a bet on deterministic, content-addressed dependency management. It's the right bet for reproducible builds and cache efficiency, but it's also a bet against the network effects of npm's registry dominance. The `cache: auto` abstraction (detect manager, restore appropriate cache) is the right escape hatch — it makes the recommendation a default, not a requirement.

**Comparison to Code Storage.** [[Code Storage]] is API-first Git infrastructure betting that agent-created repos will outnumber human-created ones. SmolForge shares that thesis but takes the full-stack approach: not just Git storage, but the entire forge surface area (issues, PRs, CI/CD, agents, transcripts). Code Storage is the Git layer; SmolForge is the platform. They're complementary — an agent platform could use Code Storage for repo infrastructure and SmolForge for the collaboration layer. But SmolForge's transcript storage and repository agents make it the more interesting platform for agent-native development.

**The self-hosting flex.** SmolForge hosts its own source code (`forge.smol.ai/swyx/forge`). This is eating your own dogfood at the deepest level — the platform's reliability, performance, and feature completeness are continuously tested by the development of the platform itself. If SmolForge goes down, development of SmolForge stops. That's either brave or reckless, but it's definitely a statement.

**Comparison to the broader agent infrastructure landscape.** [[Best Infrastructure Platforms for Coding Agents in 2026]] surveys sandbox platforms; SmolForge operates one layer up — it's where the code those agents produce gets stored, reviewed, and deployed. [[Cloudflare Temporary Accounts for Agents]] solves the deployment auth problem; SmolForge solves the collaboration and provenance problem. Together they sketch a future where the entire development lifecycle — from agent sandbox to deployed application — runs on Cloudflare infrastructure with agents as first-class participants.

---

*Sources: [[raw/forge-smol-ai]], [[summary/forge-smol-ai]]*
*Last updated: 2026-08-07*
