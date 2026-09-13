---
url: https://www.firecrawl.dev/blog/ai-software-factory
date_fetched: 2026-09-13
---

# How to Build an AI Software Factory: Agents That Open, Review, and Merge PRs

## TL;DR

An AI software factory is five stages with a gate at each one. The agent is the cheap part.

| Stage | What it decides | Published example | 
|---|---|---|
| Intake | Which work is worth starting | Sentry's Seer scores every incoming issue for actionability first | 
| Isolation | Where the agent runs without colliding | Stripe boots pre-warmed devboxes in about 10 seconds | 
| Tools | What the agent can reach | Stripe's Toolshed exposes roughly 500 internal tools over MCP | 
| Verification | Whether the change is right | Spotify's LLM judge vetoes about 25% of agent sessions | 
| Merge gate | Who is accountable | Faire requires two human reviews on agent-authored PRs | 

The short version: every company that made this work built the gates before the fleet. Spotify's Fleetshift shipped in 2023, two years before it had an agent to put in it. Generation scales with spend, review does not, and that asymmetry is the whole design problem.

**Where Firecrawl fits.** Agents need live web context the repo does not carry. Search + scrape for the open web and a curated developer index for code, both behind one MCP block, with prompt injection detection on every fetch.

On January 6, 2026, Stephen Toub opened nine pull requests from his phone at 35,000 feet. Seven of them merged. He works on `dotnet/runtime`, and he wrote up what the experience told him:

AI changes the economics of code production. One person with good judgment and a phone can generate PRs faster than a team can review them.


That sentence is the entire subject. A single engineer with a coding agent can now saturate a team's review capacity from an airplane seat. The interesting question stopped being how to make agents write code and became how to absorb the output.

An AI software factory is the answer companies have converged on, and it is what turns autonomous coding agents from a demo into throughput a team can absorb. This guide breaks it into five stages, each with the published architecture behind it and the configuration to build it. For the broader picture of how AI agents reason, call tools, and pull in web context, start with that primer, then come back here for how a coding-agent fleet actually gets shipped.

Assume you have already picked an agent, which we covered in our roundup of the best AI coding agents. The software factory is everything around it.

## What is an AI software factory?

An AI software factory is the system around a coding agent rather than the agent itself. Work arrives from a queue, agents run in isolated workspaces, verification happens automatically, and a human sits at an explicit merge gate. Also called an agentic software factory.

The useful distinction is between an agent and a software factory. Running a coding agent on your laptop is an agent: you choose the task, you watch it work, you read the diff, you merge. Everything except the typing is still you, and your attention is the limit.

A software factory moves those steps into infrastructure. Nobody decides which issue an agent picks up, because intake rules do. Nobody sets up a workspace, because isolation is provisioned. Nobody checks whether the change compiles, because verification runs before a human is involved at all. The person shows up at the end, on the decisions that carry accountability.

Addy Osmani puts it more compactly:

A software factory is harnessing loops at scale.


The loops he means are the ones we covered in our guide to loop engineering: an agent that runs, checks its own work, and runs again until a verifier says stop. The software factory is the machinery that runs many of them at once without anyone watching.

Three properties separate a real software factory from a pile of scripts:

- **It is queue-driven, not prompt-driven.**Work enters from issues, alerts, or a Slack channel, and the system decides what is worth starting. Nobody is typing prompts.
- **Environments are disposable.**Every agent gets a clean workspace it can destroy, so a bad run costs nothing and parallel runs cannot corrupt each other.
- **Verification runs before review.**By the time a diff reaches a person, it has already compiled, passed tests, and been checked for scope.

Miss the third and you have not built a software factory. You have built a machine that generates review work faster than you can absorb it, which is the failure mode the rest of this article is organized around avoiding.

Guillermo Rauch, Vercel's CEO, made the strategic case when Vercel open sourced its own reference platform for cloud coding agents, and he named the same systems this article draws on:

You've heard that companies like Stripe (Minions), Ramp (Inspect), Spotify (Honk), Block (Goose), and others are building their own "AI software factories". Why? [...] On a business level, the moat of software companies will shift from 'the code they wrote', to the 'means of production' of that code. The alpha is in your factory.

