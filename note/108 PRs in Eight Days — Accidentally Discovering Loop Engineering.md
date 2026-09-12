# 108 PRs in Eight Days — Accidentally Discovering Loop Engineering

Brittany Ellich's field report of pointing an agent at her task board for eight days and getting 108 PRs out the other end — versus a 5–10 PRs/week baseline. The number is the hook, but the durable content is the machinery: a three-layer protocol/loop/worker system, a one-file-per-task markdown board with single-writer discipline, a self-terminating `/loop`, and a hard-capped memory file whose recurring defects graduate into tasks. The honest ending is that she is now the bottleneck, doing spec work at both ends of the pipeline.

---

## Key Quotes

> "unless you're shipping nuclear safety code, you probably don't need a human reviewer for your PRs"

Her Bluesky provocation. What saves this from hot-take territory is what follows: the most consistent pushback wasn't about AI review quality but about review as alignment and junior mentorship, and she concedes the point — on a two-to-three person team that talks daily, alignment happens in conversation, not in a review tab. Crucially she clarifies she didn't remove the human from review; she moved where it happens.

> "The loop reads the board, runs gh, updates task frontmatter, and dispatches work. It writes no code... The worker writes code in its own isolated worktree and reports back with a small structured block. It never touches the board, and it never talks to me directly."

The architecture in two sentences. This is separation of concerns enforced structurally rather than by instruction: the dispatcher owns state, the worker owns code, and neither can corrupt the other's artifact. "It never talks to me directly" also means the human has exactly one channel in — the board.

> "One markdown note per task fixes this structurally... the board becomes queryable as data instead of something that has to be parsed as text."

Her first board was one file with a table in it, which lasted a day before she and the loop started colliding on the same file. One-record-per-file is a data-modelling decision, not an Obsidian preference: it shrinks the write-conflict window from "the whole board" to "the same task," makes the board queryable, and means a stray `|` in her notes can't break the system.

> "Bare /loop without an interval is self-paced... it can end the loop entirely once the board says nothing can move without me... meaning that I finish using tokens as soon as it runs out of work."

The most underappreciated detail in the piece. An interval makes `/loop` a cron job that runs until killed; bare `/loop` is a loop that knows when it's done. The stop condition is written as a conjunction — nothing in progress, nothing waiting on CI, nothing merge-ready — and when it holds, the loop says what it's waiting on and stops. Termination is a designed feature of the protocol, not an accident of exhaustion.

> "Both ends of my pipeline are spec work now: deciding what goes in the queue, and verifying what comes out. It almost feels like being a software engineer in 2026 is just being a product manager and QA."

The bottleneck migration, observed from inside rather than argued from theory. Her board sits at zero queued and eight tasks waiting for her to test: the loop is fast, and the human is the constraint — at queue-entry (so she's building a skill to turn issue-exploration sessions into backlog items) and at verification (so she's looking at automating accessibility, mobile, and regression checks).

---

## Key Themes

#pattern #concept #tool #person

- **#pattern — Protocol / Loop / Worker**: three layers with strict write ownership. The protocol is a markdown rules file that lives next to the board, so "editing the rules and editing the board are the same act"; the loop dispatches but writes no code; the worker codes but writes no state. Single-writer discipline extends one layer down: the loop is the only writer to board and memory, so three concurrent workers can't corrupt anything.
- **#pattern — Self-terminating loop**: no interval, self-paced delays, and an explicit stop condition that ends the run when nothing can move without her. Token burn is bounded by available work rather than by a timer.
- **#pattern — Memory with a hard cap**: a memory file of codebase facts, repeated preferences, and recurring defects, each with a confirmation count and date, pasted into every worker dispatch. The cap forces curation tradeoffs; high-confirmation entries graduate into tasks of their own (the stale-skill deletion), so the process digests its own recurring pain into backlog items.
- **#concept — Review moved, not removed**: human judgment relocates from the PR tab to testing-and-outcome level, and the definition of done is written precisely — task completed, green CI, approved AI code review.
- **#concept — Batched releases at high merge velocity**: with 108 PRs/week she prefers CI plus a preview/stage environment over continuous deployment, averaging ~2 production changes a day, to shrink the regression-testing surface.
- **#tool — Claude Code `/loop`**, `gh`, git worktrees branched from the default branch, Obsidian Bases for the board.
- **#person — Brittany Ellich**; the lineage she cites: **Addy Osmani** (coined "loop engineering"), **Boris Cherny**, **Peter Steinberger**.

---

## Critical Analysis

