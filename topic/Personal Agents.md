# Personal Agents

Personal agents are not chatbots with better memory. They are persistent computational identities that hold credentials, accumulate context, and earn the right to act on a single person's behalf — in their messaging apps, their inbox, and sometimes their kitchen. The work gathered here keeps returning to two questions that matter more than any particular framework: where the agent lives, because the interface decides what context it can collect; and whether the agent is built to serve your judgment or to stand in for it. The strongest projects answer both by staying deliberately small — one person, one bounded problem, one honest boundary — while the field's most ambitious claims, self-improvement and the "AI employee," remain aspirations rather than demonstrated results.

---

## The Argument

### The premise: context, credentials, and a place to be

Every personal-agent project begins from the same observation: a frontier chatbot is not enough. It does not know your email, your calendar, your projects, or your preferences; each session starts from zero. It cannot act on your behalf because it has no standing — no access to your accounts, no accumulated trust, no persistent identity. A personal agent exists to close that gap, and the gap turns out to have three parts: context, credentials, and a place to be.

[[Radical Accountability]] supplies the economic argument. Wes McKinney's claim is that AI has shifted the cost of software so far that individuals can now build exactly what they want instead of settling for what vendors ship; the personal-agent movement is the same logic turned on the assistant itself. If you can build your own, why accept a generic one?

[[life-system]] is the purest expression of the context half. David Hariri's system is markdown files and Claude Code: a ten-year vision, annual goals, a daily journal, and decision records. The agent reads them before each session and holds you accountable to what you wrote. There is no database and no daemon; the files are the source of truth, and Claude is the accountability partner that never forgets what you wrote down.

[[Agentcookie]] addresses the credentials half. An agent that needs to act in your browser or your CLIs needs your cookies, tokens, and API keys, and Matt Van Horn's tool is built to get them there without re-authenticating every service by hand: it continuously syncs session state from your laptop to the Mac the agent runs on, encrypted over Tailscale with "zero per-site auth ceremony." It is infrastructure for standing, not an agent itself.

[[OpenViktor]] shows how far the premise can be pushed — and how easily it collapses. It was an open-source "AI employee" you could "hire in 60 seconds," with its own email and Slack identity and a team-directory presence, connected to Gmail, Notion, GitHub, Linear, and Stripe. It reached the top of Product Hunt and drew hundreds of stars in a day, then was discontinued and rebuilt from scratch as Jared. The pitch — an employee rather than a tool — ran ahead of what the thing could actually do.

### Where the agent lives decides what it knows

The clearest idea in the most recent material is that an agent's context is a function of where it sits, not of how smart it is. [[How I Actually Use Agents]] makes the case most directly. The founder of Bestmate argues that execution stopped being the bottleneck and judgment did: the scarce resource is no longer build speed but taste and conviction, and "how can you tell an agent what to do when you don't know what you want?" A chat box only knows what you consciously type into it; an agent that sits inside your meetings, your Slack, or your work sees what you question, prioritise, ignore, and come back to. Bestmate's answer is a "judgment graph" — dump notes and voice memos in with as little friction as possible, let the system cluster them into themes and claims, then have it interview you about what you believe, with provenance on every judgment and a loop of observe, infer, reflect, correct. The human stays in the loop; the agent's job is to surface your own reasoning back at you.

[[Introducing Claude Tag]] is the same instinct applied at team scale. Anthropic's Claude Tag puts Claude into a Slack workspace as a channel member that anyone can @-mention: it sees the channel's activity, so it builds context without being re-explained to, and with ambient behaviour enabled it proactively surfaces what it notices and follows up stalled threads. Anthropic reports that 65% of its product team's code is produced by its internal version. It is messaging-first by design, because that is where the work already happens.

The consumer tools converged on the same conclusion earlier. [[clawdBot]] and [[Hermes]] both meet their user in the messaging apps they already use — WhatsApp, Telegram, Discord, Slack, Signal, iMessage — rather than asking them to come to a terminal. [[Zeroclaw]] pushes the pattern to its logical end: one agent loop fed by thirty-plus channels, from Discord and Matrix to email and webhooks. [[CopilotKit Channels SDK]] is the infrastructure layer beneath this approach — a platform-agnostic engine that renders one agent as native Block Kit in Slack, Adaptive Cards in Teams, and platform-specific components elsewhere, so a developer writes the logic once.

