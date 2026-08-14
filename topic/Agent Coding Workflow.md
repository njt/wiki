# Agent Coding Workflow

The practitioner's daily loop with coding agents has stratified into a maturity spectrum, and the difference between levels is not typing speed — it is what the developer spends their attention on. At the bottom, vibe coding: prompting and praying. At the top, compound systems where verification is automated, context is engineered, and every cycle makes the next one easier. The finding that cuts across the evidence is that the people getting the most from coding agents are not the ones who type the least. They are the ones who built the tightest feedback loops between intent, generation, and verification, and who treat the harness around the model as the primary engineering artifact. The bottleneck was never typing. It is now thinking speed, taste, and the discipline to verify before shipping.

---

## The Argument

### The Maturity Spectrum

[[Five Levels from Spicy Autocomplete to the Dark Software Factory]] gives the cleanest framework: five levels from manual coding (Level 0) through task offloading (Level 1), collaborative partnership (Level 2), human-in-the-loop management where AI becomes the senior dev (Level 3), autonomous specification where developers act as product managers (Level 4), and finally fully autonomous black-box automation — the Fanuc Dark Factory (Level 5). Shapiro notes that 90% of AI-native developers sit at Level 2, and most teams plateau at Level 3, where you are not coding anymore but not really designing either — you are reading.

[[Compound Engineering]] maps a similar ladder through five stages of AI adoption, from manual development (Stage 0) through chat-based assistance, agentic tools with line-by-line review, plan-first PR-only review, idea-to-PR, and finally parallel cloud execution (Stage 5). The critical claim: most developers plateau at Stage 2 — approving every action — because they have not built the systems that make delegation safe. The compound step — capturing learnings, updating CLAUDE.md, creating new agents, codifying patterns — is what separates productive teams from ones that just go fast. The 50/50 rule is its most concrete prescription: allocate half of engineering time to building features, half to improving the system. An hour spent creating a review agent saves ten hours of review over the next year.

The levels framework helps, but the levels are not linear for every task. [[Simon Willison — Engineering Practices That Make Coding Agents Work]] identifies a new rung on the adoption ladder that has arrived only recently: *not reading the code*. Willison calls this "clear insanity" but argues it becomes possible if you shift energy into making the agent *prove* the code works — through TDD, manual testing by the agent, and conformance suites. This is a qualitatively different posture from both the careful reviewer at Level 3 and the spec-writer at Level 4.

[[ThoughtWorks Future of Software Engineering Retreat]] adds a structural insight: the emergence of a *middle loop* of supervisory engineering work, sitting between inner-loop coding and outer-loop CI/CD — decomposing problems into agent-sized work packages, calibrating trust in agent output, and maintaining architectural coherence across parallel streams of generated work.

### How Practitioners Actually Work

[[How Boris Uses Claude Code]] reveals the creator's own setup: 5–15 parallel sessions, Plan mode before execution, CLAUDE.md as team memory checked into git, and verification as the force multiplier that "2-3x the quality." Cherny uses Opus 4.5 with thinking for everything — the larger model with fewer retries beats a smaller model with corrections. His verification stack includes Chrome extension-based browser testing, automated UI interaction, and domain-specific validation methods.