**The team context is the fine print on the headline number.** 108 PRs in eight days is not a transferable benchmark. It was achieved on a two-to-three person team that is "cooking," with high trust, no mandatory second reviewer, and — unspoken but load-bearing — a single person who both feeds the queue and verifies the output. In an org with review SLAs, compliance gates, or juniors being mentored, this exact system produces a different outcome. Ellich is unusually honest about this; readers quoting the number should be held to her caveats.

**Verification moved to a place with no artifact.** "I still look at every change, but at the testing-and-outcome level" — line-by-line review leaves a written trail (comments, approvals, discussion); manual outcome testing leaves her memory. The system's quality gate is unaudited and doesn't compound, and she knows it: her two next iterations are exactly attacks on this — a skill to cheapen queue-entry, and automation for the accessibility/mobile/regression checks before things reach her. Compare [[Aviator Verify]], which makes outcome-level verification produce evidence, and [[A New Era for Software Testing]] for the checklist-driven QA she's reaching toward.

**The memory design is the most stealable part.** Confirmation counts make the cost of repeated mistakes visible; the hard cap forces relevance tradeoffs; the graduation rule (high-confirmation defect → its own task) converts chronic friction into backlog. It's a crude, effective forced externalization of unwritten knowledge — the same "if you've said it twice, it belongs in a file" instinct as [[Organizing Claude Code for Product Work]], and a working answer to the continuity-not-memory argument in [[Maybe Coding Agents Don't Need a Bigger Memory]].

**The CD reversal deserves more attention than she gives it.** Preferring batched releases over continuous deployment inverts a decade of "deploy small, deploy often" — and it's rational, because the constraint moved. When merge was expensive, small deploys de-risked merge; when merge is free and verification is expensive, batching amortizes verification across many changes. The economics of the release level reasserting itself over the merge level is the same force measured in [[Writing Code vs. Shipping Code]]: throughput gains that attenuate when they hit the human-paced part of the pipeline.

**What's missing: cost and survival rate.** No token spend, no cost per delivered task, no reject/rework rate, and no statement of how many of the 108 survived her testing. Eight tasks waiting for testing at once suggests verification debt accumulates in bursts — the queue drains faster than the human can absorb. This is the same missing measurement that [[Automating Myself Out of Development]] concedes, and without it "108" is an activity metric, not an outcome.

**Her gap diagnosis of the genre is correct.** "Most of what I've read about it stops at 'give it a clear goal and a turn cap'" is a fair hit on the loop-engineering literature: the hard-won details here are stop conditions, single-writer ownership, one-record-per-file boards, and memory curation — none of which fit in a slogan. Her starter advice is correspondingly spec-first: write down exactly what "done" means before anything else, because every good decision in her protocol came from being forced to define a state precisely.

---

## Related Pages

- Strengthens [[Loop Engineering]] with the practitioner counterpart to Osmani's taxonomy — she built the practice before knowing its name, and supplies the operational layer (stop conditions, single-writer discipline, memory caps) that sits between the five-component taxonomy and a system that actually runs; her "stops at a clear goal and a turn cap" jab also complicates the taxonomy's sufficiency from below.
- Extends [[Poor Man's Loop Engineering]] with a sustained run of its two ingredients: the workers self-test via green CI plus an approved AI code review, but her case shows the "someone other than itself reviews" ingredient migrating out of the PR and into human outcome testing — where independent review happens is negotiable; that it happens somewhere is not.
- Confirms [[Automating Myself Out of Development]] with an independent convergence: board-as-markdown, overnight churn, and the same bottleneck shift ("no time to code" → "no time to spec and review"). Ellich's contribution over Isabekyan's is the self-terminating loop and the single-writer rule — the daemon becomes a loop that knows when it's done.
- Grounds [[The End of Code Review]] in lived data: Monperrus argues mandatory human review is indefensible, and here is a practitioner who ran that regime for eight days at 20x her old throughput — but her caveats (tiny high-trust team, review relocated to outcome testing rather than abolished) are exactly the boundary conditions the indefensibility argument needs to carry to real teams.

Also contrasts with: [[Agentic Code Review]] (her approved AI review inside the loop is the Osmani field guide's pattern instantiated), [[Building Autonomous Goal Loops That Deliver]] (her stop condition is a weaker, human-triggered version of a verifiable goal condition).

---

*Sources: [[raw/3mrjj34puva23-108-prs-in-eight-days-accidentally-discovering-loop-engineering]], [[summary/3mrjj34puva23-108-prs-in-eight-days-accidentally-discovering-loop-engineering]]*
*Last updated: 2026-09-13*
