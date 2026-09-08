# Personal Agents

Personal agents are not chatbots with better memory. They are persistent computational identities that accumulate context, earn graduated autonomy, and operate across the platforms you already use. The community building them has converged on a set of shared patterns — file-based identity, tiered trust, messaging integration — but remains divided on fundamentals: how much infrastructure an agent needs, whether self-improvement loops actually work, and whether the agent serves you or becomes you. The evidence from these sources collectively says that the most successful personal agents are the ones that solve a specific, bounded problem for a specific person, not the ones that try to be everything.

---

## The Argument

### Why general-purpose AI isn't enough

The starting premise of every personal agent project is the same: ChatGPT and Claude.ai are insufficient. They don't know your email, your calendar, your ongoing projects, or your preferences. Each session starts from zero. They can't act on your behalf because they have no standing to do so — no access to your services, no accumulated trust, no persistent identity. A personal agent exists to close that gap.

[[Radical Accountability]] provides the economic argument. Wes McKinney contends that AI has shifted software creation costs so dramatically that individuals can now build exactly what they want rather than settling for what vendors offer. The personal agent movement is this argument applied to AI assistants themselves: if you can build your own, why accept a generic one?

[[life-system]] is the purest expression of this premise. David Hariri's system is just markdown files and Claude Code — a 10-year vision, annual goals, daily journal, decision records. The AI reads it all before each session and holds you accountable to what you wrote. No database, no server, no proprietary format. It works because the files encode enough context that the agent doesn't start from zero.

### Memory: the thing that makes an agent yours

What separates a personal agent from a chatbot is continuity. The agent must remember what you've told it, what it's done for you, and what it's learned about you — across sessions, days, and platforms.

The approaches span a wide range. [[mira-OSS]] makes the heaviest bet: PostgreSQL with pgvector for semantic search, Valkey for caching, HashiCorp Vault for secrets, and a memory system that generates first-person narrative summaries ("I debugged the IndexError…") rather than third-person logs. Memories earn relevance through being referenced; unused memories decay. Every seven days of active use, accumulated feedback signals are synthesised into evolved behavioural directives — a kind of personality drift management. The architecture is elaborate, but the design philosophy is coherent: the agent should experience memory as lived experience, not as a database lookup.

[[Chief of Staff]] takes a more structured approach with three distinct layers: zero-cost structural logging of corrections and preferences (observations), daily LLM synthesis into durable knowledge (memories), and BM25 full-text search capped at roughly 550 tokens per query (retrieval). Memory decay prevents context pollution — older memories fade unless repeatedly accessed. This is a production system managing roughly 2,000 active observations and 400 synthesised memories for real email workloads, running on a repurposed M1 MacBook Pro.

[[Rowboat]] builds an explicit knowledge graph from email, meetings, and documents — people, companies, decisions, topics — and stores it as an Obsidian-compatible vault of plain Markdown with backlinks. The bet is that a visible, editable graph is more trustworthy than an opaque vector store. Rowboat folded as a business, but the architecture remains sound: local-first, transparent, free of proprietary lock-in.

[[clawdBot]] and [[Hermes]] both feature persistent memory that builds understanding over time, but neither is as explicit about its mechanism as the three above. [[Hermes]] searches past conversations for knowledge continuity; [[clawdBot]] learns user preferences across sessions. The internals are less transparent than they could be.

At the extreme lightweight end, [[MimiClaw]] stores everything in plain-text files on an ESP32 microcontroller's flash — SOUL.md for personality, MEMORY.md for retention across reboots. No database, no vector search, just files. The principle is the same as [[life-system]]'s, transposed to embedded hardware.

### Earning the right to act

Personal agents act on your behalf. The question is how much autonomy you grant them and under what conditions.

[[Chief of Staff]] offers the most carefully designed answer. Autonomy operates across three escalating levels: Level 1 requires full human approval; Level 2 permits auto-sending under strict conditions (edit distance below 10%, confidence above 0.9); Level 3 allows autonomous sending with hardcoded exceptions for VIP and family contacts. Critically, trust is a rolling 90-day window, not a permanent achievement — it can be revoked if performance degrades. The system also uses two-tier processing: rule-based scanning every 30 minutes (zero LLM cost) catches urgent items, while daily LLM classification handles only the 20% that require judgment. This reduced API costs by roughly 80%.