@rauchg, April 14, 2026


## The five stages every published software factory shares

Most of these systems are built on background coding agents, meaning agents that run unattended in their own environment rather than in your editor. Read enough of these architectures and the same skeleton appears, whatever the company calls it. Mastra ships it as six named stages in Mastra Factory. Spotify describes it as nested feedback loops. Stripe calls the pieces blueprints. The shape is the same.

| Stage | Stripe (Minions) | Spotify (Honk) | Shopify (River) | Ramp (Inspect) | 
|---|---|---|---|---|
| Intake | Slack message, emoji reaction | Fleetshift picks targets across repos | `@river`in a public channel | Assigned task | 
| Isolation | Pre-warmed EC2 devboxes | Kubernetes pods, constrained access | Disposable harness on durable sessions | Modal sandboxes from filesystem snapshots | 
| Tools | Toolshed, ~500 internal MCP tools | Internal systems over MCP | Credentials proxy and gateway | Tests, telemetry, feature flags, screenshots | 
| Verification | Lint and tests in under 5s, then capped CI | Deterministic checks, LLM judge, CI | Automated PR review mode | Visual and telemetry verification | 
| Merge gate | Human review after two CI runs | Human review | Human review | Human review | 

The stage order matters more than the tooling. Each gate stops work from reaching the next stage, and the expensive stages sit at the end.

One structural note before the details. Anthropic's managed agents architecture splits a software factory into a brain (the model and harness, stateless), hands (disposable sandboxes), and a session (a durable append-only event log). Shopify cites it directly in Under the River. If you build nothing else from this article, build the session log, because it is what lets everything else be disposable.

Running agents in parallel inside a single session is a different problem from running a fleet. Our guide to multi-agent orchestration with Codex covers subagents, fan-out, and worktree mechanics at that level.


## Stage 1: how work reaches an agent

Intake decides what is worth starting. Get this wrong and every downstream stage burns tokens on work that should never have begun.

The naive version assigns an agent to every open issue. The published versions all filter first. Sentry's Seer scores each incoming error for actionability and only investigates the ones that clear the bar. Shopify made a different call and routed intake through Slack, with one rule: agents work in public channels, never DMs. Shopify CEO Tobi Lütke described the constraint as deliberate:

River does not respond to direct messages. She politely declines and suggests to create a public channel for you and her to start working in. [...] Every conversation is therefore searchable. Anyone at Shopify can jump in.

@tobi, May 9, 2026


The argument is organizational rather than technical. A private agent session teaches one person and dies with the window.

A minimal intake filter is a label plus a query. This pulls issues that a human has explicitly marked as agent-eligible and small:

Microsoft's data says why the size filter belongs there. Across ten months on `dotnet/runtime`, agent PRs of 1 to 50 changed lines succeeded 76 to 80% of the time, while performance work landed at 54.5%. The published summary is blunt about the shape of it: Copilot's coding agent is "excellent at implementing well-specified changes, very good at investigating issues, and relatively poor at architecting solutions."

### The dedup gate, and why most versions of it fail open

There is a second question intake should ask, and almost nobody does: has someone already fixed this upstream?

For one engineer that is a nice-to-have. At Stripe's 1,300 merged agent PRs a week, spawning agents onto already-solved problems is exactly the waste a software factory exists to remove, and nothing surfaces it unless something checks.

Firecrawl's developer index covers this shape of question, since it indexes issue threads, merged pull requests, READMEs, and docs rather than blog posts about them. Scoped to the dependency in question:

Here is the part that matters for a gate. When you scope with `repos`, the response adds a top-level `repos` block, and it distinguishes two cases that otherwise look identical:

Both return HTTP 200 with `success: true`. The second returns zero results. So a gate written as `if not results: spawn_agent()` cannot tell "nobody has reported this" from "we have no coverage of that repo," and waves both through. That gate fails open.

Reading `repos[0].indexed` and treating `false` as *unknown* rather than *clear* is a one-line fix that turns a search box into a gate. Unknown routes to a human or retries unscoped.

For single-issue triage, where the question is whether a production bug is even yours, we covered the workflow in depth in fixing production bugs with a coding agent.