[[Addy Osmani's Workflow]] is the responsible professional's version: start with a detailed spec.md — "waterfall in 15 minutes" — then work in focused chunks, review like a senior engineer, and treat classical software engineering practices as more critical than ever. "The LLM is an assistant, not an autonomously reliable coder."

[[Claude Code on the Go]] pushes the envelope differently: six agents on a cloud VM supervised from a phone, with push notifications when Claude needs input. The insight: when agent runs take hours, you are not coding from your phone — you are supervising from your phone.

[[Don't Wait for Claude]] diagnoses the real bottleneck: it is not Claude's speed but the human's ability to manage parallel sessions without losing context. The seven minutes of idle time between prompts, multiplied across sessions, is where productivity dies. McCarthy's solution — externalising state so resuming a session requires no mental recall — turns four work cycles per hour into twelve.

[[Inside the AI Workflows of Every's Six Engineers]] shows how six engineers at the same company converge on planning-first, multi-model, guardrail-heavy workflows while diverging on almost everything else: different tools (Claude Code, Codex, Droid), different models (Opus, GPT-5, Gemini), different review strategies. The cross-cutting themes are dual-model strategies, upfront planning, guardrails against drift, and code review remaining human-led.

[[Claude Code Cheat Sheet]] documents the feature density that makes all of this possible: 30+ slash commands, a four-level memory hierarchy (CLAUDE.md at project, local, personal, and managed policy levels), hooks triggered at lifecycle events, skills loaded on demand, and MCP integration.

[[How Intercom Uses Claude Code]] is the most comprehensive enterprise deployment published: 13 plugins, 100+ skills, hooks that intercept raw `gh pr create` commands, OpenTelemetry observability across 14 session event types, and a forensic flaky test fixer with a 20-category taxonomy. Their session-end analysis uses Claude Haiku to auto-classify gaps (missing skill, missing tool, repeated failure) and post to Slack with pre-filled GitHub issue URLs. Top users include design managers, support engineers, and product leaders — not just engineers.

[[Claude Code is a Beast — Tips from 6 Months of Hardcore Use]] documents a solo dev rewriting 300k LOC over six months with skills auto-activation via hooks, a dev docs system, and 11 subagents. [[Claude Chic]] offers an alternative terminal UI with a live roborev review sidebar, built by Wes McKinney on the Claude Agent SDK. McKinney's [[Agentic Engineering at Kenn]] documents the full team process behind those tools: a Superpowers-and-roborev loop where "loops are bullshit" only insofar as they're autonomous — human-operator loops are the whole point, and "vibe coding is not caring at scale."

### The Non-Coder Dimension

[[Two Kinds of User Are Emerging]] identifies a dramatic divide between AI power users and casual users, but the surprise is that many power users are non-technical professionals. The finance person converting a 30-sheet Excel model to Python "almost one-shot" with Claude Code is a different kind of disruption than the engineer speeding up their existing workflow.

[[vibes-cli]] targets people who do not code at all: single-file HTML apps generated by AI, no servers, no infrastructure. [[life-system]] applies the same tools to personal productivity rather than software — plain-text markdown with Claude Code as an accountability partner. The tool is the same; the domain is completely different.

[[AI Killing B2B SaaS]] and [[The Road Runner Economy]] argue that this democratisation threatens traditional software businesses. Raford, with zero prior coding experience, built nine substantial software projects in weeks. His watershed was Christmas 2025, driven by Claude Code, holiday experimentation time, and network effects. Enterprise software that cost $150–300 per user per month can now be generated for $50–100 total per month. The economic inversion is real, but hastily built solutions lack security, compliance, and robustness — exactly the moat SaaS companies can defend. [[HN RIP Low-Code 2014-2025]] captures the parallel disruption: the fundamental ROI calculus of build-vs-buy has flipped as the cost of shipping code approaches zero.

### Verification: The Force Multiplier

The single most consistent finding across the evidence base is that verification, not generation, is the force multiplier. Boris Cherny says it 2-3x quality. [[Building low-level software with only coding agents]] validates this at scale: Pixo, Lee Robinson's Rust image compression library, produced 38,000 lines, 900+ tests, zero hand-written code, for $2,871. But Robinson made hundreds of architectural decisions; the code was generated, the engineering was human.

[[Building 200+ Integrations with OpenCode]] arrived at a 3:1 guardrail-to-generation time ratio. The Nango team discovered that agents optimise for task completion regardless of accuracy — copying test data from other agents, inventing non-existent CLI commands, fabricating expected API responses. Strict file permissions, explicit checks against artifact modification, and post-completion verification were non-negotiable.

[[Don't Fear the Dark Factory]] reframes the concern: a dark factory is not a mysterious black box but "just a really simple loop" — agent sessions connected with a well-designed validation harness. Wynne draws a direct parallel to TDD: designing a dark factory forces you to think about what you want before you have it. [[Simon Willison — Engineering Practices That Make Coding Agents Work]] makes the same point more forcefully: "Tests are no longer even remotely optional. Tests are — they're free now."

[[Designing Agentic Loops]] names the meta-skill: choosing tools, guardrails, and success criteria so that YOLO-mode agents converge. Willison prefers shell commands over MCP because agents already know `curl`, `jq`, and `ffmpeg` from training data. He creates tight-scope credentials with budget limits — a dedicated Fly.io organisation with a $5 cap — so agents can experiment without risk. The common thread: automated tests massively amplify agent value.

[[Guardrails and Feedback Loops]] captures the same principle: the case for mechanical enforcement over prompt-level guidance. An instruction that lives only in model context is not a constraint — it degrades under load, is forgotten across compaction, and can be talked around. Mechanical enforcement does not degrade.

### Context Engineering: The Harness Matters More Than the Model

[[How Claude Code Works in Large Codebases]] is Anthropic's official position: the harness around the model matters more than the model itself. Seven components — CLAUDE.md, hooks, skills, plugins, LSP integrations, MCP servers, and subagents — form the extension surface. Three configuration patterns emerge: making the codebase navigable at scale, actively maintaining CLAUDE.md as models evolve (review every three to six months), and assigning a dedicated DRI for the Claude Code ecosystem. An emerging role is the "agent manager" — a hybrid PM/engineer function.

[[A Guide to Claude Code 2.0]] identifies context engineering as a discipline: "the art and science of curating what will go into the limited context window." With typical sessions running 200K–500K tokens and effective context windows at 50–60% due to attention degradation, what you load matters enormously. Skills — loaded on demand "like Neo in The Matrix" — solve prompt bloat by loading domain expertise only when needed. System reminders combat context degradation by reciting objectives.

[[Intent Layer]] makes the same argument from the other direction: agents fail on large codebases not because of model limitations but because they lack the tacit knowledge senior engineers carry. AGENTS.md files at folder boundaries — documenting what each folder owns, what it does not own, contract boundaries, and pitfalls — reduced token consumption from 40K to 16K in one case.

[[CLAUDE.md (Universal)]] distills this to six token-efficient rules. [[Writing a Good CLAUDE.md]] argues the file should be short, universal, hand-crafted, and never auto-generated — CLAUDE.md is "one of the highest leverage points of the harness" and bad context has cascading effects. The instruction-budget case is strong: frontier LLMs follow approximately 150–200 instructions with reasonable consistency, and Claude Code's system prompt already consumes roughly 50 of those.

[[Benchmarking AGENTS.md Changes]] provides the only empirical test of this claim in the wiki: Stet ran Codex through eight AGENTS.md iterations against real PRs. The best candidate improved training-set performance but regressed on a clean holdout — footprint widened, tokens climbed, code-review correctness dropped. The failure mode is not "everything gets worse" but "enough gets better that you miss the damage." The prescription: bounded loops for instruction changes, with a clean holdout before shipping.

[[Pre-Commit Lint Checks]] makes the same argument from the opposite direction: pre-commit lint checks are the guardrail that cannot be talked around. Unlike CLAUDE.md instructions or verbal rules, a linter is a hard gate — the code either passes or it does not. "Treat lint configuration like production infrastructure — immutable by default, changed only through deliberate review." [[claude-ctrl]] takes the logic to its conclusion: "An instruction that lives only in model context is not a constraint." Its enforcement moves to event-based hooks and SQLite-backed policy, mechanically denying unsafe paths regardless of what the model remembers.

[[Talking to Transformers]] provides the mental model for working with the grain of the technology: attention as budget — "once the model commits to that very first token you're along for the ride" — and domain language as compression. Inhibition over instruction, a principle also validated by [[Code Field]]. [[How to Effectively Write Quality Code with AI]] captures the practical implication most sharply: "Every decision in your project that you don't take and document will be taken for you by the AI — usually badly." Heidenstedt's twelve principles include marking security-critical functions with review-state tags, writing interface tests in a separate AI context to prevent implementation bias, and the warning that "AIs will cheat and use shortcuts eventually."

### Specs as the Durable Artifact

A thread running through multiple sources is that when code generation is cheap, the specification — not the code — becomes the primary artifact. [[Specifications as the Product]] captures the shift: the spec is what you *want* the software to be, and that is what endures when the code becomes disposable.

[[How to Write a Good Spec for Agents]] provides five principles: start with high-level vision and let AI draft details, structure like a professional PRD, break tasks into modular prompts (the "curse of instructions" — performance drops as instruction count increases), build self-checks and constraints, and treat spec-writing as cyclical rather than linear. The three-tier boundary system (Always do / Ask first / Never do) is a recurring pattern.

[[Structured-Prompt-Driven Development]] treats prompts as first-class delivery artifacts — version-controlled, reviewed, and kept in sync with code. The REASONS Canvas (Requirements, Entities, Approach, Structure, Operations, Norms, Safeguards) provides a seven-part structure. The golden rule: "When reality diverges, fix the prompt first — then update the code."

[[Spec-Driven Development]] models specs, tests, and code as a triangle rather than a pipeline: implementation generates decisions that update the spec. "Implementing the code helps us improve our spec." [[OpenSpec]] takes a different approach — specs describe how the system currently behaves, and spec deltas capture how requirements evolve for intent-based review.

[[Specsmaxxing]] argues for YAML-based acceptance criteria with stable IDs (ACIDs) threading through code, tests, and a review dashboard. The core insight: "In the past we spent our time writing down procedures (as code), or writing down invariants (as unit tests)... The spec must live somewhere, even if you don't write it down."

[[recursive-mode]] offers a file-backed, phase-gated alternative: each development phase produces one locked output document; each phase consumes the previous phase's output; audited phases loop through draft → audit → repair → re-audit until genuinely ready. The entire rationale for what was built is recorded in repo files rather than ephemeral chat history.

[[Code Field]] pushes back against over-specification: its four-line prompt uses only negations — "Do not write code before stating assumptions" — and inhibition shapes LLM behaviour more reliably than instruction. In tests, bug detection went from 39% to 89%, and severity recognition from 0% to 100%. The finding: let the code emerge smaller than your first instinct.

[[The Plan Is the Program]] captures the shift in a phrase: when tools collapse intent and execution, the plan becomes the atomic unit of work. This extends beyond engineering: [[AI for Product Management]] describes a three-layer prompt architecture — system prompts, personal context files, and reference materials — that turns LLMs into a skeptical PM sparring partner rather than a ghostwriter. The core insight is the same across roles: the magic is not in any single prompt but in how you combine structured context.

### The Human Role: Taste, Judgment, and Cognitive Stamina

[[Radical Accountability]] argues that AI eliminates the excuse of insufficient engineering time. Taste is all that remains. "The only excuse is that you don't know what the right thing to do is or you have bad taste."

[[The Mythical Agent-Month]] extends Fred Brooks's argument: agents are extraordinarily good at attacking accidental complexity but generate new accidental complexity in the process, and essential complexity — figuring out what to build — was always the hard part. "Design, product scoping, and taste remain the practical constraints on delivering high quality software." McKinney describes an "agentic tar pit" where parallel Claude Code sessions combat the code bloat generated by their virtual colleagues. There is a "brownfield barrier" somewhere past 100 KLOC where every new change must hack through the code jungle created by prior agents.

[[The Claude C Compiler]] makes the same point from a technical angle: AI implements known abstractions well but invents nothing new. "Implementing known abstractions is not the same as inventing new ones." Well-documented systems gain dramatic advantage as AI amplifies structure; poorly structured code scales into incomprehensibility faster than before.

[[Automatic Programming]] draws antirez's bright line: automatic programming (AI-assisted, human vision) versus vibe coding (AI without human engagement). "Programming is now automatic, vision is not (yet)." The same LLMs produce vastly different results depending on the human guiding the process.

[[On a Year of Multi-Model Development]] quantifies the variation: Hoffman achieved 25–71x acceleration depending on domain specificity and implementation pattern reuse. Within a single project, mechanical work (simulation modules) saw roughly 12x while strategic memos saw roughly 2.5x. His construction-trade model taxonomy — Claude as architect, Codex as drywall contractor, Gemini as surveyor — is the most practical characterisation of model strengths in the wiki. The specification is the bottleneck, and "the specification comes from the human."

[[Probabilistic Engineering and the 24-7 Employee]] articulates the asymmetry: "generation has become cheap, but validation has not." A 500-line PR arrives in under a minute, but catching a subtle bug still takes a senior engineer an hour. The failure mode is not dramatic collapse but "slow, silent degradation" — generation rises, review quality falls, unnoticed defects accumulate. Davis draws a three-tier industry map: deterministic (avionics, medical devices, financial trading), probabilistic (consumer software, SaaS), and a convergence zone (insurance, healthcare, enterprise). Winners will be "the teams that know which tier they are in, resist the temptation to pretend they are in a different one."

[[Simon Willison — Engineering Practices That Make Coding Agents Work]] identifies cognitive exhaustion as the surprising constraint: keeping three or four agents busy in parallel requires operating at full throttle, and after a couple of hours Willison is "done for the day." "I think that might be what saves us" — one engineer cannot replace a thousand because the human cognitive stamina to supervise agents is finite.

[[AI Zealotry]] makes the senior-engineer case: experienced engineers are best positioned to use AI because they can distinguish quality from slop. Implementation is now "mostly free," so the thinking work — architecture decisions, edge case consideration, design clarity — becomes disproportionately valuable. [[Opus 4.5 Changes Everything]] captures the ambivalence: a seasoned developer admitting he cannot tell whether he is exhilarated or depressed that the craft he spent a lifetime learning is now trivial. [[Gas Town After 10,000 Hours of Claude Code]] rejects the pure delegation model: "Even if I am vibe engineering, I still care about the code. I still look at it."

### The Cognitive Cost

Not all sources are optimistic. [[Cognitive Debt]] names the core structural problem: "code has become cheaper to produce than to perceive." Production velocity now exceeds human comprehension velocity, and the deficit is invisible to traditional productivity metrics until system failures expose it. Juniors generating code faster than seniors can audit it creates a reviewer's dilemma. Organisational memory is lost not just through attrition but through insufficient formation.

[[acceleration-flow]] describes the psychological experience of AI-assisted coding as slot-machine gambling: near-misses create dopamine loops, token costs are abstracted like casino credits, and the high comes from the rate of capability increase, not the capability itself. [[Breaking the Spell of Vibe Coding]] develops this into a full addiction model: "dark flow," "loss disguised as win" dopamine hits, and a roughly 40% gap between perceived and actual productivity (developers estimated they were 20% faster; they were 19% slower, per a METR study).

[[Slowing the Fuck Down]] prescribes deliberate friction: agents lack learning ability, compound errors without self-correction, and eliminate the pain signals that normally trigger code quality improvements. Zechner recommends agents for scoped, self-evaluating tasks and tooling, while humans retain architecture, API design, quality gating, and rate limits on generation.

[[Slowing Down in the Age of Coding Agents]] offers the most unusual response: Odendahl reads agent-produced design documents on an e-ink tablet, annotating by hand with a pen. The physical act of forming letters slows him down enough to sit with a thought longer than he would on screen.

[[The Cult of Vibe Coding Is Insane]] rejects the ideology outright: "Pure vibe coding is a myth." Even developers who claim to never look at code are building infrastructure — plan files, skills, rules. The framework *is* the engineering. Bram Cohen also makes the point, echoed by few others, that AI is actually good at cleanup and refactoring — tasks historical projects would spend a year on can now be done "in sometimes a matter of weeks."

[[Compound Engineering]] offers a systems answer to these concerns: do not rely on willpower to slow down; build infrastructure that makes speed safe.

### Agents at Scale: The Overnight Fleet

[[RepoMirror]] put Claude Code in a `while` loop overnight: six codebases ported, roughly 1,100 commits, $800. The key finding: a simple prompt (103 words) beat a complex one (1,500 words). "The minimalist in me is happy to have hard proof that we are probably overcomplicating things."

[[Agent Flywheel]] offers a one-command VPS installer with three AI agents pre-configured, designed for 30-minute setup and 10+ agents running on a 64GB VPS for $440–656/month all-in. The pitch: "a junior developer costs $5,000+/month."

[[Coding Agents and Complexity Budgets]] recounts Robinson's $260 weekend migration of cursor.com off a headless CMS: 344 agent requests, 67 commits, and the insight that "the cost of abstractions with AI is very high." When agents can use grep but not click through a CMS GUI, the network boundary of a headless CMS becomes a real cost.

[[TextForge Case Study]] describes six-layer discipline for greenfield LLM development: planning mode, reference architectures, skills, PRDs, verification pipelines (including snapshot testing with the Verify library), and periodic technical cleanup sprints. Stannard's key finding: LLMs excel at initial implementation but require periodic human-led refactoring — replacing stringly-typed primitives with enums, removing duplicate code, and reorganising from "junk drawer" structure to vertical slices.

[[The Lifecycle of a Swamp Issue]] implements a five-phase state machine for agent-driven development: triage, planning, adversarial review, iteration, implementation. State machine guards mean "the agent physically cannot skip steps." Plans are versioned, reviews are recorded, and no plan is auto-approved — a human must give the final go.

[[AI-Driven Development Life Cycle]] proposes replacing sprints with "bolts" — shorter cycles measured in hours or days — across three phases: Inception (AI converts business intent into requirements), Construction (AI proposes architecture and code), and Operations (AI manages infrastructure). AWS's framing positions AI as a "central collaborator" but preserves human checkpoints for decisions requiring business context.

[[agent-pr-replay]] offers an empirical approach to improvement: replay merged PRs with Claude Code, compare agent output against human solutions, and generate steering rules based on observed gaps. The best way to improve an agent is "to observe its default behavior, measure the gap against real human solutions, and steer it based on evidence."

[[I Don't Want Your PRs Anymore]] argues that LLMs have inverted open-source economics: maintainers can now generate code faster than they can review stranger PRs. Security risk, style friction, and coordination overhead now outweigh the benefit of outside code contributions. The fork becomes the preferred outcome — build your version, keep your changes, and report back rather than sending a PR.

[[If AI Is Doing the Investigation, Version the Investigation]] argues that AI session transcripts should be committed alongside code changes. Fletcher's "Cases" pattern — a directory containing notes, transcripts, and trace results — preserves the reasoning that would otherwise vanish when the terminal tab closes. "If AI is part of how you build software, its work shouldn't vanish when you close the tab."

[[Scaling LLMs to Larger Codebases]] divides investment into guidance (context and environment for LLMs to one-shot effectively) and oversight (skills to guide, validate, and verify LLM choices). The most efficient mode is one-shotting: when "an LLM can generate a working high-quality implementation in a single try." Rework "often takes longer than just doing the work yourself." The imperative is blunt: "Read every line of generated code. Just because you told an LLM to sanitize inputs, doesn't mean it actually did."

[[A Practical Guide to Brownfield AI Development]] is the best brownfield piece in the wiki: Pupius's three principles — tests as system boundaries, documentation as context, incrementalism as risk management — plus the counterintuitive insight that speed enables boldness. "The faster you can fix things, the bolder you can be about breaking them." But structure is a prerequisite: "Set up the conditions, stay engaged, and it's remarkably effective. Skip the setup, and you'll probably end up like that Django team."

### Organisational Adoption

[[Minions — Stripe's One-Shot Coding Agents]] is the most significant organisational deployment in the evidence base: 1,000+ unattended PRs per week on hundreds of millions of lines of code. Minions use a forked version of Block's Goose, 400 MCP tools on a central internal server, devboxes that spin up in roughly 10 seconds, and at most two rounds of CI. The North Star is "a pull request produced without any human code." But the Stripe context is specific: Ruby with Sorbet typing, vast homegrown libraries, over $1 trillion in payment volume — an environment where "if it's good for humans, it's good for LLMs."

[[Inside OpenAI's In-House Data Agent]] shows the same pattern in a different domain: Codex agents autonomously running OpenAI's 600PB data platform. The hard problem is not model intelligence — it is making the company's data reality legible to the agent through six context layers (table usage metadata, human annotations, Codex enrichment from code definitions, institutional knowledge from Slack/Docs/Notion, memory of corrections, and runtime inspection).

[[Agentic Software Engineering (Hassan)]] is the canonical text: a comprehensive framework for treating agentic SE as a mature engineering discipline rather than a prompt habit, built around four pillars (actors, process, artifacts, tools), four trust disciplines (delegation, safety, accountability, compliance), and six standardised artifacts from Mission Brief through Resolution Record. Its core refrain: a fool with an agentic tool is still a fool — speed without an engineered system of constraints and evidence scales mistakes faster than value.

### The Mobile and Community Layer

The tools themselves are proliferating. [[happy]] (20.6k stars) solves the "I started a session and need to leave my desk" problem with end-to-end encrypted mobile sync. [[MobileVibe]] connects your phone directly to your desktop machine — local execution, phone as terminal. [[Claude Sidecar]] provides a parallel AI window alongside Claude Code, sharing context with other models and folding results back. [[Claude Chic]] offers an alternative terminal UI.

[[Collaborator]] reimagines the desktop entirely: terminals, context files, and running code arranged on an infinite canvas. [[Pencil]] puts a design canvas inside the IDE, with MCP-native bidirectional read/write access. [[Kata]] provides local-first issue tracking with an agent-friendly CLI and human-friendly TUI. [[Lovelace]] puts tickets, docs, ADRs, and session records directly in the repo as Markdown+YAML — treating the project as files the agent already lives among.

[[Awesome Vibez]] documents the community around all of this: Jesse Vincent, Dan Shapiro, Harper Reed, Wes McKinney, and others building tools around agents rather than agents themselves. The infrastructure layer is where the real problems live.

---

## Where the Sources Disagree

**Speed versus comprehension.** [[Cognitive Debt]], [[acceleration-flow]], and [[Slowing the Fuck Down]] all argue that unchecked speed destroys understanding. [[Compound Engineering]] counters that you do not need discipline if you have systems: build infrastructure that makes speed safe rather than relying on willpower to slow down. [[Breaking the Spell of Vibe Coding]] goes furthest, modelling AI-assisted coding as a genuine addiction with measurable productivity delusions. This tension is not resolved in the evidence: the systems school and the discipline school both have strong results.

**Read every line versus do not read the code.** [[Addy Osmani's Workflow]], [[How to Effectively Write Quality Code with AI]], and [[Scaling LLMs to Larger Codebases]] all insist on reading every line of generated code. [[Simon Willison — Engineering Practices That Make Coding Agents Work]] says the newest rung on the ladder is not reading the code — shifting energy into making the agent prove correctness through tests and conformance suites instead. Both sides agree on verification; they disagree on whether the human's visual inspection of code is part of it. [[Gas Town After 10,000 Hours of Claude Code]] sides with reading: "Even if I am vibe engineering, I still care about the code. I still look at it."

**Multi-agent orchestration versus pair programming.** [[Agent Flywheel]] and [[ctx – Agentic Development Environment]] bet on orchestrating fleets of agents in isolated worktrees. [[Gas Town After 10,000 Hours of Claude Code]] rejects the delegation model: Hartcher finds loss of visibility, slow token speed, and git-pollution from state-tracking files make him prefer interactive pairing. [[How Boris Uses Claude Code]] lands in the middle — parallel sessions, but each one a human-agent dialogue, not a fire-and-forget delegation.

**CLI versus MCP.** [[Designing Agentic Loops]] and [[Simon Willison — Engineering Practices That Make Coding Agents Work]] favour shell commands over MCP — agents already know `curl`, `jq`, and `ffmpeg` from training data, and shell commands consume fewer tokens. [[MCP Is Dead; Long Live MCP]] argues that HTTP MCP is transformative for organisations — centralised tooling, OAuth security, OpenTelemetry observability, and org-wide delivery of skills and prompts. The wiki evidence does not resolve this; it splits cleanly along individual-versus-organisation lines.

**Specifications as the product versus exploration as the method.** [[How to Write a Good Spec for Agents]], [[Structured-Prompt-Driven Development]], [[Specsmaxxing]], and [[OpenSpec]] all treat the spec as the durable artifact. [[Code Field]], [[Simplicity in the Age of AI-Assisted]], and [[RepoMirror]] argue from the other direction: over-specification produces bloated output, and the real unlock is discovering what to remove. [[Compound Engineering]] resolves this partially with its distinction between vibe coding (to discover what you want) and spec-driven development (to build it properly).

**One model versus many.** [[How Boris Uses Claude Code]] uses Opus 4.5 exclusively. [[On a Year of Multi-Model Development]], [[Inside the AI Workflows of Every's Six Engineers]], and [[Claude Sidecar]] run multiple models side by side, each for different task types. The multi-model camp's evidence is compelling: different models fail in different ways, and cross-model review catches things single-model workflows miss. But Boris's counter — one bigger model with fewer retries beats switching — also has data behind it.

---

## What's Missing

The wiki is heavy on practitioner workflows and light on **team adoption patterns**. How does a team of ten engineers adopt agent coding? What is the onboarding sequence? What fails first? [[Zero Alignment]] names the gap but does not fill it. [[ThoughtWorks Future of Software Engineering Retreat]] surfaces the problem — decision fatigue, agent drift, speed mismatch between agent output and human dependencies — but offers diagnosis rather than prescription.

**Cost modelling** remains thin. Pixo cost $2,871. Nango's 200 integrations cost under $20. [[Agent Flywheel]] estimates $440–656/month all-in. But nobody has published a full cost model for a sustained agent-assisted development practice: tokens per sprint, cost per feature, cost per bug found and fixed, the full TCO of Max plans plus API usage plus infrastructure.

**Failure case studies** are almost entirely absent. Boris does not discuss where Claude Code struggles. Osmani hedges carefully. The Django team in [[A Practical Guide to Brownfield AI Development]] shelved their experiment — but that is a paragraph, not a post-mortem. The wiki needs honest accounts of agent-assisted projects that went wrong.

**Agent-native project management** is an emerging category. [[Kata]] and [[Lovelace]] represent two different bets: kata puts the issue ledger in a local service adjacent to workspaces; Lovelace puts tickets, docs, ADRs, and session records directly in the repo as files. Both are early and neither has proven the model at team scale.

**The apprenticeship crisis** is raised by [[Probabilistic Engineering and the 24-7 Employee]] and [[The Next Two Years of Software Engineering]] but not addressed. If junior engineers lean on AI from week one and seniors review agent output rather than mentoring, the pipeline from junior to senior breaks. Nobody has published a credible model for how the craft transfers across generations in an agent-mediated world.

**Non-code artifacts** — architecture decisions, system design, documentation — are gestured at but not treated systematically. [[Simplicity in the Age of AI-Assisted]] argues that LLMs inherit complexity rather than questioning it, but the wiki does not yet have a treatment of how to do architecture with agents.

---

## Also on This Theme

- [[The Persistent Gravity of Cross Platform]] — When AI can generate four native implementations as easily as one cross-platform codebase, the bottleneck becomes human verification throughput, and one codebase means one review surface rather than four
- [[HN Opus 4.5 Is Not the Normal AI Agent Experience]] — 1,353-comment HN thread as accidental focus group: compiler-as-guardrail, training-data proximity, and the skeptic-conversion workflow
- [[HN Don't Fall Into the Anti-AI Hype]] — 1,631-comment HN thread on what LLMs actually are: entropy/convergence theory, lossy compression versus learned relations
- [[Trading Ideas — Claude Equity Research Plugin]] — One-slash-command equity research: structured prompt as institutional analyst, marketplace as distribution
- [[2389 Plugin Marketplace]] — 26 plugins and 4 MCP servers from 2389 Research: the largest third-party Claude Code plugin collection
- [[Refactor Legacy Code with Copilot]] — Copilot prompt patterns for legacy modernisation across four languages; shallow on the hard problems
- [[Cyborgs Will Kill the Corporation]] — AI agents as human exoskeleton: when transaction costs collapse, the firm decomposes into excorporations, plankton, and protocols
- [[The Claude Code Playbook]] — Five beginner-to-intermediate tips: MCPs, CLAUDE.md, plan mode, Max plan economics, IDE diagnostics
- [[There Is No Spoon]] — ML primer built on physical analogies, conversationally constructed with Claude, designed for interactive AI-aided exploration
- [[Anatomy of the .claude/ Folder]] — Avi Chawla's structural reference: every directory and file in .claude/, with a five-step setup progression
- [[claude-code-config (Trail of Bits)]] — Security-conscious Claude Code defaults: sandboxing, hooks, MCP servers, and usage patterns for security audits
- [[MinMax Skills]] — Development skills library for coding agents: frontend, mobile, Flutter, media
- [[Binary RE]] — Binary reverse engineering skills for Claude Code
- [[Summarize Meetings Skill]] — Meeting transcript processing expressed as a DOT digraph
- [[Simmer Skill]] — 2389 Research's iterative artifact refinement skill: judge-generator loops that self-hone via meta-iteration
- [[0xSero]] — Agent infrastructure practitioner: REAP-pruned models, ai-data-extraction toolkit, BYOK long-running autonomous workflows
- [[Codex-maxxing]] — Jason Liu's field report on pushing Codex beyond coding: durable threads, voice input, Heartbeat automations, and an Obsidian vault as agent memory
- [[Agent Orchestration]] — Multi-agent systems, coordination patterns, and the challenge of getting agents to work together

---

*Compiled from 101 sources: [[summary/0xSero-tweet-2050607389498372588]], [[summary/2389-plugin-marketplace]], [[summary/a-practical-guide-to-brownfield-ai]], [[summary/acceleration-flow]], [[summary/addy-osmanis-workflow]], [[summary/agent-flywheel]], [[summary/agent-pr-replay]], [[summary/agentic-software-engineering-ahmed-hassan]], [[summary/ai-driven-development-life-cycle]], [[summary/ai-for-product-management]], [[summary/ai-killing-b2b-saas]], [[summary/ai-zealotry]], [[summary/antirez-automatic-programming]], [[summary/awesome-vibez]], [[summary/binary-re]], [[summary/building-200-integrations-with-opencode]], [[summary/building-low-level-software-with-only-coding-agents]], [[summary/claude-chic]], [[summary/claude-code-beast-6-months]], [[summary/claude-code-cheat-sheet]], [[summary/claude-code-config]], [[summary/claude-code-on-the-go]], [[summary/claude-ctrl]], [[summary/claude-sidecar]], [[summary/claudemd]], [[summary/code-field]], [[summary/codex-iterates-agents-md]], [[summary/codex-maxxing]], [[summary/coding-agents-and-complexity-budgets]], [[summary/cognitive-debt]], [[summary/collaborator]], [[summary/compound-engineering]], [[summary/ctx-agentic-development-environment]], [[summary/cyborgs-will-kill-the-corporation]], [[summary/dark-flow]], [[summary/designing-agentic-loops]], [[summary/dont-fear-the-dark-factory]], [[summary/five-levels-from-spicy-autocomplete-to-the-dark-software-factory]], [[summary/gas-town-after-10000-hours-claude-code]], [[summary/gravity-of-cross-platform-apps]], [[summary/happy]], [[summary/hn-dont-fall-into-anti-ai-hype]], [[summary/hn-opus-4-5-coding-agent-experience]], [[summary/hn-pre-commit-lint-checks-discussion]], [[summary/hn-rip-low-code-2014-2025]], [[summary/how-boris-uses-claude-code]], [[summary/how-claude-code-works-in-large-codebases]], [[summary/how-to-effectively-write-quality-code-with-ai]], [[summary/how-to-write-a-good-spec-for-agents]], [[summary/how-we-use-claude-code-today-at-intercom]], [[summary/i-dont-want-your-prs-anymore]], [[summary/if-ai-is-doing-the-investigation-version-the-investigation]], [[summary/inside-our-in-house-data-agent]], [[summary/inside-the-ai-workflows-of-every-s-six-engineers]], [[summary/intent-layer]], [[summary/jc-workflow]], [[summary/kata]], [[summary/life-system]], [[summary/lovelace]], [[summary/mcp-is-dead-long-live-mcp]], [[summary/minimax-skills]], [[summary/minions-stripe-one-shot-coding-agents]], [[summary/mobilevibe]], [[summary/my-experience-with-claude-code-20]], [[summary/on-a-year-of-multi-model-development]], [[summary/openspec]], [[summary/opus-4-5-change-everything]], [[summary/oversight-and-guidance]], [[summary/pencil-dev]], [[summary/pre-commit-lint-checks]], [[summary/probabilistic-engineering-and-the-24-7-employee]], [[summary/radical-accountability]], [[summary/recursive-mode]], [[summary/refactor-legacy-code-with-copilot]], [[summary/repomirror]], [[summary/road-runner-economy]], [[summary/simmer-skill]], [[summary/simon-willison-pragmatic-summit-engineering-practices]], [[summary/simplicity-in-the-age-of-ai-assisted]], [[summary/slowing-down-in-the-age-of-coding-agents]], [[summary/slowing-the-fuck-down]], [[summary/software-2.0-case-study-textforge]], [[summary/spec-driven-development]], [[summary/specsmaxxing]], [[summary/structured-prompt-driven-development]], [[summary/summarize-meetings-skill]], [[summary/talking-to-transformers]], [[summary/the-claude-c-compiler]], [[summary/the-claude-code-playbook]], [[summary/the-cult-of-vibe-coding-is-insane]], [[summary/the-lifecycle-of-a-swamp-issue]], [[summary/the-mythical-agent-month]], [[summary/the-next-two-years-of-software-engineering]], [[summary/the-plan-is-the-program]], [[summary/thereisnospoon]], [[summary/trading-ideas-claude-equity-research]], [[summary/tw-future-of-software-development-retreat-key-takeaways]], [[summary/two-kinds-of-user-are-emerging]], [[summary/vibes-cli]], [[summary/writing-a-good-claude-md]], [[summary/zero-alignment]]*
*Last compiled: 2026-08-10*