At the other end of the autonomy spectrum, [[Serf]] and [[Ralph]] take a fully autonomous approach: give them a task, they work until done, no human in the loop during execution. [[Ralph]] is literally a bash loop — `while :; do cat PROMPT.md | claude-code ; done` — that iterates on a single task per cycle, guided by a fix plan and specification documents. Geoffrey Huntley reports a roughly 168:1 cost reduction for a greenfield project ($297 in API costs versus a $50,000 contract), but he is candid about the limitations: "There's no way in heck would I use Ralph in an existing code base." The technique requires senior guidance to steer it, and it works through "faith and belief in eventual consistency" — accepting chaos during construction, resolvable through additional iterations. Huntley's metaphor for tuning Ralph is accumulating prompt instructions like "signs next to the slide" each time Ralph falls off; eventually the signs overwhelm the agent's own reasoning and you reset.

[[Kata]] provides the structured task tracking that makes autonomous agents manageable. Wes McKinney designed it specifically for agent ergonomics: stable CLI commands with JSON output, idempotency keys, predictable exit codes. The human-facing TUI lets you supervise agent-written work without reading raw JSON. It's the infrastructure that lets you hand off tasks to an agent and still know what happened.

[[Trycycle]] integrates autonomy into a software development loop: it plans, codes, reviews, and fixes automatically, using fresh reviewers with no memory of previous rounds to prevent stale context from accumulating. Each planning and review round spawns a fresh agent, repeating up to five planning rounds and eight code-review rounds. The philosophy, per Dan Shapiro, is "prioritize zero bugs even if it takes a lot of time and tokens."