## Stage 2: pick an isolation model

Two agents editing one working directory is the fastest way to lose a day. This is where most homegrown software factories stall, because the obvious answer works until roughly the fourth concurrent agent.

Three models, in ascending order of cost and capability:

| Model | Isolates | Does not isolate | Good for | Real example | 
|---|---|---|---|---|
| Git worktrees | Files, branch | Ports, databases, installed deps, network | One machine, 2 to 5 agents | Claude Code's `--worktree` | 
| Containers | Files, deps, network, processes | Host resources | Conflicting dependencies, untrusted changes | container-use, Sculptor | 
| Cloud sandboxes | Everything, plus concurrency | Nothing you need | Fleet scale, unlimited parallelism | Stripe devboxes, Ramp on Modal, Spotify on Kubernetes | 

Worktrees are where to start. A git worktree is a second working directory on its own branch, sharing one repository. The examples below use Claude Code, though the pattern is the same in Codex; we compared how the two behave on longer tasks in Claude Code vs Codex. Claude Code creates one per session:

That lands in `.claude/worktrees/feature-auth/` on a branch named `worktree-feature-auth`. You can pin the isolation to a specific subagent instead, so a refactoring agent always gets its own tree:

### The part every worktree guide skips

A worktree is a fresh checkout, which means your `.env` is not in it. Neither is `node_modules`, and neither is a free port. This is the thing that actually breaks at agent four, and it is why teams conclude worktrees "do not scale" when what they hit was an unconfigured checkout.

Claude Code handles the gitignored-file half with a `.worktreeinclude` file at the project root, using gitignore syntax:

Ports and databases are still yours to solve. Assign them from the worktree name rather than hardcoding, and give each agent its own database. Add `.claude/worktrees/` to `.gitignore` while you are there.

Two more settings worth knowing. New worktrees branch from the repository's default branch, which is usually what you want for independent tasks; set `worktree.baseRef` to `"head"` when agents need your in-progress work. And you can branch a worktree straight from a pull request, which is how you point an agent at review feedback:

Cleanup is the failure mode nobody plans for. Interactive sessions prompt on exit, but non-interactive runs with `-p` do not clean up at all, and each one holds a lock until a later sweep releases it. If you are scripting a fleet, clean up explicitly:

Graduate to containers when agents start needing conflicting dependency versions, and to cloud sandboxes when concurrency matters more than the cost of running infrastructure. Ramp's Inspect makes the case for the top rung plainly: "When background agents are fast, they're strictly better than local: same intelligence, more power, and unlimited concurrency."

## Stage 3: give the agent hands, not just a brain

A model with a repository is an autocomplete. A model with your test runner, your telemetry, your feature flags, and your deploy tooling is a colleague. The gap between those two is the tool layer, and it is the least glamorous and most decisive stage in the software factory.

The published numbers say how seriously the leaders take it. Stripe's Toolshed hosts roughly 500 internal tools behind one MCP server, with controls that block destructive actions. Cloudflare runs an internal MCP Portal and generated `AGENTS.md` across more than 3,900 repositories. Ramp wires tests, telemetry queries, feature flags, and screenshot verification into every sandbox.

The leverage is that this is fleet-wide configuration, not per-agent setup. One MCP block reaches every agent:

A representative tool layer for a software factory looks roughly like this:

| Capability | Answers | Reached via | 
|---|---|---|
| Test and lint runners | Does it build and pass | Shell in the sandbox | 
| Telemetry | Did it break in production | Observability MCP server | 
| Feature flags | Is this path even live | Internal MCP tool | 
| Ecosystem and docs | Is this API still real | `firecrawl_developer_search` | 
| Deploy and rollback | Can this be undone | Internal MCP tool, gated | 

Rauch pushes the same point further, arguing that the context itself should live in one place:

Your software factory should be a monorepo. All your company context (design, marketing, sales, engineering, support…) in one place for agents to build upon

@rauchg, August 18, 2026


We arrived at the same place at Firecrawl. We run an internal monorepo called Firebrain that holds the complete company context, from product and engineering to marketing, sales, and support, in one place agents can read. The payoff is agents that already know how the company works before they start a task. It matters more than usual for us because the team is fully remote, spread across five continents and many time zones, so the repo is often the only colleague awake.