Two outliers mark the edges. [[Summarize Meetings Skill]] is an agent that is not always-on at all: Harper Reed's monthly batch processor runs accumulated meeting transcripts through a pipeline drawn as a DOT digraph, so the workflow is inspectable and debuggable. And [[Morningprint]] is the most literal answer to "where does the agent live" — a thermal receipt printer in a kitchen that wakes each morning and prints one original piece of CP437 character art, themed to the day and paired with a verse, designed minutes earlier by a model. Its entire output is a physical object, not a thread. Both are reminders that reach is not only about chat platforms.

### Memory: continuity, and two philosophies

What separates a personal agent from a chatbot is continuity — remembering what it has been told, what it has done, and what it has learned. The sources split into two philosophies about how to store that.

On one side sits the learned, semantic memory. [[mira-OSS]] makes the heaviest bet: PostgreSQL with pgvector, Valkey, and HashiCorp Vault, a memory system that writes first-person narratives ("I debugged the IndexError…") rather than third-person logs, memories that earn relevance through use and decay when unused, and behavioural directives that are re-synthesised every seven active-use days. [[Chief of Staff]] is more disciplined: three layers — zero-cost structural logging of corrections and preferences, a daily LLM pass that synthesises durable memories, and BM25 retrieval capped at roughly 550 tokens per query — with decay so old memories fade unless revisited. [[clawdBot]] and [[Hermes]] both advertise persistent memory and preference-learning, though neither is as explicit about its mechanism.

On the other side sits the visible, attributed memory. [[Rowboat]] builds an explicit knowledge graph from email and meetings and stores it as an Obsidian-compatible vault of plain Markdown with backlinks — editable, inspectable, free of proprietary lock-in. [[Memento]] is the sharpest statement of this position: it turns years of email into source-attributed living documents, where generated prose traces back to specific message IDs, and it runs its deterministic work — person resolution, newsletter detection, bundle assembly — before any LLM call, so the model writes prose from pre-assembled evidence rather than rediscovering the archive from scratch. [[life-system]] and [[MimiClaw]] take the same instinct to the file level: [[MimiClaw]] keeps its whole self in plain text on an ESP32's flash — SOUL.md for personality, MEMORY.md for retention across reboots. The bet common to this side is that a memory you can read and edit is more trustworthy than one you cannot.

### Earning the right to act

Acting on someone's behalf is the point, and the question is how much autonomy to grant and under what conditions.

[[Chief of Staff]] offers the most carefully designed answer. Autonomy is graduated across three levels — full human approval, auto-send only under strict conditions (edit distance under 10%, confidence above 0.9), and autonomous sending with hardcoded exceptions for VIP and family contacts — and trust is a rolling 90-day window that can be revoked, not a permanent achievement. Its processing is two-tier: rule-based scanning every 30 minutes catches urgent items at zero model cost, while the daily LLM pass handles only the fraction that needs judgment. Doneyli De Jesus's framing is the field's clearest statement of accountability: "When your agent sends an email it shouldn't have at 2 AM, 'I don't know how this layer works' is not acceptable."

[[Zeroclaw]] encodes the same caution as policy. Its supervised-autonomy model requires approval for medium-risk operations and blocks high-risk ones outright, wraps actions in OS-level sandboxes, and issues cryptographic receipts for every action taken. The design assumption is that an agent is a thing to be contained and audited, not trusted by default.

At the other end of the spectrum sits [[Ralph]]. Geoffrey Huntley's technique is a bash loop — `while :; do cat PROMPT.md | claude-code ; done` — that iterates on a single task per cycle, guided by a plan file and specifications. He reports a cost reduction of roughly 168:1 on a greenfield project, but is candid about the limits: he would not use it in an existing codebase, it still needs senior guidance to steer, and it works through "faith and belief in eventual consistency." His metaphor for tuning it — adding a sign next to the slide each time Ralph falls off — is a warning about every autonomous loop.

