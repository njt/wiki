# sx — AI Asset Package Manager

sx is a CLI from sleuth-io that functions as "your team's private npm for AI assets" — a package manager for distributing skills, MCP configs, commands, rules, agents, hooks, and plugins across a team's AI coding assistants. It follows the manifest-and-lock pattern (like npm/cargo/uv), supports scoped installation (org/repo/path/team/user/bot), and works across 10+ AI clients including Claude Code, Codex, Cursor, Gemini, and GitHub Copilot.

---

## Key Quotes

> "Capture what your best AI users have learned and spread it to everyone automatically."

sx's pitch in one sentence. The problem is real: expert users develop custom AI assistant configurations that stay siloed on individual machines. sx treats these configurations as distributable assets rather than personal preferences.

> "Your team's private npm for AI assets — skills, MCP configs, commands, and more."

The npm analogy is doing real work here. Just as npm turned JavaScript libraries into shareable, versioned packages, sx aims to do the same for AI assistant configurations. The manifest-and-lock architecture (sx.toml + per-user lockfiles with timestamped rotations) is lifted directly from the package manager playbook.

> "The relay forwards requests over a WebSocket your machine opens — vault content stays local."

The cloud relay design for web-based clients (claude.ai, chatgpt.com) is clever: expose vaults as MCP endpoints without uploading content anywhere. Your machine opens the WebSocket; the relay just forwards. This sidesteps the enterprise data-residency objection.

## Key Themes

#tool #package-manager #skills #plugins #claude-code #team-collaboration #asset-distribution

**Manifest-and-lock as the right abstraction.** sx's biggest architectural decision was adopting the npm/cargo/uv pattern wholesale rather than inventing something new. The manifest (sx.toml) is the team's source of truth; the lock file is the per-user resolved state; audit and usage streams are append-only JSONL. This is boring technology, which is exactly what you want for a tool that manages what your AI assistants can do.

**Scoping is the hard problem.** The six install targets (org, repo, path, team, user, bot) reveal that distribution isn't just about "push assets to everyone." Some skills are org-wide (code style); some are repo-specific (project conventions); some are per-user (personal preferences). The `--bot` scope in particular — requiring `SX_BOT=<name>` — acknowledges that CI runners and autonomous agents have different identity and needs than human developers. This is more thoughtful than most team-configuration tools.

**The cloud relay is a hedge.** Supporting web-based AI clients (claude.ai, chatgpt.com) via a WebSocket relay that keeps vault content local is the right bet. Web-based AI tools are growing faster than CLI-based ones for many users. But the relay architecture — your machine as the secure endpoint, the relay as a dumb forwarder — is fragile. It requires your machine to be online and connected. For teams, this is a single point of failure unless multiple team members run relays.

**Cross-client support is table stakes.** Supporting 10+ AI clients isn't a feature — it's an admission that the AI coding assistant market is fragmented and will stay that way. Teams use different tools. A package manager that only works with Claude Code is a Claude Code plugin, not a team-wide solution. sx's value proposition depends on this breadth.

**skills.sh integration is a distribution play.** 85k+ community skills via `sx add anthropics/skills/frontend-design` and `sx add --browse` turns sx from a private package manager into a discovery surface. This is the npm registry moment: the tool becomes more valuable as it bridges private team assets and public community resources.

## Critical Analysis

sx is solving a real and growing problem. As teams adopt [[How Intercom Uses Claude Code|100+ skills across their org]], the question of how to distribute and manage those skills becomes urgent. Copying files between repos doesn't scale. sx's manifest-and-lock pattern is the right architecture — proven by decades of package management — applied to a new domain.

But "proven architecture" cuts both ways. The package manager pattern brings package manager problems: dependency conflicts, version hell, stale assets, supply chain risk. sx addresses versioning (lockfiles with timestamped rotations) but the security model is thin. Installing a skill from a team member is installing arbitrary prompts and hooks that execute in your AI assistant's context. The RBAC and change request flow on the roadmap is doing a lot of unspoken work — without it, sx is a distribution mechanism without a trust mechanism.

The comparison to [[2389 Plugin Marketplace]] is instructive. 2389's approach is a centralized registry of plugins from a single org. sx's approach is decentralized: each team runs their own vault (local path, git repo, or skills.new). The decentralized model is better for enterprise security (vault content stays in your control) but worse for discovery (no central registry of what's available). The skills.sh integration bridges this gap for public skills, but private team assets remain isolated.

The cloud relay is simultaneously the most innovative and most fragile piece. Exposing vaults as MCP endpoints for web-based clients is genuinely clever. But "your machine opens a WebSocket" means the relay is only as reliable as the machine it runs on. For a team of 50 people relying on consistent skill behavior, "Bob's laptop is asleep" shouldn't break everyone's Claude.ai experience. The relay needs a server-side deployment option.

The audit trail (`.sx/audit/YYYY-MM.jsonl`) is the most underrated feature. In a world where AI assistants are acting on behalf of teams, knowing who installed what skill when is a compliance requirement, not a nice-to-have. This is the kind of infrastructure that separates "tool for individual developers" from "tool for organizations."

sx's biggest risk is timing. It's a package manager for a package format that doesn't exist yet. Claude Code has plugins and skills, but the format is Anthropic's, not an open standard. If Anthropic changes the format, sx adapts or breaks. If the AI coding assistant market consolidates around a single client, sx's cross-client value proposition collapses. If it fragments further, sx's maintenance burden multiplies. The bet is that AI assistant configuration becomes important enough to warrant dedicated infrastructure — and that the infrastructure layer sits above any single client.

---

## Cross-Links

- [[How Intercom Uses Claude Code]] — The reference for what sx manages at scale: 13 plugins, 100+ skills, hooks enforcement. sx is the infrastructure layer Intercom built themselves
- [[2389 Plugin Marketplace]] — Current plugin distribution model (centralized registry). sx is the decentralized alternative (per-team vaults)
- [[MinMax Skills]] — Skills library at 11.8k stars. sx would manage distribution of exactly this kind of collection
- [[Claude Code is a Beast — Tips from 6 Months of Hardcore Use]] — Skills auto-activation via hooks; sx solves the distribution side of the same problem
- [[Anatomy of the .claude/ Folder]] — Understanding what sx is managing and where assets land
- [[Intent Layer]] — Hierarchical context at folder boundaries; sx's scoping (org/repo/path) mirrors this structure
- [[Writing a Good CLAUDE.md]] — What gets distributed via sx; the asset that needs versioning and team governance
- [[claude-ctrl]] — Enforcement hooks distributed via sx; "an instruction in context is not a constraint"
- [[Agent-Native Architectures (Every)]] — Files as universal interface; sx bets on file-based assets as the distribution primitive
- [[Supply Chain Security for Software Developers]] — The supply chain problem sx inherits by being a package manager for executable AI configuration
- [[Feedback Loop is All You Need]] — The rules and hooks sx distributes are the enforcement layer; skills are the feedforward layer
- [[CLAUDE.md (Universal)]] — The kind of asset that makes sense to distribute org-wide via sx

---
*Sources: [[raw/sx]], https://github.com/sleuth-io/sx*
*Last updated: 2026-05-16*