On instructions files, resist the urge to write a manual. OpenAI's Codex team runs a repository of roughly a million lines with an `AGENTS.md` of about 100 lines, which "serves primarily as a map, with pointers to deeper sources of truth elsewhere." Microsoft's data backs the general principle from the other direction: adding `.github/copilot-instructions.md` and firewall configuration moved agent success on `dotnet/runtime` from 41.7% to a sustained 71%. Preparation was worth more than any model upgrade in the same period.

For the tradeoffs between wiring tools over MCP versus letting agents shell out, see our MCP vs CLI comparison and the case for why agents prefer the CLI. To package a role so every agent in the fleet inherits it, see our overview of Claude plugins and skills.

## Stage 4: verification is where software factories actually differ

Every software factory generates code. What separates them is what happens between generation and a human's eyes, and this is the stage with the most counterintuitive published results.

Spotify's feedback loops post describes the clearest structure: nested loops, cheapest first.

- **Inner loop.**Deterministic verifiers selected automatically from what is in the repo. Format, compile, test. No model involved.
- **The judge.**A model evaluates whether the diff stayed in scope. It vetoes roughly 25% of agent sessions, and about half of those are recoverable by course-correcting the agent rather than discarding the work.
- **Outer loop.**CI and PR checks.

Spotify also ranks its failure modes, and the ranking is the useful part. A failed PR generation is an annoyance. A PR that fails CI is a burden on an engineer. A PR that passes CI and is functionally wrong erodes trust in the whole system. Design toward the first failure and away from the third.

### The confidence-score trap

Here is the most valuable negative result published in this space, and it will save you a quarter.

Faire built an internal reviewer called `faire-review` and tried the obvious quality lever first: filter comments by the model's own confidence score. It did not work. At the strictest gate, 65% of all comments were thrown away and the acceptance rate moved by 3%. Their write-up shows a comment scored 0.93 that was dismissed sitting beside one scored 0.35 that was accepted and fixed.

What actually lifted acceptance to 73% was better context about the change, better targeting of where a comment is worth making, and model-graded gates in place of a deterministic threshold. Self-reported confidence is not a quality signal. A second model asking "is this comment worth a human's time" is.

AI code review is the most widely deployed piece of the software factory, and Uber reached the same conclusion from a different direction on uReview, which now covers more than 90% of roughly 65,000 weekly diffs. Its principle is that precision beats volume, because developers lose confidence fast when suggestions are noisy. The payoff is measurable: 65% of uReview's comments get addressed in the same changeset, against 51% for human reviewers.

### The check that tests do not cover

Deterministic verifiers answer whether the code compiles and the tests pass. Neither question covers whether the page still looks right. A CSS change can clear every check in the inner loop and quietly break the phone view, because almost nobody writes tests asserting that a layout is reasonable.

That is worth closing, because it is precisely the failure Spotify ranks as most damaging: a change that passes CI and is wrong. Give the agent a browser, and pin the routine as a skill file so every agent in the fleet runs it the same way instead of improvising:

Step 1 is the load-bearing one. Reading the diff keeps the check scoped to what actually changed, rather than testing the whole app badly. We covered the fuller routine, including Devin's three-phase testing pattern and how the instruction wording decides whether any of it helps, in fixing production bugs with a coding agent.

The last step used to be where this broke. Producing screenshots is easy. Getting them onto the pull request meant a person dragging files into a comment box, which puts a human back in the loop at exactly the point you were trying to automate. GitHub closed that on September 1, 2026 with a repeatable `--attach` flag, in `gh` v2.99.0 and later:

Text after `#` becomes alt text, and `gh` falls back to the filename without it. The flag behaves the same on `gh issue create`, `edit`, and `comment`, and on `gh pr create`, `edit`, and `comment`. Images cap at 10MB; video is 10MB on free plans and 100MB on paid ones. GitHub Enterprise Server is not supported in this release.

Distributing the skill is its own fleet problem, and `gh skill` handles it in preview. Publish once, and every agent installs the same pinned version instead of carrying a local copy that drifts:

