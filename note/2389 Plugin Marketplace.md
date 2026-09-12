# 2389 Plugin Marketplace

The most comprehensive third-party Claude Code plugin ecosystem: 26 plugins and 4 MCP servers from 2389 Research, spanning development workflows, agent orchestration, quality assurance, infrastructure, and personal productivity. Distributed as a marketplace installable with a single command (`/plugin marketplace add 2389-research/claude-plugins`), all open source under MIT.

---

## Key Quotes

> "Plugins that actually get stuff done."

The pitch is pragmatic utility over theoretical elegance. 2389 positions these as tools their own team uses daily — dogfooding as credibility.

> "Open source Claude Code plugins and MCP servers from 2389 Research"

The "from 2389 Research" is doing real work here. These aren't community contributions aggregated into a directory; they're produced by a single organization with a coherent philosophy about how agents should work. That's a strength (consistent design language) and a limitation (no ecosystem diversity).

> "Digital drugs that modify AI behavior through prompt injection."

The `agent-drugs` plugin description is the most provocative line on the page. It reframes prompt injection — normally a security vulnerability — as a feature. This is either a joke that landed poorly or a genuinely interesting inversion: if you control the injection surface, injection becomes configuration.

---

## Key Themes

#tool #marketplace #plugin-ecosystem #open-source

### The Marketplace as Platform Play

2389 is testing the thesis that Claude Code is a platform, not just a CLI tool. The Chrome Web Store made Chrome a platform; the VS Code marketplace made VS Code a platform. 2389's marketplace attempts the same for Claude Code. The `/plugin marketplace add` command is the install vector; the plugin format is the packaging standard; the marketplace.json registry is the discovery mechanism.

But 26 plugins from one org isn't an ecosystem — it's a portfolio. The question hanging over this: where are the non-2389 plugins?

### Plugin Patterns That Keep Recurring

Several plugins are variations on the same architectural insight dressed for different contexts:

- **Parallel exploration**: test-kitchen (implement variants, tests pick winner), jam (diverse agent perspectives build variants), speed-run (parallel codegen via Cerebras for token efficiency). Three different implementations of "try multiple approaches simultaneously."
- **Review**: fresh-eyes-review (different model reviews your code), review-squad (panel of specialized subagents), documentation-audit (verify docs against codebase). Three flavors of "don't trust the first answer."
- **Refinement loops**: simmer (investigation-first judges propose improvements), prbuddy (CI monitoring, review triage, automatic fixes). Two takes on "iterate until it's right."

This isn't redundancy — it's a design space being explored. But it also suggests these plugins emerged bottom-up from specific needs rather than from a top-down architecture that would have consolidated them.

### MCP Servers as the Interesting Frontier

The four MCP servers (slack-mcp, socialmedia, journal, agent-drugs) are more novel than most of the development plugins. They extend what Claude Code can *connect to*, not just how it *operates*. This is where the marketplace adds genuine new capability rather than packaging existing workflows:

- **socialmedia**: Agents talking to each other through a shared communication layer. This is [[AI Agents with Human-Like Collaborative Tools]] made practical.
- **journal**: Private persistence for agent state. Same category as [[Claude-Mem]] and [[mira-OSS]] but minimal — just a writing surface.
- **slack-mcp**: Brings Claude Code into team chat. The reverse of what most Slack bots do.

### The CEO Plugin Is the Most Telling

`ceo-personal-os` ("reflection frameworks, goal systems, coaching-style reviews drawing on Gustin, Ferriss, Robbins, Lieberman, Campbell, Eisenmann, Collins, Martell, Gerber, and Blank") reveals who 2389 thinks the Claude Code power user is: not just the developer, but the executive who wants an AI chief of staff. This maps to [[Two Kinds of User Are Emerging]] and the observation that many power users are non-technical professionals. It also parallels [[life-system]] (personal life OS on plain-text markdown) and [[Chief of Staff]] (AI chief of staff for executives).

---

## Critical Analysis

This marketplace is a Rorschach test for what you think Claude Code should become.

**If you see a platform:** 2389 is building the app store before Apple does. Plugin marketplaces create network effects — more plugins attract more users, more users attract more plugin developers. Being first mover matters enormously if Claude Code plugins become the standard way to extend agent capability.

**If you see a portfolio:** This is one company's internal toolkit, polished and published. The plugins reflect 2389's specific workflow preferences (heavy on parallel exploration, light on enterprise integration) and their specific tech stack (Firebase, Tailwind, iOS/SwiftPM). A genuinely diverse ecosystem would include plugins for different stacks, different workflow philosophies, different risk tolerances.

**If you see a bet on packaging:** The plugin format itself is the interesting question. Is a Claude Code plugin meaningfully different from a well-written CLAUDE.md file? The marketplace adds discovery and versioning, but the runtime behavior — skills that auto-trigger when relevant — is available to any skill file. The plugin format's value might be social (discoverability, trust signals) rather than technical.

The comparison to [[MinMax Skills]] is instructive. MinMax is a library of development skills organized by domain (frontend, mobile, Flutter, media). 2389 is a library organized by workflow phase (explore, implement, review, refine). Neither is a community marketplace — both are single-org collections. The community marketplace for Claude Code plugins doesn't exist yet, and it's not clear the plugin SDK is mature enough to attract one.

**The worry:** If the only "marketplace" is 2389's, and 2389's plugins become the de facto standard for extending Claude Code, then Claude Code's plugin ecosystem has a single point of cultural failure. This isn't a security risk — the plugins are open source — but a diversity risk. One organization's workflow philosophy shouldn't define the extension surface for a tool used by hundreds of thousands of developers.

**The counter-worry:** Someone has to go first. 2389 shipped 26 plugins, documented the marketplace format, and open-sourced everything. That's the hard part. The easy part — other developers contributing their own plugins — might follow naturally.

---

*Sources: [[summary/2389-plugin-marketplace]], https://github.com/2389-research/claude-plugins*
*Last updated: 2026-05-14*