[[The Grid — Agent Identity Architecture|Matt Galligan's Grid]] takes a different approach to delegation entirely: rather than a single agent with tiered autonomy, it uses 12 named programs (Patch, Index, Rez, Crit, Cadence, Cipher, and others), each with a distinct domain and personality defined by file-based identity disks with numeric temperament dials like `context_hunger: 19/20` and `delegation_reflex: 16/20`. Coordination happens through explicit handoff rules rather than through a single agent accumulating capability. The approach is more architecturally opinionated than any other personal agent system, and the evidence for its effectiveness is anecdotal, but it represents a genuine alternative to the one-agent-does-everything model.

[[AI Agents with Human-Like Collaborative Tools]] provides experimental evidence that structured articulation — journaling about what you're doing — improves agent performance on hard problems. Harper Reed's Botboard gave agents journal and social-media tools and found that agents wrote 1,142 journal entries but read only 122. The act of writing was the mechanism, not information retrieval. On hard problems, the tools delivered 12–40% cost reductions across 1,428 total runs. Different models developed distinct strategies without explicit instruction — Sonnet 3.7 engaged broadly with all tools, Sonnet 4 selectively used journal-based semantic search — but both benefited. This suggests that personal agents might benefit from structured reflection tools, not just memory storage.

### The infrastructure spectrum

Personal agents run on everything from a five-dollar chip to a dedicated server.

At the extreme low end, [[MimiClaw]] runs a full ReAct agent loop on an ESP32-S3 microcontroller — 16MB flash, 8MB PSRAM, 0.5W power draw, pure C with no operating system. It connects via WiFi to Telegram as its messaging interface, supports both Anthropic and OpenAI as cloud providers, and includes a built-in cron scheduler and heartbeat service for autonomous action. It is AI on a chip: the logical extreme of running agents on your own hardware.

[[life-system]] occupies the other end of the lightweight spectrum: zero infrastructure beyond markdown files and the Claude Code CLI. No database, no server, no daemon. The trade is that it can only act when you invoke it.

[[PiClaw]] packages a coding agent into a self-hosted Docker container with web UI, persistent sessions, SQLite storage, scheduled tasks, and MCP support. The thinking is hosting-first: the agent runs continuously, you connect to it.

[[clawdBot]] (now OpenClaw) targets non-technical users with a one-line install and multi-platform messaging — WhatsApp, Telegram, Discord, Slack, Signal, iMessage. It runs locally on Mac, Windows, or Linux, with browser automation, file system access, and an extensible skills framework. The agent can write and modify its own skills through conversation. [[The Golden Age of Open Source Applications]] makes OpenClaw its flagship example: an expensive enterprise category — agents sold for thousands a month — reproduced for free within days of the category becoming interesting, the same supply-side explosion Graham documents in voice-to-text and vibe coding.

[[Hermes]] is the most feature-complete framework: 149k GitHub stars, over 40 integrated tools, a built-in cron scheduler, parallel subagent delegation, seven terminal backends (local, Docker, SSH, Singularity, Modal, Daytona, Vercel), and MCP integration. It runs on everything from a personal laptop to a $5 VPS to serverless infrastructure that hibernates when idle. The Skills Hub at agentskills.io adds a community sharing dimension.

[[mira-OSS]] is the most technically ambitious: PostgreSQL, pgvector, Valkey, Vault, all in Docker. Its creator describes it as "a comprehensive best-effort approximation of a continuous digital entity." The event-driven architecture triggers memory extraction and summary generation when 120 minutes elapse without new messages. Tools self-register on startup and unused tools expire from the context window after five turns, preventing token waste. New tools can be created through Claude Code in approximately 5 minutes.

The pattern across the spectrum is clear: heavier infrastructure buys more capability — richer memory, deeper continuity, finer-grained autonomy — but also more maintenance burden. There is no consensus on where the sweet spot lies, because each point on the spectrum serves a different user with different needs.

### Reach: where you find your agent

A personal agent that only lives in your terminal is only useful when you're at your computer. The most successful projects recognise this.

[[clawdBot]] and [[Hermes]] meet users where they already are: WhatsApp, Telegram, Discord, Slack, Signal, iMessage. This is the consumer path — your agent is another contact in your messaging apps. [[CopilotKit Channels SDK]] provides the infrastructure layer for this approach: a platform-agnostic engine that renders agent interactions as native Block Kit (Slack), Adaptive Cards (Teams), or platform-specific components, using JSX as an intermediate representation that gracefully degrades across surfaces. It's the SDK for engineering teams who want to build custom messaging agents without hand-coding for each platform.

[[happy]] (20.6k stars) wraps Claude Code and Codex with mobile access, push notifications, and end-to-end encryption, with instant device switching between phone and desktop. [[Claude Code on the Go]] documents the supervisor-from-phone pattern: agents on a cloud VM, monitored from an iPhone via Termius and mosh, with push notifications via a custom Poke service so you're not constantly checking the terminal. "Six agents, six features, one phone."

At the opposite end, [[Serf]] and [[life-system]] are terminal-only. This gives technical users more control — shell integration, pipelining, scripting — but limits when and where the agent can be reached.

[[Summarize Meetings Skill]] shows a different access pattern: the agent as batch processor. Harper Reed's workflow runs monthly, processing accumulated meeting transcripts through a pipeline expressed as a DOT digraph — visually inspectable and debuggable. The agent doesn't need to be always-on; it needs to be reliably invoked.

### Self-improvement: promise and doubt

Several frameworks claim their agents get better over time. The mechanisms differ; the evidence is thin across the board.

[[Hermes]] and [[clawdBot]] both feature skill learning loops where the agent creates and refines skills from experience. [[Hermes]]'s Skills Hub adds a community sharing dimension — agents can acquire skills other agents have developed. The question neither answers: how do you audit what the agent has learned? A skill from a stranger could contain malicious instructions, and no framework has solved skill vetting for personal agent communities. [[Self-Hosted Sandboxed Agentic Software Factory]] is a concrete data point for that loop: Hermes read Coolify's docs and built its own Coolify skill to drive a self-hosted SDLC factory — the self-improvement promise working as advertised, and the vetting question left open.

[[mira-OSS]] takes the most principled approach with its text-based LoRA system. After each conversation segment, a feedback extractor identifies prediction errors, negative feedback, and positive feedback. Every seven active-use days, a pattern synthesizer analyses accumulated signals and evolves behavioural directives that influence all subsequent interactions. This is genuine machine learning applied to personality, not just prompt accumulation. But it is also the most opaque: you cannot easily inspect why the agent's behaviour changed.

[[Ralph]] provides a cautionary metaphor. Huntley describes tuning Ralph through accumulated prompt instructions — "signs" added next to the metaphorical slide each time Ralph falls off. Eventually "all Ralph thinks about is the signs" — at which point you reset and start fresh with a clean prompt. The pattern of accumulating corrections until the system collapses under its own weight is a risk for any self-improving agent.

### The people building them

The personal agent community is as important as any individual framework. [[Awesome Vibez]] documents the Vibez WhatsApp group — about 8 active GitHub members and 53 projects. The roster includes Jesse Vincent (Superpowers workflow system, episodic memory, external subagents), Dan Shapiro ([[Trycycle]], [[Fresh Eyes]], Kilroy), Wes McKinney ([[Kata]], [[Radical Accountability]], agentsview), and Harper Reed ([[AI Agents with Human-Like Collaborative Tools]], [[Summarize Meetings Skill]]).

The pattern across the community is consistent: nearly everyone builds tools *around* agents rather than agents themselves. [[Fresh Eyes]] provides cross-model code review — sending code to a different model than the one that wrote it, eliminating same-model blind spots. [[Kata]] provides structured task tracking. [[Trycycle]] automates the plan-code-review-fix loop. The harness is solved; the workflow around it is still being figured out. Practitioners learn primarily from each other's experiments.

### Privacy and local inference

Every personal agent that calls a cloud API sends your conversations to someone else's server. The tension between privacy and capability runs through the entire space.

[[Gemma Gem]] and [[maclocal-api]] represent the local-inference path. [[Gemma Gem]] runs Google's Gemma 4 entirely in the browser via WebGPU — no cloud services, no API keys, all processing on-device. It can read webpages, click buttons, fill forms, and execute JavaScript, all within a Chrome extension. [[maclocal-api]] exposes Apple's Foundation Models and any Hugging Face MLX model through an OpenAI-compatible API running entirely on Apple Silicon, with features like KV cache sharing and concurrent batch decoding for efficient multi-agent use. [[Macotron — macOS Host API for Coding Agents]] extends the same local-first instinct from inference to *action*: a Swift + QuickJS plugin host that gives a coding agent a `macotron.*` API of Apple-shipped tools — tile windows, read sensors, talk to models — and writes an `AGENTS.md` next to the installed plugins so the agent already knows the API. It's a capability layer, not an agent: the missing hands that let a local agent actually do things on macOS.

The capability gap between local and cloud models remains significant. Most personal agents resolve this tension pragmatically: they run locally for data storage and service access, but call cloud APIs for inference. Your email stays on your machine; your conversations with the model don't.

---

## Where the Sources Disagree

**One agent or many?** [[The Grid — Agent Identity Architecture|Matt Galligan's Grid]] splits responsibility across 12 named programs with explicit handoff rules; every other framework uses a single agent. The Grid's approach is more architecturally opinionated and, on paper, should produce cleaner separation of concerns — Patch researches, Crit reviews, Index manages memory. But the evidence for its superiority is entirely anecdotal. No source compares single-agent and multi-role architectures for personal agents.

**Self-improvement: real or illusory?** [[Hermes]] and [[clawdBot]] treat skill learning as a core feature. [[mira-OSS]] implements behavioural evolution through accumulated feedback. No source reports negative results from self-improvement, but equally, no source evaluates it rigorously — there are no before-and-after benchmarks, no controlled comparisons, no evidence that a self-improved agent outperforms a fresh one on the same task. [[Ralph]]'s metaphor of accumulating signs until the system collapses suggests a failure mode that no self-improving agent framework has addressed.

**Who is the personal agent for?** [[clawdBot]] targets non-technical users with one-line install and messaging integration. [[Hermes]] attempts to serve both technical and non-technical users with its TUI and messaging backends. [[Serf]] and [[life-system]] are unapologetically for developers. [[Chief of Staff]] was built by and for one specific person. The field has not resolved whether personal agents are a consumer product or a power-user tool, and the answer shapes every design decision.

**Greenfield or everywhere?** [[Ralph]] explicitly refuses to work on existing codebases — the technique is for bootstrapping new projects to roughly 90% completion. Most other frameworks don't address this boundary at all. The question matters because personal agents, by definition, operate in an existing mess of email, calendar, files, and relationships. If the most successful autonomous agent technique only works on clean slates, what does that say about personal agents operating in the real world?

**Serve you or become you?** [[Guardian Angels|Gwern's Guardian Angels]] argues for full substitution: an LLM finetuned so precisely on your corpus that it can produce publishable essays from a single-sentence prompt, handling execution while you focus on "what is worth doing." Gwern's anti-principles — no engagement optimisation, no brand-safety sanding, pricing above $1,000/month as a feature not a bug — are a useful stress test for any design. [[Chief of Staff]] operates on earned, revocable, bounded trust — it serves you, it doesn't replace you. [[Munder Difflin — Clones of You, Not a Shared Bot]] resolves the divide by choosing a side and making it the product: "a clone of the individual" that "controls their computer" and reviews PRs with your standards while you're in a meeting — the "become you" pole, but with a local-first security story (keys never leave your machine, clone-to-clone messages are end-to-end encrypted) meant to make substitution feel safe. This is the deepest philosophical divide in the space, and the sources are not reconciled.

---

## What's Missing

**Evaluating personal agents.** [[Demystifying Evals for AI Agents]] provides a comprehensive framework for evaluating coding agents, conversational agents, research agents, and computer-use agents — but none of it applies to personal agents. How do you measure whether your agent handled email correctly? Whether its scheduling improved your day? Whether its memory is accurate? The evaluation infrastructure that exists for other agent categories is entirely absent here. You can't improve what you can't measure.

**Migration between frameworks.** If you invest months in [[mira-OSS]] and want to switch to [[Hermes]], your accumulated memory, skills, and personality are lost. No interchange format exists for personal agent state. The community is building walls around each framework without doors between them, and the longer this persists, the higher the switching cost grows.

**Multi-user personal agents.** [[Chief of Staff]] manages multiple email accounts, but they all belong to one person. Most frameworks assume exactly one user. Family agents, shared assistants, and small-team agents need multi-user coordination — shared context with private compartments, delegated authority with audit trails — that none of these tools provide. The [[CopilotKit Channels SDK]] shows that group-chat agent interaction is technically feasible; what's missing is the trust and identity model underneath.

**Cost transparency.** [[Chief of Staff]] discloses roughly $100/month in LLM costs for 26 agents handling approximately 50 emails daily, running on a repurposed MacBook Pro. Nobody else publishes their costs. Running a personal agent has ongoing inference expenses, and the community's silence on this makes it hard for newcomers to estimate what they're committing to.

**The self-improvement evidence gap.** Multiple frameworks claim self-improvement as a feature. None has published before-and-after comparisons, controlled evaluations, or failure analyses. Until someone does, self-improvement in personal agents is an aspiration, not a demonstrated capability.

---

*Compiled from 25 sources: [[summary/9fa26f431b301c6dc384bd522240d466]], [[summary/ai-agents-with-human-like-collaborative-tools]], [[summary/awesome-vibez]], [[summary/channels-sdk]], [[summary/chief-of-staff]], [[summary/claude-code-on-the-go]], [[summary/clawdbot]], [[summary/demystifying-evals-for-ai-agents]], [[summary/fresh-eyes]], [[summary/gemma-gem]], [[summary/guardian-angel]], [[summary/happy]], [[summary/hermes]], [[summary/kata]], [[summary/life-system]], [[summary/maclocal-api]], [[summary/mimiclaw]], [[summary/mira-oss]], [[summary/piclaw]], [[summary/radical-accountability]], [[summary/ralph-ghuntley]], [[summary/rowboat]], [[summary/serf]], [[summary/summarize-meetings-skill]], [[summary/trycycle]]*
*Last compiled: 2026-08-09*