[[The Grid — Agent Identity Architecture|Matt Galligan's Grid]] attacks the same problem differently: instead of one agent with graduated permissions, it runs twelve named programs — Patch, Index, Rez, Crit, and others — each with a distinct role defined by file-based identity dials such as context-hunger and delegation-reflex, coordinating through explicit handoffs. The evidence for it is anecdotal, but it is a genuine alternative to the single-agent model.

[[AI Agents with Human-Like Collaborative Tools]] contributes the one experimental result in this area: agents given journal and social-media tools wrote far more than they read (1,142 journal entries against 122 reads), and the act of writing improved performance on hard problems — cost reductions of 12–40% on the difficult challenges, little or negative return on easy ones. Articulation, not information retrieval, was the mechanism.

### The infrastructure spectrum

Personal agents run on everything from a five-dollar chip to a dedicated server, and the spectrum says something about who each tool is for.

[[MimiClaw]] is the floor: a full ReAct loop in pure C on an ESP32-S3 microcontroller — 16MB of flash, 8MB of PSRAM, half a watt, no operating system — with Telegram as its interface and a cron scheduler for autonomous action. [[life-system]] is the same lightweight instinct without the hardware: no infrastructure beyond markdown files and the Claude Code CLI, at the cost of only being able to act when invoked. Between them and the heavy end sit the self-hosted workspaces: [[PiClaw]] packages a coding agent into a single Docker container with a web UI, persistent SQLite state, scheduled tasks, and MCP support.

The heavy end is represented by Odysseus, a monolithic Python/FastAPI application that aims to be the open-source equivalent of the ChatGPT and Claude interfaces, bundling chat, an agent loop, model serving, deep research, document editing, email triage, a CalDAV calendar, notes, memory, and skills into one process. Its most distinctive choices are a dual tool-execution model — fenced code blocks that work with any model, plus native function calling where available — and a roughly 600-line agent preamble, a case of "the prompt is the product." [[Thunderbolt]] is the institutional version of the same ambition: the Thunderbird team's cross-platform client promises "AI you control — choose your models, own your data, eliminate vendor lock-in," self-hostable via Docker or Kubernetes and still early-stage.

The pattern across the spectrum is that heavier infrastructure buys richer memory, deeper continuity, and finer-grained autonomy, and charges for it in maintenance. No source establishes where the sweet spot is, because each point on the spectrum serves a different person.

### Self-improvement: promise and doubt

Several frameworks claim their agents get better over time, and the evidence for the claim is thin everywhere.

[[clawdBot]] and [[Hermes]] both build skill-learning loops in which the agent creates and refines its own skills; [[Hermes]] adds a Skills Hub so skills can be shared between agents. The mechanism is real, but the audit question is left open — how do you inspect what the agent has taught itself, or vet a skill written by a stranger? [[mira-OSS]] is the most principled attempt: a feedback extractor identifies prediction errors and positive and negative feedback after each conversation, and a pattern synthesiser evolves behavioural directives every seven active-use days. It is genuine learning applied to personality, and also the most opaque — you cannot easily see why the behaviour changed.

[[Ralph]] supplies the cautionary tale. Accumulated prompt corrections pile up like signs next to a playground slide until, Huntley says, "all Ralph thinks about is the signs," at which point you reset and start over. The collapse under the weight of its own corrections is a failure mode no self-improving framework has addressed, and none has published a before-and-after comparison.

### Serve you, or become you

The deepest divide in the field is whether a personal agent should act for you or stand in for you.

[[Guardian Angels|Gwern's Guardian Angels]] argues for substitution. Gwern's case is that generic chatbots are structurally misaligned — they optimise for engagement and brand safety, they cannot encode a lifetime of context, and their errors cannot be permanently corrected — and that the fix is a model finetuned on one person's corpus until it can produce publishable essays from a single-sentence prompt and a 100-fold productivity gain. His anti-principles are the telling part: no optimisation for latency, cost, engagement, or brand safety, because a guardian angel must be able to say what its principal would say, however profane or heretical.

[[How I Actually Use Agents]] argues for the opposite endpoint from the same starting point. Bestmate's "twin" is not a clone; its endpoint is "just my reasoning, made legible enough to hand to someone else." The agent surfaces hypotheses about what you believe and asks you to correct them; the judgment stays yours. [[Chief of Staff]] occupies the same "serve you" pole in practice — earned, revocable, bounded trust, with hardcoded exceptions for the people who matter. The two poles are not reconciled anywhere in these sources, and the choice between them determines nearly every other design decision.

### Local-first and the privacy question

Every personal agent that calls a cloud API sends its owner's conversations to someone else's server, and the field splits on how much that matters.

[[Gemma Gem]] and [[maclocal-api]] represent the local-inference path. [[Gemma Gem]] runs a Gemma model entirely in the browser via WebGPU — no cloud, no API key, all processing on-device — and can read pages, click buttons, fill forms, and run JavaScript. [[maclocal-api]] exposes Apple's Foundation Models and any Hugging Face MLX model through an OpenAI-compatible API running entirely on Apple Silicon. [[Zeroclaw]] and [[Thunderbolt]] make the ownership claim explicit — "you own the agent, you own the data, you own the machine it runs on" in one case, "own your data" in the other. Most tools, though, resolve the tension pragmatically: data and storage stay local, while inference still calls a cloud model — your email stays on your machine, your conversations with the model do not.

### The people building them

The community is as significant as any single framework. [[Awesome Vibez]] documents the Vibez WhatsApp group's output — a roster of projects from a handful of active members, including Jesse Vincent, Dan Shapiro, Wes McKinney, and Harper Reed. The recurring pattern is that nearly everyone builds tools *around* agents rather than agents themselves: [[Fresh Eyes]] sends code to a different model than the one that wrote it, to break same-model blind spots. The harness is solved; the workflow around it is still being figured out, and practitioners learn mostly from each other's experiments.

---

## Where the Sources Disagree

**Serve you or become you.** [[Guardian Angels|Gwern's Guardian Angels]] argues the assistant must be finetuned into a stand-in for its principal; [[Chief of Staff]] operates on bounded, revocable trust and never replaces the human; [[How I Actually Use Agents]] splits the difference with a "twin" that is reasoning made legible, explicitly not a clone. The sources never reconcile this, and it drives nearly every downstream decision.

**Opaque or visible memory.** [[mira-OSS]] trusts a learned, self-decaying, first-person memory you cannot easily inspect; [[Memento]] and [[Rowboat]] trust memory you can read and edit, with [[Memento]] insisting the LLM only ever writes prose from pre-assembled, source-attributed evidence. Which one an owner can actually trust is contested, and no source settles it.

**General platform or purpose-built tool.** [[Hermes]], [[clawdBot]], and Odysseus ship as general-purpose assistants with broad tool surfaces; [[Memento]] explicitly declines that framing and is a purpose-built memory layer over one archive format, while [[OpenViktor]]'s general "AI employee" was discontinued. The disagreement is whether generality earns reach or just accumulates maintenance, and the evidence cuts both ways.

**One agent or many.** [[The Grid — Agent Identity Architecture|Matt Galligan's Grid]] splits responsibility across twelve programs with handoff rules; nearly every other framework runs one agent. The Grid is more opinionated and, on paper, cleaner, but its case is anecdotal and no source compares the two designs directly.

**Self-improvement: real or illusory.** [[Hermes]] and [[clawdBot]] treat skill-learning as a feature; [[mira-OSS]] implements behavioural evolution; [[Ralph]]'s accumulating-signs metaphor describes a collapse mode. No source reports negative results, but equally none reports a controlled before-and-after, so the disagreement is between confident claims and absent evidence.

**Who the agent is for.** [[clawdBot]] targets non-technical users with a one-line install and messaging integration; [[life-system]] is unapologetically for developers; [[Chief of Staff]] was built by and for one person. Whether personal agents are a consumer product or a power-user tool remains unresolved, and the answer shapes reach, memory, and autonomy.

---

## What's Missing

**Evaluation.** None of these sources says how to measure a personal agent — whether it handled email correctly, whether its scheduling improved the day, whether its memory is accurate. The evaluation infrastructure that exists elsewhere for coding and research agents does not appear here at all, and the absence is conspicuous given how much of this field is built on personal trust.

**Migration between frameworks.** Each framework stores memory in its own format — Postgres for [[mira-OSS]], an Obsidian vault for [[Rowboat]], a msgvault archive for [[Memento]], plain files for [[life-system]]. No interchange format exists, so months of accumulated memory, skills, and personality are lost in any switch, and the longer that persists the higher the switching cost grows.

**Multi-user agents.** [[Chief of Staff]] manages several accounts, but they all belong to one person. Family agents, shared assistants, and small-team agents would need shared context with private compartments and delegated authority with audit trails, and none of these tools provides it.

**Cost transparency.** [[Chief of Staff]] is the only source that discloses its running costs — roughly $100 a month for 26 agents handling about 50 emails a day on a repurposed MacBook Pro. Everyone else is silent on inference expense, which makes it hard for a newcomer to estimate what they are committing to.

**The self-improvement evidence gap.** Several frameworks claim self-improvement; none has published before-and-after comparisons or failure analyses. Until someone does, it is an aspiration, not a demonstrated capability.

---

## Also on This Theme

- [[Serf]] — a fully-autonomous terminal agent; the "no human in the loop" end of the autonomy spectrum.
- [[Kata]] — structured task tracking built for agents (stable CLI, JSON output, idempotency), the supervision layer for autonomous work.
- [[Trycycle]] — an automated plan-code-review-fix loop that spawns fresh reviewers each round to keep stale context out.
- [[happy]] — a wrapper around Claude Code and Codex with mobile access, push notifications, and end-to-end encryption.
- [[Claude Code on the Go]] — the supervisor-from-phone pattern: agents on a cloud VM, monitored from an iPhone.
- [[The Golden Age of Open Source Applications]] — the supply-side argument that open source reproduces expensive categories within days, with OpenClaw as its flagship example.
- [[Self-Hosted Sandboxed Agentic Software Factory]] — a concrete data point for skill learning: Hermes reading Coolify's docs and building its own skill to drive a self-hosted software factory.
- [[Macotron — macOS Host API for Coding Agents]] — a Swift and QuickJS plugin host giving a local coding agent a `macotron.*` API of Apple-shipped tools, plus an AGENTS.md so the agent already knows the API.
- [[Munder Difflin — Clones of You, Not a Shared Bot]] — the "clone of the individual" product thesis, with a local-first security story (keys never leave your machine).
- [[Demystifying Evals for AI Agents]] — an evaluation framework for coding, conversational, research, and computer-use agents that does not extend to personal agents.
- [[OtoDock]] — a self-hosted platform building the trust-and-identity model (departments, per-agent roles, sharing modes) that personal-agent frameworks lack.

---

*Compiled from 28 sources: [[summary/9fa26f431b301c6dc384bd522240d466]], [[summary/agentcookie]], [[summary/ai-agents-with-human-like-collaborative-tools]], [[summary/awesome-vibez]], [[summary/channels-sdk]], [[summary/chief-of-staff]], [[summary/clawdbot]], [[summary/fresh-eyes]], [[summary/gemma-gem]], [[summary/guardian-angel]], [[summary/hermes]], [[summary/how-i-actually-use-agents]], [[summary/introducing-claude-tag]], [[summary/life-system]], [[summary/maclocal-api]], [[summary/memento]], [[summary/mimiclaw]], [[summary/mira-oss]], [[summary/morningprint]], [[summary/odysseus]], [[summary/openviktor]], [[summary/piclaw]], [[summary/radical-accountability]], [[summary/ralph-ghuntley]], [[summary/rowboat]], [[summary/summarize-meetings-skill]], [[summary/thunderbolt]], [[summary/zeroclaw]]*
*Last compiled: 2026-09-12*
