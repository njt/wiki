# Team-Wide Agentic Harness

Ian Langworth's case for treating the agent harness as a team-scale, version-controlled artifact: a checked-in repo of skills, conventions, and evergreen context that everyone shares, reviews, and improves — rather than a personal `.claude/` directory that lives on one machine. The article is a practitioner's sketch of the moment an individual harness crosses the threshold into team infrastructure.

---

## Key Quotes

> "Skills are code. A skill is a set of instructions given to an agent. Any code change should be reviewed."

The thesis in three sentences. Langworth argues that skills occupy the same category as build scripts, CI config, and infrastructure-as-code: they determine behavior at scale, and they deserve the same review discipline. This is a sharper version of the point made in [[Steering Claude Code]] — instruction-delivery mechanisms have different durability and blast-radius characteristics, and the ones that affect every agent invocation warrant the most scrutiny.

> "Conventions that live comfortably in one person's head survive contact with everyone else's."

The honest question the whole piece hangs on. Langworth has been running his harness solo and now wants to socialize it. The unspoken risk: conventions that feel natural to their originator — where to put `tmp/`, when to use a worktree, what goes in `notes/` vs `plans/` — may be invisible to everyone else. This is the same adoption problem that [[Steering Claude Code]] surfaces with CLAUDE.md conventions: the instruction that works brilliantly for one person's mental model may be baffling to the next.

> "An `evergreen` docs directory, a place with descriptions of the company, the product, and procedures we do often. These are the atomic blocks of context you feed into your context window before you dispatch an agent."

Langworth's term for what [[Claude Code Mastery]] calls "compounding infrastructure" and what [[The Agentic Product Standard v2.0]] formalizes as an 8-layer harness. The insight is that context isn't just a prompt — it's a curated, maintained, version-controlled corpus. Langworth's "evergreen" naming captures something important: this content decays, and keeping it fresh is part of the work.

> "The most important sandbox rule: scope all work into a single directory, giving the sandbox access only to that directory."

A convention that's easy to state and hard to enforce without tooling. [[cco]] and [[How We Contain Claude]] address this at the enforcement layer; Langworth is describing the convention layer that sits above it. The two layers have to match — a sandbox tool that doesn't respect directory conventions is as useless as a directory convention without a sandbox tool.

## Key Themes

#harness #team-practice #skills #code-review #sandboxing #evergreen-context #convention #agentic-development

## Critical Analysis

The piece is valuable precisely because it's incomplete. Langworth isn't presenting a finished system — he's narrating the moment of transition from solo harness to team harness, and he's honest about what he doesn't know. The "let's see what happens" ending is the right posture for mid-2026: nobody has team-scale agent practices figured out yet.

Three things the article gets right:

**Skills-as-code is the right frame.** Once you treat a skill as code, the entire software engineering apparatus — review, testing, versioning, rollback — snaps into place. This is a more useful framing than "prompt engineering" because it carries the implicit expectation of review discipline. [[Vibe Coding as a Team Sport]] makes a parallel argument with different machinery (bram's To-Apply/To-Commit gates). [[CEOS (Claude + EOS)]] is that argument applied to business operations rather than code — a skills package whose data-ownership table (one writer skill per directory) doubles as a concurrency model for a leadership team sharing one git repo.

**The convention layer matters more than the tool layer.** Langworth spends most of the piece on conventions (where files go, how sandboxes work) rather than on specific tools. This aligns with [[Components of a Coding Agent]]'s finding that the harness matters more than the model, and with [[Tuning Claude Code Into a Better Engineering Partner]]'s thesis that workflow beats prompts. The hard part of team-scale agent use isn't picking the right MCP server — it's agreeing on where the `plans/` directory lives and what goes in it.

**Evergreen context as maintained infrastructure.** The "evergreen" concept — company descriptions, product docs, standard procedures — is one of those ideas that sounds obvious and isn't. Most teams' context for agents lives in Slack threads and meeting notes. Langworth is arguing for a curated, maintained, version-controlled alternative. [[Context Engineering at the Frontier (Linus Lee)]] makes the same argument from the search-engineering side; Langworth makes it from the team-practice side.

Two gaps worth noting:

**The nested-skills monorepo pattern adds another dimension to team-scale adoption.** [[Claude Code Skills System]] documents how `.claude/skills/` in subdirectories (`apps/web/.claude/skills/deploy/`) appears as directory-qualified names (`/apps/web:deploy`) and auto-activates when Claude works on files in that directory. This means a monorepo team doesn't need one global harness — each package owns its skills, and Claude picks the right variant by file locality. The `allowed-tools` frontmatter field adds a trust dimension: a project skill that grants `Bash(git:*)` is a permission decision, not just a convenience, and reviewing it requires understanding the tool surface it unlocks.

**The review bottleneck.** Langworth says skills should be reviewed, but doesn't address who reviews them, against what criteria, or at what velocity. [[Agentic Code Review]] documents teams where review is already the bottleneck — adding skill review to the queue without addressing throughput is a recipe for stagnation. The answer might be agent-assisted skill review ([[Orchestrating AI Code Review at Scale]]), but Langworth doesn't go there.

**The single-player-to-multiplayer jump.** Langworth's conventions (tmp/, worktrees/, plans/, notes/) work for one person. Whether they survive a team of five or fifty is an open empirical question. [[Running an AI-Native Engineering Org]] documents the organizational patterns that emerge at scale; [[The Agentic Product Standard v2.0]] provides the structural taxonomy. Langworth's piece sits usefully between them — the practitioner's sketch that motivates the formalism.

The article's implicit bet: that a shared, reviewed, version-controlled harness produces better agent outcomes than a collection of individually-optimized personal setups. That bet hasn't been empirically settled, but it's the same bet behind [[Lean Software Production]] (the product is the system that produces code) and [[I Stopped Coding and Started Architecting Agents (And Why You Should Too)]] (build the harness, not the code). Langworth's contribution is naming the social mechanism — brown bags, shared conventions, checked-in skills — that makes the bet testable. [[Munder Difflin — Clones of You, Not a Shared Bot]] productizes the same idea as a provisioned, versioned "shared knowledge base" every new clone inherits on day one, but draws a boundary Langworth leaves implicit: org-level context compounds into the team hive mind while personal context — repos, notes, style — never leaves the individual's own node ("shared ≠ personal, ever").

---
*Sources: [[raw/team-wide-agentic-harness]]*
*Last updated: 2026-07-18*
