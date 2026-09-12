# Introduction to Obsidian

Bryan Hogan's practitioner introduction to Obsidian, the markdown-based note-taking app. Less a tutorial, more an opinionated field report: why local-first markdown wins, how to resist the plugin rabbit hole, and an honest admission that graph view is mostly eye candy.

---

## Key Quotes

> "Keep it simple. Obsidian should help you work on other things."

## Key Themes

#tool #obsidian #knowledge-management #note-taking #local-first #markdown

**File over app.** Hogan's core argument is that Obsidian's value proposition starts with file ownership. Local markdown files are portable, immune to vendor enshittification, and readable by any tool from vim to AI agents. He cites Obsidian CEO Steph Ango's "file over app" philosophy. This is the same design instinct driving [[tolaria]] (files-first, git-first), [[life-system]] (plain-text markdown as life OS), and [[robot.wtf]] (shared wiki for humans and agents). The file-over-app thesis is quietly becoming the foundational assumption of the agent era: if your notes aren't plain text, your agent can't work with them.

**The plugin trap.** Hogan advocates minimalism -- core plugins over community plugins, no custom themes, restraint over exploration. His rule: "set limits on customization exploration." This is the Obsidian equivalent of [[Slowing the Fuck Down]] -- deliberate friction against the urge to optimize the tool instead of using it. The Obsidian community's plugin ecosystem is both its greatest strength and its biggest time sink.

**Zettelkasten and bottom-up organization.** Hogan uses Zettelkasten and Evergreen notes with a bottom-up methodology, letting structure emerge from linked atomic notes rather than imposing top-down folder hierarchies. This is the same pattern behind [[LLM Wiki]] -- let connections accumulate and structure crystallize organically. The `[[wikilink]]` syntax is the primitive that makes both human Zettelkasten and LLM-maintained wikis work.

**Graph view is overrated.** Hogan is refreshingly honest: graph view looks impressive on social media but has limited daily utility. Canvas needs more development. The real value is in the linking, not the visualization. This maps to a broader pattern in the wiki -- [[graphify]] and [[lat.md]] offer richer structural analysis than Obsidian's built-in graph.

**Syncing remains unsolved.** Google Drive + DriveSync + GitHub backups is a functional but fragile stack. Obsidian Sync exists as a paid solution. [[Headscale]] offers self-hosted networking infrastructure, but file sync for knowledge bases across platforms remains an awkward problem without a clean universal answer.

## Critical Analysis

This is a competent beginner's guide, not a deep technical piece. Its value is in what it *doesn't* say: there's no breathless hype about "second brain" or "building a knowledge graph that will change your life." Hogan uses Obsidian for four mundane things (writing, collecting information, tracking projects, logging media) and tells you to keep it simple. That pragmatism is rare in the Obsidian community, which tends toward elaborate vault architectures that become maintenance burdens.

The interesting tension: Hogan advocates minimal plugins and simple organization, but this wiki's own existence -- an LLM-maintained knowledge base inside an Obsidian vault -- represents a far more ambitious use of the same tool. Obsidian is simultaneously a simple markdown editor for humans and a structured filesystem for AI agents. Those two use cases pull in different directions. The human user wants fewer features and less complexity; the agent user wants structured frontmatter, consistent linking conventions, and machine-readable schema. [[tolaria]] is betting that the agent use case will eventually need its own tool. For now, Obsidian serves both, but the seams are visible.

The "file over app" philosophy is the most durable insight here. Apps come and go; markdown files survive. Every tool in this wiki that stores knowledge in plain text -- [[life-system]], [[robot.wtf]], [[napkin]], [[Planning With Files]] -- is making the same bet Obsidian made, and it keeps paying off.

A harder-line cousin of Hogan's minimalism is [[Keep AI Out of Your (Obsidian) Vault]]: it extends "keep it simple" into "keep the AI out," arguing that generated notes become "AI slop" that drowns out your own writing, and that search (Omnisearch, Smart Connections) is the one place AI genuinely earns its keep in a personal vault. Same file-over-app premise, opposite conclusion about how much of the vault should be machine-written.

---
*Sources: [[summary/obsidian-introduction]]*
*Last updated: 2026-05-14*