One honest limit. This proves the page rendered and the flow ran. It does not prove the change is well built or that it holds up next quarter. It kills the "said it was fixed, never actually ran it" failure, and it does not replace review.

### Order your checks by what they cost

Stripe caps agents at a maximum of two CI runs before handing off to a human. At more than 3 million tests, a CI cycle is the expensive resource, so the question becomes which checks earn a slot ahead of it.

The ordering falls out of unit economics. Lint and tests return in under five seconds locally. A scoped ecosystem lookup costs 2 credits per 10 results as of September 2026. A CI cycle across a suite that size costs vastly more than either. So when an agent writes against an API it half-remembers, checking that call against real issues, PRs, and migration guides before pushing is close to free:

Run live, that returns actual migration guides, including `Migrating-v4-to-v5.md` from the Deepgram JavaScript SDK, with citation URLs pinned to a commit SHA rather than a branch, so the reference does not drift.

Be realistic about the hit rate. On Firecrawl's own DevDex benchmark the index returns a correct answer in its top ten 63% of the time, against 45% for ordinary web search. That is a meaningful edge and roughly a third of queries still miss. It narrows the search, it does not end it. We covered the retrieval mechanics and the build-versus-buy decision in giving coding agents up-to-date documentation.

## Stage 5: the merge gate

An agent merge policy is the least written-about stage and the one that determines whether the software factory is an asset or a liability. The question is not whether an agent may open a PR. It is what has to be true before one lands.

Published policies cluster into tiers by blast radius:

| Change class | Example | Gate | 
|---|---|---|
| Mechanical, reversible | Dependency bump, lint fix, test migration | Automated checks, auto-merge | 
| Scoped feature or fix | Bug fix under 50 lines, new test coverage | One human review | 
| Agent-authored, non-trivial | Anything touching product behavior | Two reviews, per Faire's rule | 
| Domain-sensitive | Auth, payments, cryptography, data deletion | Named domain owner, regardless of author | 

Faire's rules are the most concrete published set, and two of the three are about review sequencing rather than review itself. Require two reviews on Copilot-authored PRs, the assignee's plus another human's. Do not request review from code owners until the PR already has one review, which stops agents from pulling domain experts into unfinished work. Mark ready for review only when the agent requests it.

Mastra's Factory separates the gate into three distinct approvals, which is a clarifying way to think about it: accepting the work into the queue, approving the plan, and merging the PR. Its docs are explicit that these are independent, and "acceptance authorizes the task to enter work. It's separate from approving the plan and from merging the eventual PR."

Then there is attribution. The Linux kernel's 2026 policy states that AI agents must not add a `Signed-off-by:` line, since that trailer is a legal assertion a model cannot make, and requires an `Assisted-by: AGENT_NAME:MODEL_VERSION` trailer instead. GitHub went the other way with Agent HQ, managing agent identity the way it manages human developers. Either is defensible. Silence is not, because six months later nobody can answer which changes came from where.

The far end of the spectrum is worth knowing about. OpenAI's Codex team reports that on its internal project "humans may review pull requests, but aren't required to," agents "often squash and merge their own pull requests," and the team has "pushed almost all review effort towards being handled agent-to-agent." That is a real position, taken on a greenfield codebase with no external users, by the team that builds the agent. Read the next section before adopting it.

*Source: Stephen Toub, "Ten Months with Copilot Coding Agent in dotnet/runtime", retrieved September 10, 2026. 396 agent pull requests received human commits; 482 did not.*

## What the numbers actually say

The vendor pages for this category cite the same five statistics. Here is the fuller picture, including the parts that do not flatter the idea.

**Agent PRs merge less often than human ones.** Microsoft's ten-month `dotnet/runtime` review is the only public dataset with a like-for-like comparison. Agent PRs merged 67.9% of the time against 87.1% for Microsoft engineers. The autonomy split is sharper still: PRs that received human commits merged 86.2% of the time, fully autonomous ones 55.1%. Revert rates held steady at 0.6% against 0.8%, so what does land is not noticeably worse.

*Source: Stephen Toub, "Ten Months with Copilot Coding Agent in dotnet/runtime", retrieved September 10, 2026. Across 6,181 pull requests.*

