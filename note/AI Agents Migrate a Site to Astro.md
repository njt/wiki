# AI Agents Migrate a Site to Astro

Andrea Bizzotto's case study of migrating Code With Andrea — a 400+ page static site on a custom Swift generator — to Astro with AI agents, preserving 100% feature parity. The thesis: AI makes implementation cheap, but you still have to be the expert. "Feature parity" becomes a testable migration contract, risky assumptions are proven with a throwaway prototype, content conversion runs through a deterministic tool that fails loudly instead of guessing, and every production decision stays human-owned.

---

## Key Quotes

> "All of this is expert work. A model can help inspect the old codebase and implement the new one, but it cannot reliably decide which legacy behavior is a bug, which behavior users depend on, and which unused legacy features you can safely remove."

The article's core claim, and the reason the whole workflow is shaped the way it is. The machine can translate; the human must triage *what's worth translating*. This is the same boundary [[The Archaeologist's Copilot]] draws — the AI is a tireless junior on the translation layer, the human owns strategy and the decision to stop.

> "I had to turn it into a migration contract: a testable description of every route, redirect, metadata field, asset, interaction, and intentional exception."

The sharpest idea in the piece. "100% feature parity" is a slogan until you decompose it into falsifiable requirements — including *intentional exceptions* ("a discontinued feature must be deliberately excluded, not accidentally omitted"). This is [[Specifications as the Product]] applied to migration: the contract is the durable artifact, the regenerated site is disposable.

> "If an agent sees an unfamiliar piece of input and guesses, it can produce output that looks reasonable while dropping behavior. So I designed the migration tool to fail loudly on unknown placeholders."

The deterministic-tool insight. One-off LLM passes are imprecise precisely because they *can* guess and still look right. A deterministic converter with a test harness fails closed. This is Adam Jacob's "use the agent to build the program that minimizes the need for the agent" from [[Reducing Token Spend with Deterministic Workflows]], applied to content transformation.

> "A screenshot can tell you whether a page looks right. It can catch missing images, broken CSS, and responsive problems. But it cannot tell you whether every canonical URL and redirect is preserved, an RSS feed is complete, or a form submits to the right destination."

The verification-hierarchy argument. The article's most concrete bug — `setState(() {})` rendering literal `{` `}` braces because MDX escaping broke inline code — slipped past a full build and "looked fine" on most pages. Screenshots verify *plausible output*; direct contract tests verify *behavior*. This is [[Ways of Checking]]'s "checking again re-runs the instrument; checking differently tests it" in production.

> "AI makes implementation cheaper, but you still need to be the expert."

The TL;DR. The six-step closing sequence (define outcome and constraints → prove riskiest assumptions on a small slice → turn repeated transformations into reviewable tools → verify behavior not appearance → improve after a safe baseline → keep high-stakes decisions human-owned) is a miniature migration playbook worth stealing whole.

## Key Themes

- **#concept — Migration contract**: "Feature parity" decomposed into testable requirements plus intentional exceptions. The reframe that makes a vague goal actionable and lets verification actually verify something. A direct migration-flavored instance of [[Specifications as the Product]].
- **#pattern — Prove the risky assumption first**: A throwaway prototype tests whether Astro content collections can preserve canonical URLs before bulk migration. The cheapest falsification step, and the article's clearest echo of [[Load-Bearing Assumptions]] — catch the wrong assumption before the first line of code.
- **#tool — Deterministic migration tool**: A one-way, reviewable converter with its own test harness that fails loudly on unknown placeholders. AI writes the tool; the conversion itself is repeatable and checked. Converges with [[Reducing Token Spend with Deterministic Workflows]].
- **#pattern — Two-tier verification**: Fast, focused checks on every agent run; the exhaustive production-parity + curated-screenshot suite only at release-candidate gates. Optimization against feedback-loop length, not verification thoroughness.
- **#concept — The final production decision is yours**: Agents gather evidence and prepare release candidates, but the cut-over — and the judgment call on whether a found bug is "preserved contract" or "something to fix" — stays human. Echoes the "AI was not a replacement for engineering judgment" line from [[Patreon Notification Fanout]].
- **#person — Andrea Bizzotto**: Flutter/Dart educator (Code With Andrea) who ran the migration solo, ~2 hours/day over two weeks, ≤2 agents in parallel, every issue and PR reviewed by hand.

## Critical Analysis

**The article's real contribution is the migration contract, not the migration.** Astro, tmux-on-a-VPS, git worktrees, two agents in parallel — none of that is novel ([[Domenic Denicola's Agentic Coding Setup]] runs the same infrastructure for phone-based development). What's worth stealing is the reframe of "feature parity" from a slogan into a falsifiable contract that *includes intentional exceptions*. That last part is easy to miss and quietly important: a literal migration would have carried dead sponsorship, coupon, and A/B-testing code forward. Treating "deliberately excluded" as a first-class requirement is what separates curation from copying.

