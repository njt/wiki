# Hunk

A terminal diff viewer built for the agentic code review workflow. Hunk renders an entire changeset as one continuous stream with agent annotations rendered inline above the hunks they annotate, breaking from the single-file-flipping model that has defined diff tools for decades. It treats the agent as a first-class participant in review rather than an afterthought.

---

## Key Quotes

> "The whole changeset reads top to bottom in sidebar order — no flipping between single-file views."

The core design decision: a diff viewer where the changeset is the unit, not the file. This is a response to agent-generated PRs where changes span many files and navigating them one-at-a-time is disorienting. The continuous-stream model matches how reviewers actually think about a change — as one coherent delta, not N disconnected file views. [[Agentic Code Review]] documents the volume problem this solves: 861% more churn and 441% longer reviews make per-file navigation untenable.

> "Agents leave notes in a sidecar; hunk renders each one inside the diff, right above the hunk it annotates — summary, rationale, and author."

The sidecar pattern is the architecture story here. Rather than embedding agent output in the diff itself (brittle, polluting), Hunk keeps it in a separate file and renders it as an overlay. This means the agent can produce structured review output independently, the human can read it inline alongside the code, and neither contaminates the other. It's the same separation-of-concerns that makes [[Subspace]]'s structured feedback contract work: agent output and human judgment stay in different lanes.

> "Side-by-side when your terminal is wide, stacked when it's narrow. Auto picks for you; press 1, 2, or 0 to override at any time."

The adaptive layout is a UX signal: this tool was built by people who use terminals daily and know that terminal width changes constantly (tmux splits, resized windows, SSH sessions). Auto-detection with keyboard overrides is the right balance. Compare to [[cmux]], which is also built around the reality of multi-pane terminal workflows with AI agents.

> "Pipe patches straight in and hunk is your pager, too."

Unix composability as a design value. Hunk isn't just a standalone tool — it accepts stdin, making it `git diff | hunk` or `diff -u old new | hunk`. The pager use case means it works with any VCS, any diff source, any pipeline. This is the philosophy behind [[Git Diff Drivers]] carried forward: git's extension points are powerful but under-documented; Hunk provides a polished consumer for any diff producer.

---

## Key Themes

#tool #code-review #terminal #agentic-development #diff #TUI

Hunk sits at the intersection of three trends tracked in this wiki:

**The agentic review workflow.** Addy Osmani's [[Agentic Code Review]] names the structural shift: agents produce code at accelerating throughput while humans remain the fixed-capacity bottleneck. Hunk is built for exactly this world — it doesn't generate review comments (that's the agent's job), it renders them alongside code so the human can triage efficiently. The sidecar pattern means multiple agents could annotate the same changeset, each producing independent feedback that the human reads in context.

**Terminal-first developer tools.** The terminal is the natural habitat for agent-assisted development, and a wave of tools is optimizing for it: [[cmux]] manages multi-agent terminal sessions, [[Subspace]] hands Markdown to humans for terminal-based review, and [[Zellij]] and [[Tmux Resurrect]] handle multiplexing and persistence. Hunk fits here as the *viewer* in a terminal-native review pipeline — the component that makes a diff readable and navigable without leaving the keyboard.

**The diff viewer as a distinct category.** `git diff` output is functional but primitive. [[Git Diff Drivers]] shows how to wire external diff tools into git for structured formats. [[Local Review]] provides GitHub-style commenting on local branches. [[sem]] does entity-level diff via tree-sitter across 31 languages. Hunk occupies a specific niche: the *reading* experience. It doesn't generate, transform, or annotate diffs — it renders them for human consumption, with agent input as an overlay. [[Meat (Reading Diff)]] is the *transform* stage Hunk deliberately isn't: an LLM, constrained to source-anchored elisions, compresses the diff to its load-bearing lines before any renderer sees it. The two compose — meat's abridged diff is exactly the kind of digestible changeset Hunk's continuous stream is built to display.

---

## Critical Analysis

**The sidecar architecture is the right call and the hardest one.** Embedding agent annotations in the diff itself would be easier to implement (just splice lines) but would create a mess: annotations get committed, pollute `git blame`, and break patch application. The sidecar keeps agent output ephemeral and separate. The cost is that the sidecar format must be documented and stable for agent authors to target — and the page doesn't tell us what that format is. If it's JSON with line-range anchors, that's a de facto API. If it's just "whatever the first agent writes," that's a coordination problem waiting to happen when multiple agents try to annotate the same diff.

**The continuous-stream model is a bet on changeset size.** Reading all files top-to-bottom works beautifully for PRs of 200-500 lines across 5-15 files — the sweet spot for code review. For a 5,000-line changeset across 80 files, it becomes a scroll marathon. Hunk mitigates with sidebar navigation and hunk jumping (`[`/`]`), but the fundamental tension is that one-stream-is-good assumes the changeset is cognitively digestible as a unit. Agent-generated PRs routinely violate that assumption. The right answer is probably stacked PRs ([[gh-stack]]) that decompose large changes into reviewable layers, with Hunk as the viewer for each layer.

**What's missing: the sidecar format specification.** The page says "agents leave notes in a sidecar" but doesn't define the format. Is it JSON with `{file, line_range, summary, rationale, author}`? Can multiple agents append to the same sidecar? Is there a severity or confidence field? Without this, agent authors are guessing. The structured review report format Monperrus proposes in [[The End of Code Review]] (JSON/SARIF output, signed identities) points toward what a production-grade sidecar format should look like. Hunk could be the renderer for that format.

**The npm/brew/nix install trifecta is table stakes done right.** A terminal tool in 2026 needs to be installable from the package manager the user already has. Hunk hits all three without asking for `curl | bash` or a bespoke installer. The Node.js 18 requirement on npm is a reasonable floor (18 is the oldest maintained LTS as of 2026).

**Comparison to the landscape:** [[Local Review]] is a different tool for a different moment — it's for humans to leave review comments on local branches, then export for agents to act on. Hunk is for agents to leave review comments on any diff, then humans to read. They're complements in a review pipeline: agent annotates → human reads in Hunk → human leaves structured feedback → agent acts. [[Subspace]] occupies the human-feedback step; Hunk occupies the agent-annotation rendering step. Together with [[Git Diff Drivers]] (the plumbing layer), they form an emerging terminal-native review stack.

**The bet on terminals is a bet on developers who use coding agents.** The audience for this tool isn't "all developers" — it's developers who already work in a terminal, already use coding agents, and want those agents to participate in review. That's a small but growing slice of the market, and it's the slice where tooling innovation is happening fastest. Hunk is betting that slice becomes the norm.

---

*Sources: [[raw/hunk-dev]], [[summary/hunk-dev]]*
*Tags: #tool #code-review #terminal #agentic-development #diff #TUI*
*Last updated: 2026-08-08*