**Passing tests is not the same as being mergeable.** METR asked maintainers from scikit-learn, Sphinx, and pytest to review 296 AI pull requests that had already passed SWE-bench's automated grader. Roughly half would not have been merged, a gap of about 24 percentage points against the grader. The gap did not shrink across models from mid-2024 to mid-2025.

**Throughput and instability rise together.** DORA's 2025 report found AI adoption correlated with more delivery throughput and more delivery instability at the same time, concluding that AI is an amplifier of whatever an organization already is. Its 2026 ROI modeling puts first-year return near 39% with an eight-month payback, but only through a J-curve dip it attributes partly to a verification tax. Gains run 35 to 40% on simple tasks and often under 10% on legacy code.

**Volume has a maintainability cost.** GitClear's analysis of 211 million changed lines found duplicated code blocks rose eightfold in 2024, refactored lines fell from about 25% of changes to under 10%, and churn roughly doubled from 3.3% to 7.1%.

Set against that, the adoption numbers are real and large. Shopify reports 59,918 River sessions and 3,536 River-coauthored pull requests merged in a single 30-day window, and Lütke put the share plainly: "About one in eight pull requests merged into our codebase last week was authored by River, reviewed by us." Ramp reached about 30% of merged PRs with no mandate at all. Airbnb migrated about 3,500 test files in six weeks against an estimated 1.5 years by hand, with 75% done in the first four hours.

Notice what that last one is. It is a migration. So is Spotify's fleet work, and Amazon's Java version upgrades. The proven workload is mechanical change at scale, not greenfield product design.

## Where software factories break

**Review capacity, first and always.** On `dotnet/runtime`, merged agent PRs drew 16.5 review comments against 12.4 for human PRs, and the top two reviewers produced 36% of all feedback. That is a load-bearing pair of people. Faire names the same problem directly: PR volume went up while the number of human reviewers stayed flat.

**The last stretch is expensive.** Airbnb's long-tail files needed 50 to 100 retries each. The most-quoted line from Hacker News practitioners puts it well: "Agentic software development delivers if you're not too picky. The last 10% might last a very long time and cost many tokens."

**Parallel branches collide.** Worktrees cannot check out the same branch twice, and merge queues degrade as agent throughput rises. Unglamorous, and where fleets actually stall.

**Cost is not obviously worth it, and the honest advocates say so.** When one team published a manifesto arguing code should be neither written nor reviewed by humans and recommended spending $1,000 per engineer per day on tokens, the top Hacker News response was: "The site has zero benchmarks, zero defect rates, zero cost comparisons, zero production outcomes. The only metric offered is 'spend more money.'"

**Slop accumulates unless something eats it.** OpenAI's Codex team is candid that before they automated the loop, "our team used to spend every Friday (20% of the week) cleaning up 'AI slop.' Unsurprisingly, that didn't scale."

**Open source is absorbing the externality.** Steve Ruiz, who founded the canvas library tldraw, summarized the shift when his project paused external contributions: "Writing code is now easy; the real work is reviewing it." The most instructive data point is curl's. After years of rising low-quality reports, the maintainers took all of July 2026 off from vulnerability reporting and called it "possibly our best project decision in a long while." Daniel Stenberg notes the flood has not slowed for the other security teams he sits on. The only intervention that worked was closing intake, which is a gate argument.

## Give your software factory live web context with Firecrawl

The workload that pays for a software factory is migration and maintenance. Spotify's fleet-wide changes, Airbnb's test migration, Amazon's Java upgrade: all the same shape, all mechanical, all at a scale humans resent.

Every one of those is triggered by something that does not live in your repository. A release note. A deprecation. A breaking change three dependencies deep. Fleetshift-style tooling finds the targets inside your code, but what to migrate *to* is an external fact, and a software factory that only reads its own repo cannot see the change that created the work.

Firecrawl's developer index puts that live web context behind one search call: 70M+ issue threads, merged pull requests, READMEs, and documentation pages, most refreshed daily. Filters narrow it to the current answer rather than the popular one:

Same server, same `firecrawl_developer_search` tool, available to every agent in the fleet from one MCP block.