**This is the anti-[[AI Code Migration with Claude Code]].** Anthropic's field report is a factory: dozens of parallel agents, $165K in API cost, a rulebook that learns, "done" means the file exists on disk. Bizzotto explicitly cites the Bun rewrite as inspiration, then rejects its shape — he ran at most two agents in parallel so he could *review every session's output*. The factory optimizes for throughput; this optimizes for reviewability. Both are defensible; the interesting question is what scale of migration justifies which — and Bizzotto's answer (a solo operator, a site they understand deeply, six years of context in their head) is honestly reported but not generalizable to the Bun case.

**The "expert" claim is doing a lot of work, and the article is honest about that.** "You still need to be an expert" is true but underspecified. What exactly does the expert know that the agent doesn't? The article's examples are concrete: which legacy feature is a bug vs. a behavior users depend on, which obsolete signup flow to drop, whether a wrong link label should be "preserved" or "fixed." That's product knowledge and taste, not code knowledge — the same "strategy the machine cannot provide" [[Dev Machine Foundry]] identifies. The expert's real job here is *decision-point ownership*, which is precisely [[Optimizing for Decision Points]].

**The token-budget framing deserves more weight than it gets.** Bizzotto mentions "blowing your token budget" and "agent-sized work items… improving accuracy and reducing token usage" almost in passing. But the two-tier verification split (fast checks on every run, full suite only at release gates) is really a *cost-management* decision as much as a speed one. The full screenshot suite — hundreds of routes × viewports × themes — was "painfully slow," and slow agent loops are expensive agent loops. This is the same economics [[Managing AI Coding Costs at Scale]] names, applied by a solo operator instead of a platform team.

**What's missing: failure modes and the rollback story.** We get the brace-escaping bug and the course-link-label bug, but no taxonomy of what the agents got *wrong* that had to be redone, and the "kept the old site available as a rollback target" line is stated, not exercised. For a piece whose whole argument is "trust but verify," a section on what verification caught and what it missed would strengthen it — and the absence of it makes this read more like a victory lap than a postmortem. Still, as a *replicable sequence* rather than a brag, the six-step closing sequence is genuinely useful.

## Cross-Links

- [[AI Code Migration with Claude Code]] — the factory-shaped counterpoint this solo migration explicitly rejects; the two define the scale/oversight trade-off in agentic migration
- [[Reducing Token Spend with Deterministic Workflows]] — the deterministic-tool insight ("use the agent to build the program that minimizes the need for the agent") applied to content conversion
- [[The Archaeologist's Copilot]] — the same expert-judgment boundary: the AI translates, the human decides what's a bug vs. a feature
- [[Load-Bearing Assumptions]] — "prove the riskiest assumptions with a small slice of work" as a concrete validation method
- [[Patreon Notification Fanout]] — another AI-assisted migration case study, same "not a replacement for engineering judgment" conclusion at org scale
- [[Specifications as the Product]] — the migration contract as durable artifact, the regenerated site as disposable
- [[Ways of Checking]] — the screenshot-vs-contract distinction as "checking differently tests it"
- [[Optimizing for Decision Points]] — the expert's role collapses to owning the taste-sensitive decisions

---
*Sources: [[raw/ai-agents-migrate-site-astro]], [[summary/ai-agents-migrate-site-astro]]*
*Last updated: 2026-08-14*
