---
url: https://news.ycombinator.com/item?id=47626598
title: "Show HN: ctx – an Agentic Development Environment (ADE)"
author: luca-ctx
date_fetched: 2026-05-15
date_published: 2026-04-04
topics:
  - agent-coding-workflow
---

# ctx – an Agentic Development Environment (ADE)

Show HN post by luca-ctx (53 points, 52 comments). ctx is a local-first, no-account development environment built around coding agents rather than human keystrokes. It's not a VS Code fork — it's a workbench that orchestrates multiple agent harnesses (Claude Code, Codex, OpenCode, Copilot) in isolated worktrees with a local merge queue.

Key differentiators from competitors like Conductor:
- Container/VM isolation with explicit network policy (not relying on harness-level safety)
- Local merge queue for reconciling agent-submitted changes
- Linux + remote runtime support
- Broader harness support
- Unified multi-agent transcript format

## OP Comments

**On multi-repo workflows:** workspace attachments for repo-primary, initialize at parent for cross-repo. Caveat: merge queue features are repo-specific.

**On open source:** Described GitHub repo as "issues reporting" then acknowledged the description was misleading. Product is not open source, despite the `.rs` TLD suggesting a Rust open-source project. Will remain "free, no account, fully local, bring-your-own agents/endpoints/tokens" with paid Team/Enterprise for policy enforcement and collaboration.

**On IDE vs ADE:** ctx is "a workbench around agents, not a replacement for IntelliJ/VS Code" — includes diff review and terminal but not a full IDE.

**On the "Lethal Trifecta"** (Simon Willison's article on unrestricted agent access): macOS Seatbelt sandboxing is inadequate; container-based approach with network policies is cleaner.

**On merge conflicts:** Six-step workflow — agent worktree → validate in isolation → submit to local merge queue → replay on latest target → reject if conflicts or failures → agent pulls new upstream state and resolves. Having agents read "the other agent's plan document when it hits a merge conflict, not just the diff" works well.

**On local models:** Any harness that works with local models (Codex, OpenCode) works in ctx.

**On codebase indexing:** ctx sits *around* the agent harness — if a harness has its own indexing/code-search story, you still get that. ctx only adds orchestration (merge queue, agnostic subagent support).

## Notable Comments

**Snakes3727** raised multi-repo coordination challenges.

**sspiff** criticized the misleading GitHub repo with "nothing but some links."

**bloppe** questioned why agent tools fork entire IDEs rather than augmenting existing ones.

**xrd** described their VM-based approach and asked about local models with multi-GPU setups.

**Bnjoroge** asked about merge conflict resolution and noted you need both IDE features (code nav) and agent orchestration.

**unsubtlecoder** asked for comparison with Conductor.

**nhumrich** reported the Linux build launches with a blank window.

**kamalkalwa** flagged the enduring challenge: "how do you give the agent enough autonomy to be useful without losing the ability to course-correct."

**ookblah** confirmed ctx supports fully local git flow with both git and jj for worktrees.