When the answer is not in a developer artifact, the same MCP exposes the general Firecrawl search and scrape pipeline: `firecrawl_search` returns ranked results across the open web, and `firecrawl_scrape` turns any URL, including JavaScript-rendered pages, into clean markdown the agent can reason over. That is the shape most factory-side lookups take. Search to find the two or three URLs that could plausibly answer the question, scrape to pull the current text off each one, then let the agent decide. A vendor status page, a partner API's changelog, a pricing table, a government schema: none of it lives in the repo, all of it lands in the same tool call, and none of it goes stale between runs.

A scraped page is also a place an attacker can plant instructions aimed at your agent. Firecrawl's prompt injection detection runs on every extraction, flags suspected injection attempts in the response, and lets a fleet operator drop or quarantine the offending content before it reaches the model. It is the piece that makes an open-web tool safe to hand a background agent.

## Start with one gate

Pick the stage where your agents currently produce the most rework, and put a gate there. Not a fleet, not an orchestrator, one gate. Every architecture in this article was assembled that way, and the companies that skipped straight to volume are the ones writing the honest posts about cleaning up on Fridays.

## Frequently Asked Questions

### What is an AI software factory?

It is the system around a coding agent rather than the agent itself: an intake queue that decides what work is worth starting, isolated workspaces so parallel agents do not collide, a shared tool layer, automated verification, and an explicit merge gate. Stripe, Spotify, Shopify, Uber, and Ramp have all published working versions.

### How many coding agents can you run in parallel?

There is no measured answer, and anyone quoting one is guessing. The practical ceiling is not compute, it is how many diffs you can review. OpenAI's Codex team reports 3.5 merged PRs per engineer per day, and Microsoft found the top two reviewers on dotnet/runtime produced 36% of all feedback on agent PRs.

### Do AI agents merge code without human review?

Some do. OpenAI reports that on its internal Codex project humans may review pull requests but are not required to, and agents often squash and merge their own. Most published software factories keep a human gate on anything that is not mechanical. Faire requires two reviews on agent-authored PRs.

### What percentage of pull requests do coding agents actually write?

It varies by an order of magnitude. Shopify reports 3,536 agent-coauthored PRs merged in one 30-day window, Ramp reports about 30% of merged PRs, and Stripe reports 1,300+ merged agent PRs per week. Microsoft's dotnet/runtime saw 878 agent PRs over ten months, which is 14% of Microsoft-originated PRs.

### Do agent-written pull requests get merged as often as human ones?

No. On dotnet/runtime, agent PRs merged 67.9% of the time against 87.1% for Microsoft engineers. Fully autonomous PRs merged only 55.1% of the time, while PRs that received human commits merged 86.2% of the time.

### Should agents use git worktrees or containers?

Worktrees are the cheapest isolation and work well on one machine, but they share the repository and cannot check out the same branch twice. Containers give real dependency and network isolation. Cloud sandboxes add unlimited concurrency at the cost of running infrastructure. Most teams start with worktrees and graduate when parallel agents start fighting over ports and databases.

### Does an AI software factory actually make teams faster?

The honest answer is that it moves the bottleneck. DORA's 2026 research models roughly 39% first-year ROI with an eight-month payback, but only after a J-curve dip it attributes partly to a verification tax. Gains run 35-40% on simple tasks and often under 10% on legacy code.

### What breaks first when you scale up coding agents?

Review capacity. Generation scales with spend and review does not. Every published account converges on this, from Uber running AI review on 90% of its 65,000 weekly diffs to OpenAI's team losing every Friday to cleaning up AI slop before they automated the loop.

### Can coding agents visually test their own UI changes?

Yes, if you give them a browser and a fixed routine. The pattern is to read the diff to find affected pages, screenshot each at desktop and mobile widths before and after the change, and attach the images to the pull request. Since September 2026 the gh CLI can do the attaching itself with a repeatable --attach flag in v2.99.0 and later, so no human has to drag files into a comment box.

### How is a software factory different from CI/CD?

CI/CD automates what happens after a human decides to make a change. A software factory automates the decision, the change, and the verification, and leaves the human at the merge gate. The pipeline stays; the thing feeding it changes.
