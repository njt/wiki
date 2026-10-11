# Paseo

Paseo is an Apache-2.0, single-maintainer client that sits in front of many coding agents — desktop, web and mobile — unifying providers, exposing MCP/CLI/SDK automation, and fanning issues out to per-worktree agents, all on an explicitly anti-VC, code-stays-local philosophy.

---

## What it is

The landing page is short but each line is a design position. Paseo is not an agent; it is a *harness for harnesses* — a unified client where every coding agent is a swappable provider, used "the same way." Your subscriptions, skills and configuration carry over. It runs anywhere (laptop, server, team) and connects from any device, with FAQ entries confirming code doesn't leave your machine and no forced login.

The automation story is the interesting part: the UI's capabilities are exposed through MCP, a CLI, and a TypeScript SDK, so the demo shows an agent itself orchestrating other agents — finding two ready GitHub issues and running each in its own worktree, then reporting completion. Plugins extend three layers: UI (panels, slash commands, themes), agent lifecycle (environment, MCP servers, answering permissions, turn-end follow-ups), and providers (adding a new agent as a first-class citizen).

## Key quotes

> "Every coding agent, used the same way. Switch providers any time."

The unification claim. This is the thin-client counter-thesis to the lock-in model: agents are commoditized, the client is the durable surface. It is also a bet that provider churn is a permanent condition users will pay (in setup friction) to escape.

> "A tool that sits between you and your code, your keys and your machines has to stay neutral, and funding creates pressure to monetize."

Mo's funding argument, stated with unusual directness. Paseo's position in the trust path — it holds credentials and drives your machines — is precisely why the no-investors stance is a feature, not just branding. Whether sponsorship plus "Paseo Hub" can sustain one full-time maintainer against VC-funded competition is the obvious stress point.

> "If Paseo doesn't fit how you work, change it with a plugin or fork."

Extensibility as an escape hatch from opinionated tooling, backed by Apache 2.0 and the "Fork the repo" link.

## Themes

#tool #pattern #concept

## Analysis

Paseo is best read as a bet on where value lands in the agent stack. If models and even agent harnesses keep commoditizing, the durable layer is the *interface and trust boundary* — the thing that holds your permissions, credentials, and workflow regardless of which agent is underneath. Its three-layer plugin system (UI, lifecycle, provider) is the right decomposition: it lets the ecosystem absorb novelty without the core chasing it. The worktree-per-issue fan-out demo also quietly acknowledges that multi-agent orchestration is now an end-user feature, not research.

The risks are the risks of any thin client: providers keep differentiating on things Paseo can't abstract (prompt-caching behavior, permission models, harness quirks), and "used the same way" is only true down to the least common denominator. The single-maintainer independence story is genuinely principled but fragile — the page is, in the end, a sponsorship pitch.

## Related pages

- [[vibe-kanban]] — the other end of the same abstraction: a kanban board driving multiple coding agents through worktree lanes; Paseo unifies the *client*, vibe-kanban unifies the *queue*, and both bet that agents are interchangeable behind a stable control surface.
- [[Munder Difflin — Clones of You, Not a Shared Bot]] — complicates Paseo's unification thesis: if clones-of-you per agent matter, a "used the same way" facade may flatten exactly the per-agent differences that make each agent useful.
- [[Harness Engineering — Anatomy of Eleven Coding Agents]] — the survey that shows how differently real harnesses are built; Paseo's uniform-provider abstraction has to paper over precisely that variety.
- [[Pi Coding Agent]] — a similar minimalist-harness-to-platform trajectory (extensions, SDK, RPC mode); Paseo attempts the platform layer without owning a harness at all.

---
*Sources: [[raw/paseo-sh]], [[summary/paseo-sh]]*
*Last updated: 2026-10-11*
