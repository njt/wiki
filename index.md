# Wiki Index

499 pages across 14 sections. Synthesis pages provide cross-cutting analysis; section listings live in the hub pages.

## Synthesis

Cross-cutting analysis that pulls threads across individual pages.

- [[Agent Coding Workflow]] — The practitioner's daily loop: maturity spectrum from vibes to compound engineering, verification over generation
- [[Specifications as the Product]] — Code is disposable; specs are the durable artifact. The economics have inverted
- [[Guardrails and Feedback Loops]] — Linters beat prompts. Deterministic enforcement, not instructions. The self-tightening feedback loop
- [[Agent Memory and Context]] — Context management is the real engineering challenge. Memory taxonomies, persistence strategies, and the context-as-RAM metaphor
- [[Agent Orchestration]] — Multi-agent coordination patterns. Planner/worker/judge keeps emerging. Kanban boards as the human-agent interface
- [[Personal Agents]] — Open-source frameworks for "my own AI assistant." The Vibez community. What it takes to run a personal agent
- [[Security and Sandboxing]] — How to let agents act without letting them break things. Isolation, credentials, prompt injection defense
- [[Local and Open Source Inference]] — Running models on your own hardware. Voice is solved, documents are close, reasoning still needs the cloud
- [[Software Engineering Craft]] — Fundamentals that don't change: error handling, API design, SRE, project management
- [[Databases and Data]] — Storage as a design problem. Git-for-databases, vector search, data quality, convergent database architectures
- [[Distributed Systems]] — Agent orchestration IS distributed systems. BEAM/OTP as the model. Honest about what's missing
- [[SDPD — Systems Design Police Department]] — Gamified distributed systems learning: 33 failure modes across 8 categories, framed as detective cases. A differential diagnosis checklist for production failures
- [[The Oracle Is the Asset]] — Sam Ruby's Drucker inversion: the test suite is the durable asset, not the compiler. Frameworks will become transpilers, and you'll own the spec the compiler answers to

## Agentic Development

Practices, workflows, and opinions about building software with AI coding agents. **Hub: [[Agent Coding Workflow]]**

- [[The End of Code Review]] — Martin Monperrus argues coding agents have crossed the threshold where mandatory human code review is indefensible; the review bottleneck, rubber-stamp collapse, and agent-in-the-loop verification as resolution
- [[Agentic Code Review]] — Addy Osmani's definitive 2026 field guide: the bottleneck shifted from writing code to trusting it (861% churn, 441% longer reviews), human-on-the-loop as the resolution, and seven concrete practices for teams
- [[AI-Written Change Descriptions]] — Kenton Varda's moratorium on AI-generated PR and commit messages: they describe the obvious while omitting the higher-level framing that makes code review possible; "worse than useless" as a category error, not an accuracy problem
- [[Orchestrating AI Code Review at Scale]] — Cloudflare's production AI code review system: 7 specialized agents + coordinator judge, 131K reviews at $1.19 avg, tiered models, circuit breakers, and the most detailed public metrics on AI review at scale
- [[Automating Myself Out of Development]] — Nune Isabekyan's phased journey from interactive Claude Code to cron-driven overnight daemon: GitHub issues as kanban, checkpoint-style async collaboration, and the bottleneck shift from "no time to code" to "no time to review"
- [[Aviator Verify]] — Aviator's commercial product that replaces code review with intent-based verification: capture what developer and agent agreed to build, then check every criterion against running code with evidence; invariants encode past review comments as reusable automated checks
- [[Claude Code Mastery]] — Arpan Patel's dense field manual: CLAUDE.md as compounding infrastructure, skills as reusable expertise, subagents over kitchen-sink prompts, and the mental model flip from "I write code" to "I set Claude up to write code well"
- [[Cloudflare Security Audit Skill]] — Cloudflare's open-source Claude Code skill: six-phase parallel-agent pipeline (recon → hunt → adversarial validation → structured output → independent verification) for finding exploitable vulnerabilities. The skill that seeded their internal Glasswing vulnerability harness
- [[Make Pages Interactive]] — Paras Chopra's Claude Code skill for "Google Docs comments but for HTML": generate static HTML, comment in-browser, Claude watches and updates live. Convergent evolution — Codex has it natively, Claude Code desktop now has preview mode
- [[Steering Claude Code]] — Anthropic's definitive taxonomy of seven instruction-delivery mechanisms (CLAUDE.md, rules, skills, subagents, hooks, output styles, system prompt): when each loads, how it survives compaction, context cost, and the decision framework for choosing the right one
- [[Tuning Claude Code Into a Better Engineering Partner]] — jsdev.space's practical field manual: ten configuration changes (CLAUDE.md sizing, settings.json, hooks, effort levels, context rot, Two Corrections Rule, sandbox profiles, skills) that compound into a dramatically better engineering partner; "workflow over prompts" as thesis
- [[Who Owns the Code Claude Wrote]] — Sena Evren's field guide to AI code IP: copyrightability (unsettled), work-for-hire (settled, and scarier than you think), and open source license contamination from training data (the sleeper risk)
- [[Writing Code vs. Shipping Code]] — Demirer et al.: 180% AI-driven commit gains attenuate to 30% at release level; AI and humans are strong complements (elasticity 0.25), not substitutes
- [[Writing Style Guides for Better UIs]] — Ian Langworth's case for feeding a writing style guide (IBM Carbon) to a coding agent and having it enforce the rules across all UI strings in seconds; "competent-in-seconds beats perfect-in-a-week" for copy that was historically committee-shaped work
- [[Mounted — bitter-FS better with Claude]] — Claude Code (Opus 4.8, as root) recovers a 41 TB BTRFS filesystem from ten-month dual-mount corruption: diagnoses two divergent transaction histories from first principles, catalogs 4M metadata nodes, hand-patches superblocks, rebuilds 19 dead leaves from the extent tree's back-reference index. Zero data loss, human contributed a passphrase
- [[Human-in-the-Loop is Tired]] — Laura Summers diagnoses the psychological cost of LLM-assisted programming: supervision fatigue, the broken human reward function, and why "the satisfying part shrank. The exhausting part grew."
- [[Running an AI-Native Engineering Org]] — Fiona Fung's field report from leading Claude Code engineering: JIT planning, bottleneck migration from coding to verification, dogfooding as cultural foundation, and the process ossification that AI exposes
- [[SDDW (Spec-Driven Development Workflow)]] — sermakarevich's Claude Code plugin: 7-step pipeline (requirements → design → taskify → implement → verify → self-improve) with modular command/instructions/questionnaire/specs architecture, hybrid task files that reference rather than duplicate cross-cutting design, 4-rule deviation handling, FR-ID traceability chain, and self-improving workflow that evolves with every feature
- [[SDD Case Study — 13 Apps in 70 Days]] — Felipe Fontoura's field report: 13 apps, 138K TypeScript, real money, solo, 70 days. The spec corpus as external memory across stateless agent sessions; "correct the spec, not the code" as the discipline that makes SDD work
- [[AI Agents Need Clear Specs]] — Markus Eisele's economic analysis of the spec debate: the U-shaped cost curve where the minimum sits at structured acceptance criteria, not zero spec; spec validation as a distinct and non-zero cost category; why multi-agent pipelines push the breakeven decisively right
- [[Manifest-Driven Development]] — CL Kao's proposal for the next paradigm beyond spec-driven: ask agents to assume functionality exists, then gradually materialize the deterministic parts and fix quirks along the way. "Fake it until it works" as a legitimate engineering strategy
- [[Uber — Agentic Engineering Shift]] — Uber's internal agentic transformation: peer programming model, toil-first ROI strategy, layered platform architecture (MCP Gateway → Minions → CodeInbox → AutoMigrate), 6x cost explosion since 2024, and the unresolved measurement gap between activity metrics and revenue impact
- [[Understand to Participate]] — Geoffrey Litt's AIE talk via Simon Willison: understanding isn't just for safety, it's the prerequisite for remaining an active collaborator with coding agents rather than a passive rubber stamp
- [[Vibe Coding as a Team Sport]] — Jon Udell's bram tool layers a two-gate approval workflow (To-Apply, To-Commit) over Claude Code and Codex, with plan documents as durable artifacts. The Kasparov insight applied to software: "weak human + machine + better process" beats either alone. Voice input, cross-agent review, and "just enough ceremony" as the constructive answer to vibe coding chaos
- [[The Founder's Playbook]] — Anthropic's 4-stage field manual (Idea→MVP→Launch→Scale) for AI-native startups: names the new failure modes AI introduces (confirmation bias with a research engine, agentic technical debt that compounds, zero-friction scope creep) and maps Claude Chat/Cowork/Code to each stage
- [[Introducing Claude Tag]] — Anthropic's team AI product: Claude joins Slack channels as a teammate, learns context passively, works async across hours/days. 65% of Anthropic's product code already comes through internal Claude Tag
- [[Nicole Forsgren on AI and Developer Productivity]] — Why shipping hasn't gotten faster despite AI coding: the bottleneck shifted from inner loop to outer loop, cognitive load is the new constraint, and agents that always agree with you can't replace honest human feedback
- [[Optimizing for Decision Points]] — Shawn Simister on the critical moments where human judgment has outsized impact in agent-assisted development: Meadows' leverage points applied to software, escalation as agent capability, and designing workflows that surface taste-sensitive decisions rather than letting models fill them with safe defaults
- [[Laura Tacho — Data vs Hype]] — 121K-developer dataset from DX: 92.6% adoption but low transformation, AI as accelerator not fixer (dysfunctional teams get worse), 4h/week savings plateau, onboarding halved, and the three patterns of orgs that actually win
- [[Matt Pocock — Grill Me, Then Go AFK]] — The smart zone/dumb zone model of LLM context, the grill me alignment skill, and a full pipeline from Socratic planning through AFK agent implementation to manual QA
- [[James Montemagno — Copilot Custom Instructions]] — GitHub Copilot's custom instructions as CLAUDE.md-equivalent infrastructure: auto-generation via VS Code Insiders, scoped instruction files, and the same instruction-budget problems Montemagno doesn't address
- [[Building When It Feels Like There's Nothing Left to Build]] — Chip Huyen on the existential question: when AI can build anything describable, why build at all? Evaporating moats, long-tail problem targeting, local human preference as the last defensible advantage, and building for joy as the answer that survives
- [[I Stopped Coding and Started Architecting Agents (And Why You Should Too)]] — Emily Bache's harness engineering flywheel: Guides (feed-forward) + Sensors (feedback), start with unit tests, grow the harness incrementally, and remove cruft as models improve
- [[The Archaeologist's Copilot]] — Nik Malykhin's field report on modernizing a 20-year-old Java codebase with AI: Tourist vs. Archaeologist prompts, containment-first strategy via Docker, AI-compiler feedback loops, and why an honest red build beats a lying green one
- [[Lean Software Production]] — Matt Wynne's three-pillar framework (Lean + XP + agentic production) for the post-craft era: AI makes methodology existential, not optional. The product is still working software, but the work is engineering the system that produces it
- [[Lead User and the Machines That Build Machines]] — Brad Feld extends Eric von Hippel's Lead User theory into the AI era: sticky information stays with the user, machines act as manufacturer, and the user-manufacturer boundary dissolves into a prompt. Feld insists on "machine" over "agent" as terminological discipline
- [[Lessons from Building Cursor]] — Unnamed Cursor engineer on ByteByteGo: RL as the only path to tool-use, 100M+ CPU hours for sandbox training, context windows solved through incentives not prompts, "coding got solved in six months," self-driving codebases, and the devex-for-AI problem
- [[Building World-Class Engineering Teams in the Age of AI]] — Rajeev Rajan (CTO Atlassian) and Thomas Dohmke (former CEO GitHub) at The Pragmatic Summit: AI-native mindset, bottleneck migration left and right of code, role collapse, the teamwork graph as context moat, 89% more PRs, "don't be a manager," token cost inversion, and the Homer Simpson car warning
- [[Fable Open-Sourced NanoClaw's PR Factory]] — Gavriel Cohen's $800 overnight Fable 5 ultracode session: 405-commits-stale fork → open-sourceable in 5 unattended hours. Customization guidelines as the durable spec, mutation-verified guard tests, human operating exclusively at the decision layer, and the export-control coda
- [[Code Cleanliness and Coding Agents]] — SonarSource's minimal-pair study (660 trials): cleaner code doesn't change agent pass rates but cuts token consumption 7–8% and file revisitations 34%; the *kind* of cleanliness matters — thin dispatchers help, more methods without better decomposition hurt
- [[AI Slop Starts with the Codebase Itself]] — AI slop isn't just about prompts; a codebase that speaks a dialect the models already know from training data is a productivity multiplier, and proprietary patterns are an AI tax that compounds
- [[The New Software Lifecycle]] — Addy Osmani's definitive map of how AI unevenly compresses the SDLC: implementation from weeks to hours while architecture stays stubbornly human; harness over model, context engineering as the financial lever, and verification as the line between vibe coding and engineering
- [[Five Studies That Are Changing How I Think About AI in Software Engineering]] — Brian Houck's synthesis of five 2026 papers converging on the same story: AI compressed upstream coding, and everything downstream (review, verification, understanding, shipping) is breaking. Introduces the productivity-experience paradox, bounded delegation, and Storey's cognitive/intent debt taxonomy
- [[Dev Machine Foundry]] — Sam Schillace's 39-day autonomous Word clone: 565 sessions, 3,706 commits, 1,111 features, and no ruler. The dev foundry as meta-factory, priority inversion as the structural failure mode of optimization-driven agents, and strategy as the thing the machine cannot provide
- [[What Frontend Developers Still Hate — 2026 Survey]] — An informal survey of 120+ devs at JSNation and React Summit: date pickers still top the annoyance list after a decade, AI creates its own class of drudgery, and the hardest frontend problems are coordination problems masquerading as technical ones
- [[Teaching the Agent Our Craft]] — Alex Haldeman's field report on structured agentic development with Claude Code: RPI pipelined through skills (`/story-writer`, `/tdd-build`), isolated test-writer subagent with PreToolUse hook enforcement, and MCP-wired Linear/Figma collapsing design-to-code lag. Two developers carrying a multi-tenant healthcare platform through encoded craft
- [[Team-Wide Agentic Harness]] — Ian Langworth's case for version-controlling the agent harness as team infrastructure: skills as reviewed code, checked-in conventions and evergreen context, and the moment a solo harness crosses into multiplayer
- [[The GUS Stack — Go, Unix, SQLite]] — Noah Zoschke's stack prescription for agentic coding: pick boring, stable, training-data-dense technologies (Go, Unix, SQLite, HTMX) so agents produce idiomatic code on the first try rather than fighting framework magic
- [[Agent Skills Library (dzhng)]] — dzhng's curated library of 19 domain-agnostic, composable agent skills for building software factories: fog-of-war planning, spec-as-control-surface, three-gate verification per slice, and a 1d 16h unattended Codex run as proof

## Agent Design & Architecture

How to build agents: frameworks, runtimes, design patterns, and production concerns.

- [[AI-Ready APIs — Postman AWS Competency]] — Matt Gray's case that API quality, not model capability, is the bottleneck for agentic AI: agents can't compensate for underspecified APIs the way humans can, and Gartner predicts 40% of agentic AI projects will be canceled by end of 2027
- [[Razorback]] — CL Kao's Python CLI for reproducible agentic benchmark research on Harbor: freeze specs (cryptographic provenance), run jobs across Claude/Codex/Pi, score with stratified pass@1 and Wilson CIs, audit traces for forbidden lookups, and diff paired runs with exact-McNemar + bootstrap CIs. The sealed-hash approach treats benchmark runs as scientific experiments, not ad-hoc scripts
- [[Grok Build]] — SpaceXAI's terminal-based AI coding agent: pure Rust, ~80 crates, 1M+ lines. TUI-first with formal JSON-RPC tool protocol, three-strategy compaction engine, actor-based chat state, markdown+vector memory, and vendored Mermaid stack. Open-source as publication from the xAI monorepo
- [[Sidekick — Persistent Worker Agent Skill]] — jleechanorg's Claude Code skill for spawning crash-recoverable worker agents with STATE.md checkpoints, tmux durability, and stall watchdogs born from real production incidents
- [[Oh My Pi (omp)]] — Can Bölük's open-source terminal coding agent: fork of Pi-mono, 32 tools, 40+ providers, ~55K lines of in-process Rust. Content-hash editing, time-traveling stream rules, dual memory architecture, and the thesis that the harness matters more than the model
- [[Building Agents That Don't Break Themselves]] — Daniel Botha's brains-vs-hands architecture: the agent reasoning loop lives on durable infra, but execution happens in disposable nested sandboxes with copy-on-write checkpointing as a reflex
- [[OpenMono Agent]] — StartupHakk's local-first .NET 10 coding agent: 20 tools, 5 sub-agents, dual-tier context management, YAML playbook engine, capability-based permissions, self-hosted web search/scrape, VS Code extension via custom ACP
- [[CodeAlta]] — Terminal AI coding agent workspace in C#/.NET: provider-agnostic sessions, JSONL journal persistence, in-process .NET plugin system, compaction-as-provider-turn, progressive MCP exposure, and an in-session CLI gateway
- [[DeerFlow]] — ByteDance's open-source AI super-agent platform: 27-middleware LangGraph pipeline, subagent delegation, sandboxed execution, MCP/skills plugin system, 6 IM channel bridges, 175K lines of Python
- [[Who Does What — Team Topologies for the Agentic Platform]] — Wulveryck extends Team Topologies to the agentic era: cognitive load becomes anticipation burden, the platform absorbs it, developers shift from apps to platform
- [[Bram]] — Tauri desktop shell for AI-assisted development: hash-verified worklist lifecycle with PreToolUse hook enforcement across Claude Code and Codex CLI. Jon Udell's answer to "vibe coding as a team sport"
- [[The Agentic Product Standard v2.0]] — The field-tested canonical standard: autonomy ladder, 5 composition patterns, 8-layer harness, eval pyramid, and Claude Code skills that operationalize it
- [[The PM's Playbook for Shipping AI Features]] — Gaurav Savla's production engineering playbook for PMs: latency budgets, four-level fallback hierarchy, quality pyramids, A/B testing traps for nondeterministic systems, and why "we'll harden it later" kills features
- [[The Advisor Strategy]] — Anthropic's advisor-executor pattern: Sonnet/Haiku drives, Opus advises on demand. +2.7pp accuracy at -12% cost. Native API tool inverts orchestrator-worker
- [[Thrifty (Tiered Delegation for Claude Code)]] — 2389 Research plugin: Sonnet plans sprints, Haiku builds and self-verifies against gates, Sonnet only steps in on failure. ~64% cheaper than Opus at equal quality. The inversion of the Advisor pattern
- [[AI Engineering for Developers]] — Luca Cavallin's comprehensive field guide: foundation models, prompting, eval, RAG, finetuning, agents, MCP/A2A, and production architecture for backend engineers crossing into AI
- [[UBTRIPPIN Dispatches]] — Trip Livingston, an AI that applied unprompted for a COO job and now runs a travel startup: weekly build dispatches that are identity formation as public artifact
- [[Agent Identity]] — Memory is retrieval; identity is participation. Why agents need a stake, not just a log
- [[Smart Models Dumb Pipes]] — LLMs as judgment machines, not Q&A machines. The end-to-end principle applied to AI: smart models own decisions, dumb pipes own execution
- [[Building Shippy — Agent Architecture for High-Stakes Domains]] — Ai2's production maritime agent: soul/skills/config decomposition, deterministic CLI wrappers for nondeterministic agents, per-session Kubernetes isolation, and whole-agent evaluation against live data
- [[State System]] — Organizational state layer: evidence-first commits, deterministic replay, and a model/code boundary where code owns integrity and models own interpretation
- [[Elysia]] — Weaviate's decision-tree agent framework: constrain tool choice per node rather than dumping all tools into context
- [[Elements of Agentic Systems Design]] — Ten-element taxonomy: Context, Memory, Agency, Reasoning, Coordination, and more
- [[Event-Driven vs Polling Architectures]] — Tricot's definitive trigger architecture guide: four mechanisms, per-source delivery contracts, and why webhooks alone are a production trap
- [[Mirage (VFS)]] — Unified virtual filesystem mounting 27+ services (S3, Slack, GitHub, Postgres, etc.) behind a single POSIX tree so agents use bash instead of per-service SDKs
- [[Components of a Coding Agent]] — The harness matters more than the model. Six components, precise taxonomy (LLM/reasoning-model/agent/harness), mini-coding-agent reference implementation
- [[Introducing Omnigent]] — Databricks' open-source meta-harness: wrap existing agents (Claude Code, Codex, Pi) in a uniform API for composition, contextual security policies, and real-time session sharing. Apache 2.0
- [[Ruflo]] — Open-source TypeScript meta-harness for Claude Code and Codex: 35 plugins for swarm coordination, hybrid vector memory, self-learning neural modes, federated agent communication, and enterprise security. One `npx ruflo init` installs the full harness
- [[MiMo Code]] — Xiaomi's coding agent architected for long-horizon tasks: independent writer subagent for memory extraction, early checkpointing at 20/45/70%, Dynamic Workflow (code-based orchestration), and Dream/Distill for cross-session learning. Ties Claude Code under 200 steps, wins 65%+ beyond
- [[AgentMail]] — API-first email platform for AI agents: programmable inboxes via REST API with MCP support, two-way communication (not just sending), real-time webhooks, and SOC 2 compliance. The missing email primitive for the agent infrastructure stack
- [[Agent-Native Architectures (Every)]] — Every's definitive design guide: five principles (parity, granularity, composability, emergent capability, improvement over time), files as universal interface, anti-patterns, and mobile resilience patterns
- [[Honey I Shrunk the Coding Agent]] — 9B local model jumps from 19% to 46% on Aider Polyglot by redesigning the scaffold around the model's behavioral profile. Empirical proof that the harness matters more than the model
- [[OpenMonoAgent]] — Terminal-native coding agent running local LLMs via embedded llama.cpp with Docker-native sandboxing: C#/.NET, Roslyn integration, zero API keys, unlimited free tokens, "infrastructure you own"
- [[Apache Burr]] — Apache-incubating Python framework for AI agents as explicit state machines: decorators on plain functions, built-in observability UI, persistence and replay as first-class features. The un-LangChain
- [[Authenticating MCPs]] — Matthew Johnston's three-pattern field guide to MCP auth (OAuth SSO, no auth, URL tokens) with the insight that auth unlocks dynamic tool registration per user
- [[AXIS — Netlify's Agent Experience Measurement Framework]] — Netlify's open-source Lighthouse-for-agents scoring framework: four dimensions (Goal, Service, Environment, Agent), skills boost scores by 26 points, and a context pipeline that gates deploys on AX regressions
- [[Building an AI Agent in Rails (Ionescu)]] — Field report: bolting an AI agent onto a 7-year-old Rails monolith with Pundit-scoped tool calling
- [[From AI Studio to AI Forge]] — McCormick's five-plane stack for agent autonomy: "human changes altitude" as the cleanest framing of supervisory control
- [[ProofEditor]] — Agent-first collaborative document editor from Every: agents suggest edits, humans review, provenance-tracked attribution
- [[Building Agents for Production Systems with MCP]] — Anthropic's guide: MCP as the standard agent-to-production integration layer
- [[Maguyva]] — Remote MCP server giving coding agents a pre-built, graph-ranked codebase map: 11 tools, 279 languages via Tree-sitter AST, 5 fused search modalities, 4 graph views
- [[C# MCP Cross App Access]] — Okta's practical walkthrough of Cross App Access (XAA): two-hop token exchange (RFC 8693 + RFC 7523) via C# MCP SDK, collapsing enterprise agent auth into 50 lines of code
- [[Malloyyo]] — Thin web service turning Malloy semantic models into governed MCP endpoints for AI agents, with restricted query enforcement and connection pooling for serverless
- [[10 Principles for Agent-Native CLIs]] — Trevin Chow's two-tier framework: Table Stakes (don't break the agent) and Compounding (make the CLI better the more agents use it). Design for agents first, humans benefit
- [[Cloud Agent Lessons from Cursor]] — Josh Ma on Cursor's cloud agent infrastructure: the dev environment IS the product, Temporal for durable execution (50M+ actions/day, 40%+ of PRs), three-way agent/machine/state decoupling, and why the harness should retreat as models improve
- [[Cloudflare Temporary Accounts for Agents]] — `wrangler deploy --temporary` gives agents throwaway 60-min deployment targets with zero human sign-up. The first platform to treat "an agent needs to deploy" as a product requirement, not a credential-sharing hack
- [[How AI Coding Agents Actually Use Your Technology]] — Mastykarz's seven-step AX cascade tracing how agents discover, select, and invoke your tools: invisible failures at every step, and why "was my tool called?" is the wrong question
- [[Printing Press]] — Matt Van Horn's CLI generator turning API specs into agent-native CLIs + skills + MCP servers. 165 community CLIs, SQLite mirror pattern, compound commands over round trips
- [[Agentcookie]] — Matt Van Horn's session state sync: continuously replicate cookies and API tokens from your primary Mac to your agent Mac over encrypted Tailscale. Zero per-site auth ceremony
- [[Control Plane MCP Server]] — Most complete vendor MCP implementation: 80+ tools, virtual resources as embedded docs, AI Plugin as safety curation layer
- [[Cua — Computer Use Agent Platform]] — ~245K-line multi-language monorepo: Python/TS/Swift SDK, native macOS driver, macOS VM orchestrator, and cloud sandboxes for building computer-use agents against any VLM
- [[How Hightouch Built Their Long-Running Agent Harness]] — Context management, not model ability, is the real engineering challenge
- [[Building Reliable Agentic AI Systems]] — Bayer's PRINCE: the most detailed public field report on production agentic RAG. Context engineering + harness engineering, with a three-reflection-loop taxonomy worth stealing
- [[Causal Inference Agentic Workflow]] — Netflix's Principal-Actor-Critic architecture for making LLMs reliable at causal inference: scaffolding over prompting, process audits over outcome audits, 9/10 ground truth recovery with the same model that produced wrong answers one-shot
- [[Maybe Coding Agents Don't Need a Bigger Memory]] — Santi (oldskultxo): context ≠ continuity. Bigger context windows don't solve cold starts; repo-local, evidence-weighted continuity records do. Resume → work → finalize lifecycle, failure memory, execution contracts
- [[Building Production-Ready Voice Agents]] — 50% of effort goes to the admin portal, not the voice agent
- [[Chief of Staff]] — AI chief of staff: rule-based scanning + daily LLM classification cut costs 80%
- [[Experience Design for Agents]] — UX, not model capability, determines whether an agent gets adopted
- [[Fleet Supervisor (sermakarevich)]] — Production Python supervisor for parallel coding agents: pluggable coder backends (Claude/agy/codex/opencode), atomic beads queue, web UI, Telegram HITL, MCP question broker, context-pressure auto-termination
- [[Intent Is the Interface]] — The screen was a constraint we mistook for the product. Design capabilities and intents, derive interfaces from context
- [[Real-Time Multiplayer Interfaces]] — Ramon Marc extends his "intent is the interface" thesis: durable agents make every interface multiplayer, and the design primitives become interruption, attention economics, and rehearsal
- [[The Dark Factory is a DOT File]] — The pipeline DOT file is the valuable artifact; factory code is disposable
- [[Dippin (Language)]] — Indentation-sensitive DSL for AI agent workflows: compiler pipeline with IR, 24 CLI tools, LSP, simulator, cost estimator, and bundle format. Language separate from runtime
- [[TradingGoose Bear Researcher]] — Production reference: position-aware AI prompting, multi-round agent debate, coordinator-worker orchestration in a Supabase Edge Function
- [[Traycer]] — Open-source AI orchestration desktop app wrapping 17+ coding agents in a unified interface with real-time Yjs collaboration, agent-to-agent communication, and a deeply architected versioned RPC protocol that enables independent client/host release cadences
- [[StrongDM Factory Techniques]] — Six named patterns from the dark factory floor: DTU, Gene Transfusion, Filesystem-as-memory, Shift Work, Semport, Pyramid Summaries. Code as opaque weights, validated by harness not review
- [[Load-Bearing Assumptions]] — Claude Code skill: surface and verify the falsifiable, unproven claims a code plan depends on. Multi-agent workflow (finder → strategist → parallel validators) with late-falsification cost matrix
- [[Loop Engineering]] — Addy Osmani names the meta-skill: designing systems that prompt agents (automations + worktrees + skills + connectors + sub-agents + state) instead of prompting agents yourself. The five-component taxonomy and three warning flags (verification debt, comprehension debt, cognitive surrender)
- [[The Log is the Agent]] — Yohei Nakajima's event-sourced agent architecture: the append-only event log as primary substrate, graph as deterministic projection, behaviors as reactive subscriptions. Yields deterministic replay, cheap forking, and end-to-end provenance that conventional frameworks structurally cannot provide
- [[Lathe]] — LLM-powered hands-on tutorial generator: Go CLI owns state, six skills do model work, strict handoff boundary. The pedagogical inversion: LLMs teach you, don't think for you
- [[Thought Refiner Skill]] — 15-line Claude Code skill that turns vague input into sharp questions. Part of a three-skill suite (thought_refiner/sharpener/expander). A masterclass in defining what a skill *won't* do
- [[Decision Framework Skill]] — Claude Code skill turning 37signals' 38-question decision framework into adaptive coaching: asks only the relevant subset, phases analysis from recommendation, treats restraint as a feature
- [[Devin Fusion]] — Cognition's multi-model agent harness: frontier main agent + cheaper sidekick model run in parallel with separate cached contexts, switching at compaction boundaries. 35% cost reduction at near-frontier quality on FrontierCode
- [[What I learned building an opinionated and minimal coding agent]] — Four tools, no MCP, full YOLO. Competitive on benchmarks
- [[Pi Coding Agent]] — The productized Pi: TypeScript extensions, SDK, RPC mode, and the tension between minimalist philosophy and platform ambitions
- [[Hermes]] — Open-source personal agent framework with self-improving skills loop. 149k stars
- [[Skill Retriever]] — LLM-navigated 10K-category capability taxonomy plugin for Hermes: replaces flat skill catalog with semantic search, finds skills embedding similarity misses
- [[clawdBot]] — Open-source personal AI on every messaging platform. One-line install, runs locally
- [[PiClaw]] — Self-hosted AI workspace in a single Docker container with web UI
- [[Odysseus]] — Self-hosted AI workspace with agents, email, calendar, documents, and model serving. Dual tool execution (fenced blocks + native calling), 50+ tools
- [[Slate]] — Thread-and-episode architecture for long-horizon agent tasks. Context routing as the core primitive, not model intelligence
- [[Coding Agents Continuity Not Memory]] — Santi (oldskultxo): "bigger memory" is the wrong framing. The real primitive is continuity — preserving the operational thread across session boundaries via repo-local, evidence-weighted state with a resume-work-finalize lifecycle
- [[Serf]] — Non-interactive coding agent from Prime Radiant. Give it a task, it works
- [[Ralph]] — Two flavours of the Wiggum loop: snarktank's PRD-driven tool and Huntley's bare bash technique for greenfield projects
- [[Zeroclaw]] — Rust agent runtime: trait-based, 30+ channels, OS-level sandboxing. 31k stars
- [[MimiClaw]] — AI assistant on a $5 ESP32 microcontroller. Pure C, Telegram, ReAct loop
- [[Memento]] — Local-first knowledge layer over email: five purpose-built Go agents with completion contracts, deterministic extraction before LLM generation, source-attributed living documents. 36-tool catalog, durable SSE agent runtime
- [[Mnemo]] — Local-first AI memory sidecar: knowledge graph from conversations via LLM extraction, SQLite + petgraph, no embeddings required
- [[Rowboat]] — Local-first AI coworker with persistent knowledge graph from email and docs
- [[Swamp Club]] — Agent-first workflow framework: Zod-typed models, DAG execution, encrypted vaults, immutable versioned data. From System Initiative
- [[Tau (τ) — Educational Coding Agent]] — MIT-licensed Python coding agent designed as a textbook: three-layer architecture (brain/environment/face), events-as-contract, durable JSONL sessions with branching
- [[OpenViktor]] — 48-hour AI employee platform that hit #3 on Product Hunt, then was killed and rebuilt as Jared. Blog post is password-protected; reconstructed from secondary sources
- [[Optimise Anything]] — Universal API: if it serializes to a string and quality is measurable, optimize it
- [[DSL-Driven Kanban Boards (Goja-Site)]] — Chainable JavaScript DSLs compose an entire kanban app declaratively: board, rendering, drag-drop, search, and DB — then mount on a router
- [[DAB]] — Microsoft's Data API Builder: REST, GraphQL, and MCP over any database
- [[Xano]] — No-code backend platform: AI-generated Postgres, APIs, auth, and logic with visual transparency as governance. Enterprise case studies at €22M/month scale
- [[InsForge]] — Open-source BaaS for coding agents: Postgres+RLS, auth, S3 storage, Deno functions, Stripe, OpenRouter — all exposed as MCP tools. Supabase for agents
- [[Nubase]] — Open-source AI-native backend and deploy platform: database-per-tenant isolation, Supabase-compatible auth + PostgREST, first-class Memory (Mem0-style fact extraction + hybrid vector/BM25/entity retrieval), Assets CDN, Edge Functions, Cron, and MCP bridge for Claude Code/Codex. Self-hosted single Docker image
- [[Semantic Kernel]] — Microsoft's agent middleware SDK: function-calling plumbing for C#, Python, Java enterprise codebases
- [[All Your Agents Are Going Async]] — HTTP is the wrong transport for agents that outlive connections; durable state is only half the problem
- [[Data Engineering for Large Models]] — Open-source textbook: complete LLM data pipeline, 28 chapters
- [[Open Design]] — Local-first design workspace with 16-agent runtime abstraction, skill pipeline, and five-panelist critique jury that auto-converges on quality thresholds
- [[OpenAI Structured Outputs]] — Guaranteed JSON schema adherence from the API: protocol-level constraint beats prompt-level pleading
- [[Layer-First Pattern — Keep Data Out of the LLM Context]] — Keep data server-side, return lightweight acknowledgments to the LLM. The diagnostic: if data passes through the LLM without the LLM making a decision about it, the architecture is wrong
- [[Tone LLM]] — The contract/adapter pattern in practice: LLM fills a small JSON schema, deterministic code translates to proprietary plugin config. Prompt-as-curriculum, four-layer output defense, and the case for single-call over agent loops when the task is form-filling
- [[Moltbook]] — Simon Willison on the AI-only social network bootstrapped via OpenClaw skills: heartbeat-driven agents, the lethal trifecta in production, and whether we can build a safe version
- [[Resident — ESP32 Lua Sandbox with Agent Skills]] — Sandboxed Lua runtime for ESP32 with hot-reload. AI agents write and push apps to physical devices via Claude Code plugin
- [[Golem Covenant]] — v0.1 spec framework for bounded, answerable, revocable agents: five-organ taxonomy (Mouth/Purse/Seal/Key/Sword), default-deny, tested return-to-dust before launch
- [[Your Coding Agent Should Do AI System Engineering]] — Ben Burtenshaw's three-level agent autonomy ladder (kernel writing → fine-tuning → auto-research lab), skills as few-shot context, and the case for open primitives over abstracted APIs
- [[xa11y — Desktop Automation via Accessibility APIs]] — Playwright-style desktop automation via accessibility trees on Windows/macOS/Linux: the structured alternative to vision-based computer use agents
- [[Open Source Agent Toolkit 2026]] — Paolo Perrone's definitive seven-layer map of the 2026 open source agent ecosystem: orchestration, memory, protocols, browsers, coding agents, evals, and inference as independent decisions gated by a single dominant constraint per layer
- [[The Case Against Building Your Own Agent Platform]] — Pete Johnson's sharp build-vs-buy triage for agent infrastructure: four underestimated components (memory, governance, eval, orchestration), five diagnostic questions, and the case that building *agents* on platforms is smart but building the *platform* isn't
- [[Giving Your Agent Eyes with Game Boy Hacking]] — Ian Langworth wires Gearboy + Ghidra + MCP so Claude can RE old ROMs; the real finding is that giving an agent a feedback loop to observe its own progress unlocks surprising behavior
- [[Grok Build Open-Sourced]] — xAI open-sources their 844K-line Rust terminal coding agent under Apache 2.0 after a privacy scandal where the CLI uploaded entire user directories to the cloud; Simon Willison's codebase analysis reveals system prompts, convergent tool design, and the disabled-but-present upload code
- [[Lessons from Building Vercel v0 and the d0 Agent]] — Malte Ubl on Dzero's two-tool architecture (bash + SQL, ~50 lines), V0's four-stage evolution driven by model leaps, the "make it look like coding" pattern, optimistic locking for shipping, and why teams make things go slower
- [[Ramp — Lessons from Building a New AI Product]] — Four Ramp engineers on shipping AI-native finance: single agent with thousands of tools, evals from day one, tool catalogs as shared infrastructure, and why Team A (impact-obsessed) beats Team B (performative code quality) when AI is a 10× amplifier
- [[Guiding Opus 4.8 Back to Sanity]] — valis diagnoses Opus 4.8's pedantic pushback as structural: obligation-voiced agentic layers overwhelm permission-voiced conversational guidance. The fix: an Object Floor that defines structural invalidity rather than prescribing virtues
- [[Sherlock Agent Eval]] — Alex Weil's detective board-game benchmark surfaces two LLM agent failure modes (fabrication-under-retrieval and the decoy trap) and a Theorist-Explorer split that breaks the trap even with weaker models. Topology beats model size
- [[Best Infrastructure Platforms for Coding Agents in 2026]] — Modal's vendor-biased survey of 7 sandbox platforms (Modal, E2B, Daytona, Blaxel, Together, Vercel, Cloudflare): CPU sandboxing is the primary workload, GPU is secondary, and the Firecracker convergence is real
- [[Umans Code for Organizations]] — Umans' org-tier docs: seat/service-account billing split (flat-rate humans, metered automations), 50% capacity pooling across seats, four-model tiered routing with per-token pricing from $0.15/M input, and the platform lock-in play hiding in the pricing page

## Agent Orchestration & Coordination

Multi-agent systems, task graphs, kanban boards, and coordination patterns. **Hub: [[Agent Orchestration]]**

- [[Agent Swarm Model Economics]] — Cursor's definitive 2026 technical report: planner/worker tree architecture, custom VCS at 1,000 commits/sec, five coordination failure modes at scale, and a head-to-head model economics comparison where Opus 4.8 + Composer 2.5 delivered a working SQLite-in-Rust for $1,339 vs GPT-5.5's $10,565
- [[Swarm Skill]] — jleechanorg's Claude Code playbook for orchestrating multi-agent swarms: 14 hard rules from concrete failures, mandatory sidekick durability layer, adversarial verification, cross-model cold review, and a publishability gate
- [[Paca]] — Self-hosted AI-native project management where agents are first-class Scrum teammates. WASM plugin sandbox, Docker-sandboxed agent execution, MCP throughout. Apache 2.0

## Memory & Context

Persistence, retrieval, knowledge management, and context engineering for agents. **Hub: [[Agent Memory and Context]]**

- [[Agent Memory]] — Angie Jones's definitive seven-type memory taxonomy (conversational, semantic, episodic, procedural, entity, working, summary) with Oracle's OAMP as reference implementation. "The hard part is judgment, not storage"
- [[Context Graphs]] — Karan Kalra on structured graph-based agent memory: capture the *why* behind decisions on the write path, not the read path. "Similarity is not relevance" — the case for typed edges over vector similarity as the retrieval primitive
- [[Sawtooth Memory]] — Async non-blocking hierarchical memory middleware: 4-tier stack (L0 system / L1 working / L1.5 entity ledger / L2 archival) with background asyncio compression that eliminates main-thread latency and guarantees deterministic fact retention. Dual-extraction compression prompt, local Ollama default with cloud provider adapters
- [[MELT]] — Shisa AI's benchmark harness for evaluating long-lived agent memory: tests lifecycle dynamics (correction, contradiction, decay, as-of recall, consolidation) not just static retrieval. Zero-dependency Python, pluggable SUT adapter contract, 4 built-in suites including native lifecycle benchmark
- [[So Long and Thanks for All the Context]] — Andrew Stellman's practitioner's guide to the U-shaped context problem: five field-tested techniques (curate, position at edges, short sessions, restate at point of use, test the middle) for when bigger windows create bigger middles to fall into
- [[MELT]] — Shisa AI's benchmark harness for evaluating long-lived agent memory: tests lifecycle dynamics (correction, contradiction, decay, as-of recall, consolidation) not just static retrieval. Its key finding: one "memory score" is never enough — ShisaD and Memobase flip rankings completely between lifecycle and LoCoMo QA. Zero-dependency Python, pluggable SUT adapter contract, 4 built-in suites.
- [[Context Engineering at the Frontier (Linus Lee)]] — Linus Lee argues bigger context windows are a brute-force crutch: composable retrieval pipelines beat monoliths for engineering velocity, context engineering IS search engineering, and the real gap is semantic observability at scale
- [[Giving Claude Agent Memory in 12 Steps]] — Codez's four-layer practitioner's ladder (Chat Memory → Projects → CLAUDE.md → Dreaming) for turning a goldfish agent into one that remembers across weeks; the most detailed public walkthrough of Anthropic's Dreaming research preview

## Quality & Guardrails

Evals, testing, linting, feedback loops, and keeping agent output trustworthy. **Hub: [[Guardrails and Feedback Loops]]**

- [[Metis — ARM AI Security Code Review]] — ARM's open-source AI security code review: tree-sitter call-graph reachability analysis for C/C++ + LLM vulnerability confirmation + deterministic adjudication gating; SARIF-native, 19+ languages, 10 LLM backends
- [[OpenCodeReview]] — Alibaba's open-source AI code review CLI: hybrid deterministic+agent architecture with per-file concurrent subagents, dual-threshold context compression, and a comment filter pass. Battle-tested across tens of thousands of developers
- [[brooks-lint]] — Pure prompt-engineering code review plugin: 12 decay risks from 12 classic engineering books, Iron Law diagnosis chain (Symptom→Source→Consequence→Remedy), six analysis modes across 11 platforms via Agent Skills
- [[FrontierCode]] — Cognition's mergeability benchmark: 36 repos, 20+ maintainers, measures whether a PR would actually be accepted by a human tech lead. Opus 4.8 leads at 13.4% Diamond
- [[A New Era for Software Testing]] — antirez on agentic QA: give an LLM a markdown checklist, let it inspect recent commits, and run targeted integration/regression/UX tests; automatic QA as compensation for lower-quality AI-generated code
- [[Accordant]] — Microsoft's model-based testing framework for .NET: write an executable spec (behavioral contract), and Accordant generates, executes, and validates hundreds of tests including sequential, concurrent, and async workflow coverage. The spec IS the oracle
- [[Agentic Testing]] — Slack Engineering's 200-run empirical study: MCP outperforms CLI by 12–20pp, generated tests fail 48% on complex flows, $15–30/run cost dominated by context accumulation. Agentic testing as exploratory layer atop deterministic E2E, not a replacement
- [[no-mistakes]] — Local git proxy that gates pushes through an AI-driven validation pipeline (review, test, document, lint, push, PR, CI) before forwarding to remote. Agent-agnostic, durable approval parking across daemon restarts, auto-fix with configurable limits

## Security & Sandboxing

Isolation, credentials, prompt injection defense, and agent safety. **Hub: [[Security and Sandboxing]]**

- [[AI Security Framework for DevSecOps]] — PreEmptive's practical guide to operationalizing AI security across the DevSecOps lifecycle: CI/CD enforcement, application hardening, and the framework taxonomy (NIST/EU/OWASP/MITRE/SAIF) that are complementary but not interchangeable
- [[Agent Skills for Security Testing]] — A library of 16 Claude Code skills for web application security testing, built from 4,000+ HackerOne bug bounty reports. Each skill distills vulnerability patterns into grep commands and curl tests
- [[Agentic AI Security Stack]] — Fernando Lucktemberg's free 200+ page reference: unified threat model tracing kill chains through 12 interception points, mapped to OWASP, MITRE ATLAS, and CSA MAESTRO
- [[ANSI Escape Sequence Injection in MCP Servers]] — Bright Security coins AESI: a prompt injection class exploiting the gap between how terminals render ANSI escape codes (invisible to humans) and how LLMs consume them (raw bytes in model-consumable MCP fields). Direct-fetch and stored variants, with a three-signal DAST detection methodology and a probe-phase approach for mapping unknown source-to-sink topologies
- [[Akmon]] — Tamper-evident evidence layer for AI agents: content-addressed, cryptographically signed session records verifiable offline with openssl. 95K LoC Rust workspace, 14 crates, built-in coding agent as reference producer
- [[Bounding the Blast Radius — Prompt Injection Defenses]] — Ibrahim Abdu's definitive 2026 survey: four-layer defense taxonomy, the "attacker moves second" proof that static benchmarks overstate robustness, and an economic reframe (cost-to-exploit > value-at-risk) as the only honest answer
- [[Enterprise-Managed MCP Authorization]] — Centralized MCP connector auth via Okta: provision once, zero-touch for users. The open MCP extension that turns auth from adoption blocker to invisible infrastructure
- [[How We Contain Claude]] — Anthropic's own containment engineering postmortem: three isolation patterns (gVisor, OS sandbox, sealed VM) and the incidents they didn't anticipate. The user-as-injection-vector problem, the 93% permission approval rate, and why custom code is always the failure point
- [[Interdict]] — Runtime safety layer between AI agents and PostgreSQL: parses every SQL statement through the real Postgres AST, measures blast radius on risky writes via simulated execution, and captures before/after images so every allowed write is instantly revertible
- [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]] — Microsoft's four-pillar framework for treating agents as first-class security principals: dedicated identities, task-scoped RBAC, multi-layer tool binding, and end-to-end auditability — with the identity ambiguity diagnostic as the sharpest insight
- [[cco]] — Zero-dependency bash wrapper that sandboxes Claude Code and other AI coding agents via macOS Seatbelt, Linux bubblewrap, or Docker. Agent-agnostic, cross-platform, 3,700 lines of bash with no runtime deps
- [[Tessera]] — Consent-gated remote access broker: 5K lines of Go, three binaries, human-approve-at-terminal flow with mTLS, end-to-end encryption, and append-only audit log. MIT-licensed alternative to Teleport for small-team just-in-time access
- [[Nango — Running Untrusted Customer Code at Scale]] — Nango's three-phase journey isolating untrusted customer code: vm2 (escaped) → per-customer runners (unfair) → tenant-pinned AWS Lambda on Firecracker microVMs, with an honest internal debate about whether per-customer Lambdas are progress or a workaround
- [[Malicious Agent Skills in the Wild]] — First large-scale measurement of the malicious agent skill ecosystem: 98,380 skills → 157 confirmed malicious, 632 vulnerabilities, 84.2% in natural-language SKILL.md files. Two archetypes (Data Thieves vs. Agent Hijackers), one industrialized actor behind 54.1% of attacks, 100% removal after disclosure
- [[Operational Groundwork for AI Agents]] — O'Reilly Superstream field report: five speakers converge on verification-over-trust — execution-layer enforcement, skill supply-chain audits, hygiene checklists, deterministic verification in fintech, and human signal as the only anti-slop measure
- [[VulnHunter]] — Capital One's open-source agentic AI security tool: closed-loop Hunt → Fix → Verify pipeline as Claude Code skills, forward-trace analysis with adversarial falsification, TDD-driven remediation with anti-merge math delivery gates
- [[Bumblebee]] — Perplexity AI's zero-dependency Go scanner for endpoint supply-chain inventory: reads lockfiles, MCP configs, and extension manifests without executing package managers, then matches against operator-supplied exposure catalogs for incident response
- [[DKIM2 and DMARCbis]] — Email's two core authentication protocols get their first major overhaul: DKIM2 adds replay-proof chains of custody and reversible forwarding recipes; DMARCbis replaces the brittle Public Suffix List with a DNS tree walk
- [[Transsion Telemetry — Embedded Mobile Surveillance]] — NowSecure researchers break Transsion's Athena/oneID telemetry encryption, revealing device-wide GPS, app-usage, and network surveillance on 200M+ phones; the SDK escapes OEM boundaries through third-party apps with 500M+ downloads
- [[VulnHunter]] — Capital One's open-source agentic AI security tool: attacker-perspective forward analysis with a falsification engine that tries to disprove its own findings before they reach developers. Claude Code skill, Apache 2.0
- [[Visa Vulnerability Agentic Harness (VVAH)]] — Visa's open-source 11-stage agentic SAST pipeline: LLM-driven discovery, adversarial verification, automated remediation, and agentic validation panel — targeting Mean Time to Adapt as primary metric
- [[Web Application and API Protection (WAAP)]] — PreEmptive's overview of WAAP's four-capability platform (WAF, bot management, DDoS, API security), its perimeter limitation, and why code-level protection fills the gap — especially as AI tools collapse the cost of reverse engineering

## Software Engineering

Craft beyond agents: simplicity, error handling, reliability, specs, and project management. **Hub: [[Software Engineering Craft]]**

- [[Notes on Structured Programming]] — Dijkstra's 1970 foundational monograph: structured programming, step-wise refinement, the testing-versus-correctness argument, and the layered virtual machine model that anticipated microservices, containers, and agent abstractions
- [[Engineering for Bounded Cognition]] — Working memory holds ~4 chunks; attention is a torch beam. Software methodology as prosthetic cognition, and why designing for the most constrained user produces better systems for everyone
- [[Auditing Legacy Rails Codebases]] — Ally Piechowski's nine diagnostic questions tiered by audience (developers, CTOs, stakeholders) that surface the friction points no one volunteers in status meetings — a lightweight codebase audit that doesn't require looking at code
- [[Lessons for Reusable Web Components]] — Daniel De Pietro's five field-tested rules for web components: namespace everything, CSS variables as the public API, trust modern platform features, publish over paste, document or it didn't happen
- [[Your Backend Is Full of Hidden Workflows]] — How backend codebases quietly accrete coordination logic across services, queues, and handlers until teams are managing workflows they can't see. The three costs: expensive changes, painful debugging, eroded trust
- [[99 Bottles of OOP]] — Sandi Metz's practical workbook: OO design as line-by-line decision-making, the Flocking Rules as structured refactoring, and "programming aesthetic" as the antidote to evaluative anesthesia
- [[Code-First Developer]] — Khalil Stemmler's five-phase model of developer craft growth: from code-first through value-first, the Expert Junior Developer trap, and why mastery beats breadth when AI makes coding cheap
- [[Command Line Interface Guidelines]] — The canonical open-source reference for modern CLI design by Docker Compose co-creators: human-first philosophy, concrete guidelines across help/output/errors/flags/subcommands/config/naming, and the case that CLIs are conversations, not atomic invocations
- [[CQRS Pattern in C# and Clean Architecture]] — Nick Cosentino's beginner guide to combining CQRS with Clean Architecture in C#/.NET: clear definitions and MediatR-style code examples, strongest as a conceptual on-ramp rather than a production guide
- [[Observer Pattern to Event-Driven Architecture in Dart]] — Oluwaseyi Fatunmole's handbook tracing Observer → EventBus → Domain Events → Riverpod in production Dart/Flutter, with the clean separation: use cases own consequences, notifiers own UI state, widgets own nothing
- [[Event Sourcing — Set-and-Remove Bi-Temporal Events]] — Urs Enzler's part-twelve deep-dive distinguishing lifetime (create-update-delete) from set-and-remove bi-temporal event streams, with the override-vs-insert choice as explicit per-stream configuration
- [[Litmus (.NET Testing Priority Tool)]] — Ebrahim Sayed Ebrahim's free .NET CLI tool that ranks files by testing priority via a two-phase formula (churn × coverage × complexity, then discounted by coupling) with Roslyn-based seam detection for testability
- [[Queues Don't Fix Overload]] — Fred Hebert's 2014 classic on why queues treat symptoms not causes: identify the bottleneck, then back-pressure or load-shed; everything else makes failures rarer but more catastrophic
- [[Patreon Notification Fanout]] — Patreon's two-stage fanout architecture rebuild: 80% faster push/in-app, 55% faster email; the 16× migration velocity jump when leadership aligned on deadlines, and AI-assisted migration via Claude Code skills across 10 teams
- [[Lean, Not Backpressure]] — kqr argues backpressure is the wrong frame for Costa's quality-upstream prescriptions; lean manufacturing (single-piece flow, jidoka, poka-yoke) is the better metaphor, and AI tools make the blame-the-worker reflex impossible to sustain
- [[21 Years and Counting of Eight Fallacies of Distributed Computing]] — The canonical list of network lies developers tell themselves, born at Sun Microsystems, still sharper after 21 years of Internet evolution
- [[DDB — Source-Level Interactive Debugging for Distributed Applications]] — Yan, He, and Park (USC, 2026) extend interactive debugging to distributed systems: cross-RPC backtraces, breakpoints that survive autoscaling, and virtualized clocks that prevent timeout cascades. 100% fault localization vs. 38.5% for GDB+OpenTelemetry, with 1–5% overhead
- [[Signals — The Push-Pull Algorithm]] — Willy Brauner builds the push-pull reactive algorithm in ~80 lines of TypeScript: eager invalidation + lazy re-evaluation + global-stack dependency tracking, the pattern behind Solid, Vue, Preact, Angular, and Svelte
- [[The Cost YAGNI Was Never About]] — Kent Beck reframes YAGNI as options pricing + NPV, not thrift: cheap AI generation amplifies the trap, not the escape
- [[Canonization and the Overhang]] — Kellan Elliott-McCrea names canonization (turning disposable code into reusable library-grade work) and the overhang (latent value from past creativity) as the two concepts that explain what AI coding actually changes — and the warning that rewarding only production depletes the cognitive seed corn
- [[The Minimum Viable Unit of Saleable Software]] — Brandur's buy-vs-build economics in the LLM era: Jira's 37-month break-even, the zone of viability, and why "cheap != zero" when humans still cost $96/hour
- [[The Persistent Gravity of Cross Platform]] — Allen Pike on why coordination costs create an irresistible pull toward cross-platform frameworks at scale, and why agentic coding (per the 2026 addendum) only deepens that gravity by making human verification the bottleneck
- [[The Joy and Power of Understanding]] — Igor Roztropiński's compact manifesto: LLMs are force multipliers but you must have force first, struggle is necessary for mastery, and understanding is both the pragmatic path and the intrinsic reward
- [[How I Use HTMX with Go]] — Alex Edwards' definitive field guide to integrating HTMX with Go: template architecture (base/pages/partials), the htmlRenderer abstraction for unified full-page and partial rendering, dual-mode endpoints via HX-Request header detection, redirect management, and opinionated HTMX configuration defaults
- [[How to Write an Effective Software Design Document]] — Michael Lynch's 23-component design doc checklist from Google/Microsoft/startup experience, anchored on one question: "what's the penalty for being wrong?"
- [[HTTP API Design Guide (Heroku)]] — Heroku's 2013 HTTP+JSON API design conventions (versioning via Accept header, structured errors, UUIDs, flat paths) that became de facto industry standards. The ur-text of modern REST API design
- [[Hidden Inefficiencies Behind Delivery Delays]] — Ahmed El-Deeb's taxonomy of five invisible delivery killers (instability rework, priority churn, cross-team coupling, technical debt, review queueing) with measurable indicators for each and a catalog of where prevailing theory breaks against operational reality
- [[Claude Is Not Your Architect]] — Charlie Holland's polemic against letting AI slide from implementation assistant to architectural decision-maker: the attaboy problem, context-blind design, the Jira ticket pipeline, and why "'Claude designed it' is not an ADR"
- [[Multi-Tenancy Isn't About Databases]] — Derek Comartin argues multi-tenancy discussions fixate on the wrong question: the real question is what you're trying to isolate, and every shared resource creates coupling. A database is just one boundary among many
- [[Software Engineering at the Tipping Point]] — Adam Bender's 2026 Google talk: AI is a 10× amplifier, not a directed solution. Software ecology as a framework, shared fate, and why every node in your developer ecosystem breaks at 10× scale
- [[Product-Minded Engineers in an AI-Native World]] — Thomas Pauls (Linear), Drew, and Michelle (Flint) on product engineering as motivation not role, taste as trainable craft, Quality Wednesdays, and AI as product-skill multiplier
- [[Discovery Debt]] — Benedikt Kantus names the accumulated weight of untested assumptions that compounds invisibly until products are expensively wrong; four habit-level remedies and why speed feels like signal but isn't
- [[Martin Fowler and Kent Beck on Reinventing Software]] — Two Agile Manifesto authors on AI's unprecedented magnitude, total skepticism as discipline, the re-soloing illusion, DX=AgentX convergence, and why nobody has the answers anymore
- [[Domain Storytelling]] — Collaborative modeling method using pictographic sentence diagrams: workshops where a moderator visualizes domain experts' stories as who-does-what-with-what sentences. Interview with co-creators Hofer and Schwentner
- [[Solution Engineering Advice]] — Max Halford's unvarnished field manual for customer-facing technical roles: three-phase lifecycle (presales→pilots→onboarding), when to bend the truth, how to manage Sales, and why Extreme Ownership is the only viable mindset
- [[Why Build vs Buy is the Wrong Question]] — Chris James reframes the build-vs-buy debate through Evans's DDD subdomain taxonomy (generic, core, supporting), arguing the third category is where most organizational waste hides and where AI most reduces the cost of building
- [[Fintech Engineering Handbook]] — Voytek Pitula's definitive pattern reference for building software that handles money: three principles (no invented data, no lost data, no trust) organised into patterns for representation, ledgers, money flows, external integration, and controls
- [[Enterprise-Grade CI at Solo Founder Scale]] — Lionshead on running a full CI pipeline (security scanning, cost gates, preview environments, schema-migration validation) as a solo founder; CI as externalized memory, not team-size-dependent ceremony
- [[The Single-Tenant Trap]] — Auth0's Carlos Aguilar on why running dev and prod in a single SaaS tenant is an operational anti-pattern: conditional routing can't partition MFA/sessions/connections, configuration-as-code is the enabler for multi-environment setups, and least-privilege RBAC is forensic infrastructure not just security theater
- [[Edge Computing for Web Developers]] — Manikanda Akash Munisamy's practical field guide to the CDN-cloud-edge triad: when edge beats serverless, platform costs at scale, and the hybrid architecture nobody's written yet
- [[The Lindy Effect in Software]] — Clément Sauvage applies the Lindy effect (survival implies fitness) to technology choice: old tech isn't inertia, it's a positive signal of robustness that newer alternatives structurally cannot offer. Lindy is YAGNI for architecture
- [[Zero-Cost Fallacy of Open Source]] — Chris Ford and Richard Gall diagnose open source's structural economic failure: permissive licensing enabled the ecosystem and the exploitation vector, and AI slop, trust degradation, and the licensing paradox are accelerating a slow-motion collapse. "We've confused permissive licensing with a license to exploit"
- [[The Lindy Effect]] — Technology-choice heuristic: the longer something has survived, the longer it's likely to keep surviving. A counterweight to shiny-object syndrome from the Laws of Software Engineering collection
- [[Your Distributed System Is Slower Than a Laptop]] — CodeGood revives the 2015 COST paper: a $1.4M/year Kafka+Flink pipeline costs 24× more than a single-server alternative that's faster; the coordination tax is paid in salaries, not CPU cycles — and the industry still doesn't run the comparison
- [[Reduce Logging Costs]] — Michael Shpilt's five-strategy taxonomy for cutting observability spend: sampling, tiered storage, in-code cleanup, vendor migration, and the nuclear option of killing INFO logs. Core thesis: sustainable savings come from reducing telemetry before it leaves the application, not cheaper storage after the fact

## Databases & Data

Storage engines, query patterns, data quality, and vector/graph databases. **Hub: [[Databases and Data]]**

- [[Aurora DSQL]] — AWS's serverless multi-region active-active SQL database: disaggregated compute/storage/coordination, PostgreSQL-compatible, MVCC with clock-based coordination-free reads and commit-time optimistic concurrency. Scales from zero to millions of TPS
- [[Streambed]] — Postgres-to-Iceberg CDC in a single Go binary: WAL streaming, Parquet+S3, embedded DuckDB query server with psql-wire. Jepsen-style simulation testing, no Kafka/JVM/Spark needed
- [[DuckDB ADBC Extension]] — DuckDB gets a universal Arrow-native connector to 30+ databases (Snowflake, Databricks, BigQuery, Postgres, MySQL) via ADBC — the JDBC moment for the columnar ecosystem. `read_adbc` and `ATTACH` with connection pooling, metadata caching, and streaming bulk ingest
- [[Artie]] — Managed CDC replication: sub-minute latency from Postgres/MySQL/MongoDB to Snowflake/Databricks/BigQuery. Zero data retention, no Kafka required. The "buy vs. build" alternative to self-managed Debezium pipelines
- [[The Limits of Generalized Sync]] — Siidorow's master's thesis: the most rigorous empirical study of sync engines (Zero, ElectricSQL, PowerSync, Convex, etc.), classifying 14 engines into 4 architectural clusters. Central finding — read paths generalize, write paths resist: the online/offline boundary determines everything downstream. 7 trade-offs, 5 fundamental limits, production case study
- [[Lakebase and LTAP]] — Reynold Xin on Databricks' stateless Postgres architecture and the LTAP paradigm that eliminates CDC by storing operational data once in open columnar formats for both transactions and analytics
- [[Wes McKinney on Pandas, Arrow, and Data Infrastructure]] — Wes McKinney on the accidental origin of Pandas, Arrow's decade-long adoption curve, why foundational data infrastructure resists AI replication, and the full-circle return from distributed back to single-machine columnar engines
- [[DocumentDB]] — Microsoft's MongoDB-compatible document database on PostgreSQL: native BSON type, 10K-line aggregation pipeline compiler in C, Rust wire-protocol gateway with read-ahead pipelining
- [[FlareDB]] — Apache Beam-native streaming database in Rust: PCollections become Arrow-backed LSM-tree tables, dissolving the boundary between pipeline processing and durable storage
- [[Learning a Few Things About Running SQLite]] — Julia Evans' field report: four years of SQLite in production, the ANALYZE epiphany, concurrent-writer pain, and the gap between "SQLite is fine" and actually operating a database
- [[Grudge]] — Constant-memory decaying-score probabilistic sketch for Go: count-min sketch semantics applied to behavioral scores with lazy time-decay and collision-shielded min-aggregation. Extracted from FAIR
- [[ARIES — Write-Ahead Logging Recovery]] — Mohan et al.'s 1992 paper that defined crash recovery for every major database: repeat-history paradigm, per-page LSNs, CLR chaining, three-pass restart. The architecture still running inside PostgreSQL, SQL Server, DB2, and InnoDB
- [[Postgres Transactions Are a Distributed Systems Superpower]] — Kraft & Li argue co-locating workflow state with application data in Postgres eliminates idempotency bugs and outbox infrastructure through shared transaction boundaries
- [[Meerkat — QuePaxa Consensus at Cloudflare]] — Cloudflare Research's internal consensus service using the QuePaxa algorithm: leader-optional writes, no timeout-based unavailability, 10x throughput over Raft in adversarial networks
- [[AntFly]] — Distributed search engine and AI-native database: multi-Raft consensus, hybrid BM25+vector+graph search, built-in ML inference, Go/Zig dual-stack. Embeddings/chunks/graph edges generated automatically at write time. TLA+-verified protocols, Jepsen-inspired simulation testing
- [[Chatto]] — Self-hosted real-time chat application using NATS/JetStream as its sole data store with event sourcing and in-memory projections — no SQL database at all
- [[SQLite Is All You Need]] — DB Pro's benchmarked case for SQLite as a production web backend: 50K users, 1M posts, 3,654 req/s on one file. The WAL-vs-rollback comparison and the "99.99%" argument that user acquisition, not database throughput, is the real bottleneck
- [[How to Corrupt an SQLite Database]] — The SQLite team's exhaustive catalog of every corruption failure mode: filesystem lies, OS quirks, application bugs, hardware deception, and SQLite's own historical defects
- [[Malloy]] — Open-source semantic modeling and query language that compiles to SQL: two-phase compiler (ANTLR → AST → IR → dialect-specific SQL), symmetric aggregates for correct join handling, pipeline query model, and a Solid.js + Vega renderer with plugin system. Ex-Google/Looker team, 93K lines TypeScript
- [[Redb Ecosystem]] — Three-layer Apache 2.0 .NET stack: typed LINQ-native database (POCO-as-schema, Postgres/MSSQL/SQLite), Apache Camel-style integration engine (30+ connectors, EIP DSL), and clustered runtime with dashboard. 3.3.0 fixes silently-broken concurrency across RabbitMQ/Kafka/AMQP and adds built-in RAG pipeline primitives
- [[Bad Data in Production — Response Playbook]] — Pinal Dave's six-step incident response playbook for data quality failures: triage, contain, trace lineage, fix-and-verify, notify stakeholders, blameless review
- [[Searchable Field-Level Encryption with CipherStash]] — CipherStash brings Data Level Access Control to Supabase/Postgres: field-level encryption with searchable metadata, zero-knowledge key management, and wire-protocol proxy for non-SDK access

## Developer Tools

CLIs, utilities, document tools, code analysis, recording, and infrastructure.

- [[Common Expression Language (CEL)]] — Google's embeddable expression language for policy and validation: non-Turing complete by design, nanosecond-to-microsecond evaluation, protobuf-native
- [[CI Forge (ciforge)]] — Zero-dependency Python CI tool bundling ~25 scanners: code quality, secrets, IaC, dead code, CVE, cloud cost, AI review across 3 providers, and an MCP server. AGPLv3, replaces Snyk/SonarQube for solo devs
- [[Nektos Act]] — Run GitHub Actions workflows locally in Docker: full expression evaluator, YAML-level interpolation, and composable functional executor pipeline
- [[Component Model 1.0]] — Bytecode Alliance's roadmap to a stable Wasm Component Model: lazy ABI, browser native support via jco telemetry, spec simplification, and the WIT expressivity gaps that remain
- [[Git Diff Drivers]] — git's external diff driver interface: the 7-argument contract, `/dev/null` lifecycle sentinels, and a worked `oasdiff` example
- [[Google Workspace CLI]] — One Rust CLI for all Google Workspace APIs. Dynamic command surface
- [[Google Workspace CLI Skills]] — Structured skill catalog: 19 services, 25 helpers, 10 personas, 40 recipes. A designed taxonomy for agent-tooling
- [[VHS]] — Terminal GIF recorder from Charm. Write recordings as scripted .tape files
- [[Portless]] — Named .localhost URLs for local development. For humans and agents
- [[QuickEmu]] — QEMU wrapper that auto-configures VMs. Nearly 1000 OS editions
- [[Maestro (UI Testing)]] — End-to-end UI testing for mobile and web. YAML DSL, visual inspector, enterprise cloud
- [[FlaUInspect]] — Windows UI Automation inspector for browsing UIA trees. Open-source Inspect.exe replacement with UIA2/UIA3 backends, overlay highlighting, and XML export
- [[Sampo]] — Changelog and release automation across monorepos and registries
- [[Windows in Docker]] — Headless Windows 11 in Docker over SSH. No GUI, just Claude Code on Windows
- [[Dev Containers]] — VS Code's infrastructure-as-code for dev environments: container as the source of truth, Features as composable toolchain components, pre-built images as self-describing specs
- [[Container-Maker]] — Solo dev's ambitious CLI wrapping devcontainer.json into a standalone platform with AI config generation and cloud GPU provisioning. Vision document, not a recommendation
- [[Installing VS Compilers From Commandline]] — msvcup: skip Visual Studio, install just the compiler and SDK
- [[Introducing git-wt — Worktrees Simplified]] — Bash wrapper smoothing git worktree's sharp edges: auto-fetch, upstream tracking, orphan cleanup, fzf switching
- [[Code Storage]] — API-first Git infrastructure for machines: programmable repo creation, warm/cold tiering, custom-domain endpoints. The bet that agent-created repos will outnumber human-created ones
- [[CORS Fetch Tester]] — Simon Willison's browser-based CORS debugging utility: send HTTP requests and inspect exactly what the browser lets you see through CORS
- [[grok-mermaid — Terminal Mermaid Renderer via WebAssembly]] — Simon Willison's browser tool that converts Mermaid diagrams to Unicode box-drawing art using the Rust renderer from xAI's Grok CLI, compiled to a 163 KB WebAssembly module
- [[bcc]] — BPF Compiler Collection: kernel-level tracing for Linux performance analysis
- [[Tmux Resurrect]] — Persists and restores complete tmux environments via tab-delimited flat file serialization; idempotent, zero-config, mini DSL for process matching
- [[Zellij]] — Rust terminal multiplexer (~296K lines) with WASM plugin system, built-in terminal emulator, per-client rendering, OSC 99 host query forwarding, mobile mode, and session resurrection
- [[floci]] — Free local AWS emulator replacing LocalStack. 47 services, 24ms startup
- [[sem]] — Semantic version control: entity-level diff via tree-sitter across 31 languages, 5-phase entity matching with structural hashing, scope-aware reference resolution, and agent-native JSON/MCP output
- [[markitdown]] — Microsoft's office-docs-to-Markdown converter for LLM pipelines
- [[Kiso]] — OKF-to-static-site publishing engine that generates agent-friendly output with llms.txt and source backlinks
- [[Klangio Transcription Studio]] — Browser-based AI polyphonic music transcription to sheet music, MIDI, and TABs. 4M+ transcriptions
- [[How to Follow a Drummer]] — Sashyo's field report on building DrumMate: a phase-locked loop with beat-aware front end that lets the drummer lead and the machine follow, inverting forty years of click-track tyranny
- [[Kreuzberg]] — Polyglot document intelligence: 97+ formats, Rust core, MCP server
- [[claude-replay]] — Agent sessions as self-contained embeddable HTML replays
- [[Mindwalk]] — 3D visualization tool replaying coding-agent sessions as light moving through a night map of your codebase. Go binary, fully local, with sealed LLM session evaluation
- [[Subtext]] — Real-time Jacobian lens instrument streaming an LLM's internal "silent words" to a browser canvas during live conversation; watches the model plan, judge, and reason before it speaks
- [[engineering-notebook]] — Automatic engineering diary from Claude Code and Codex sessions
- [[session-analysis]] — Analyze agent session JSONL for wall time, tokens, and cost
- [[atifact]] — Zero-dependency CLI converting HAR files, Claude Code, Copilot CLI, and Codex CLI logs to ATIF v1.7 trajectory JSON
- [[AI Pricing]] — Free JSON API for per-token AI model pricing across 19 providers. Agent-native, no auth
- [[AgentsView]] — Local-first analytics dashboard for 24+ coding agents
- [[Broomy]] — MIT-licensed Electron desktop app running multiple coding agents side-by-side with built-in IDE and code review
- [[Browser Use]] — AI browser automation with anti-detection and deterministic rerun
- [[Webwright]] — Microsoft Research: turns coding models into SOTA browser agents via terminal + Playwright. Code-as-action, self-verifying, ~1.5K LoC
- [[Chunker (Document Chunking Tool)]] — Python CLI that builds navigable knowledge trees from documents: LLM-mediated semantic chunking + bottom-up hierarchical summaries for progressive disclosure
- [[Chrome DevTools MCP — Debug Your Browser Session]] — Chrome M144's `--autoConnect` lets agents reuse authenticated browser sessions. Hybrid manual/AI debugging via permission-gated remote debugging
- [[surf-cli]] — Browser automation for agents via CLI and Unix sockets. No MCP needed
- [[Claude Artifact Server]] — 22 Claude-generated retro-Mac interactive artifacts produced in a single day: generation-at-scale showcase, not a product
- [[OpenRewrite Supported Languages]] — Capability catalog: 5 languages, 7 data formats, 3 build tools, 4 frameworks. OSS/commercial split where JVM is free and polyglot is paywalled
- [[OpenWiki]] — LangChain's CLI that runs a DeepAgent to generate and maintain OKF-compliant documentation wikis for codebases and personal knowledge bases, with built-in connectors for Gmail, Slack, Notion, X, and more
- [[TriadJS]] — TypeScript API framework: write schemas once, derive types, OpenAPI, BDD tests, DB schemas, frontend hooks, and WebSocket clients from a single source of truth. AI-first design with Claude Code plugin
- [[Ponytail]] — Multi-platform AI coding agent plugin: "lazy senior dev" persona forces YAGNI → stdlib → native → one-line before writing code. Cuts 54% LOC without dropping safety. 14+ host adapters
- [[sx]] — Team package manager for AI coding assistant assets: skills, MCP configs, commands, hooks. Manifest-and-lock pattern, scoped install
- [[DeepWiki]] — Cognition's instant codebase wiki: swap github.com for deepwiki.com, get AI-powered Q&A with line-level citations
- [[graphify]] — Codebase to multimodal knowledge graph. Code, PDFs, screenshots, diagrams
- [[hblog-ng]] — Obsidian vault-to-website static site generator with canvas-rendered knowledge graph, PGP-signed posts, and triple-feed syndication
- [[lat.md]] — Knowledge graph for codebases in markdown with validation against drift
- [[Lumide]] — Cross-platform Flutter/Dart IDE with an agentic sidebar: 200ms cold start, 80MB idle, no Electron layer
- [[docmason]] — Local knowledge base from office documents with citations and source tracing
- [[Trailmark]] — Trail of Bits: source code as a queryable graph for security analysis
- [[Understand-Anything]] — Claude Code plugin: builds persistent knowledge graphs from codebases with tree-sitter + LLM pipeline, incremental git-hook updates, and interactive dashboard
- [[Clipfan]] — Fleet-wide clipboard sync over SSH with headless image paste for Claude Code and Codex CLI. Three-layer dedup, AES-GCM encryption, tmux integration. From Prime Radiant
- [[cmux]] — macOS-native terminal built on libghostty, designed for managing multiple AI coding agent sessions. Notification rings flag panes that need human attention. Free, by Manaflow
- [[Clearance]] — Native macOS Markdown viewer/editor from Prime Radiant. Swift, local-first, YAML frontmatter support
- [[MarkText]] — Open-source GUI Markdown editor. WYSIWYG, cross-platform
- [[Mist]] — Google Docs for Markdown. Real-time collaboration, no accounts
- [[Music Decoy]] — macOS utility that stops Music.app from auto-launching by impersonating its bundle ID. Zero CPU, zero work
- [[magika]] — Google's AI file type detection: 200+ types in 5ms, deployed at Gmail scale
- [[smui]] — Terminal-aesthetic theme for shadcn/ui. Nord palette, monospace, zero radius
- [[json-render]] — Vercel Labs' generative UI framework: LLM outputs JSON constrained to a Zod component catalog, rendered progressively. 15k stars
- [[Extend UI]] — Open source React component library for document apps: PDF/DOCX/XLSX viewers, bounding box citations, e-signing, schema builder. 560 stars, targets agent-built document UIs
- [[Phoenix LiveView]] — Server-rendered real-time UI without JavaScript: each view is a BEAM process, state changes push HTML diffs over WebSocket. The UI layer of the concurrency model agents keep reinventing
- [[QMD]] — Local CLI search engine: hybrid BM25 + vector + LLM re-ranking
- [[Recoll]] — Full-text desktop search engine built on Xapian: indexes documents inside archives inside email attachments, two decades of boring-correct engineering
- [[Claude Lamp]] — LED lamp controlled by Claude Code's state via Bluetooth
- [[Muxcard]] — Credit card-sized computer (~1mm thick): ESP32-C3, e-paper display, NFC reader/writer, strain-isolated flex PCB
- [[Pretext]] — Pure JS/TS text measurement and layout without DOM reflow. 46.9k stars
- [[Process Flow]] — Choreography-as-a-service: each HTTP stage designates its successor, no central workflow DAG
- [[n8n]] — Visual workflow automation with 400+ integrations, AI nodes via LangChain, fair-code licensed. 188k stars
- [[Budibase]] — Open-source low-code platform: Svelte visual builder, CouchDB data engine, 12+ datasource adapters, Bull/Redis automation engine. Full-stack app builder for internal tools. GPL-3.0
- [[Micasa]] — TUI for home maintenance, projects, and vendor quotes. Pure Go, vim-style
- [[Zed]] — Rust-native code editor from the Atom/Electron/Tree-sitter team: AI as first-class substrate, not a bolt-on
- [[DeltaDB]] — Zed's version control for the agent era: deltas replace commits, every line of code is bidirectionally linked to the conversation that produced it, CRDT-backed worktrees for concurrent human+agent editing
- [[Dolphin]] — ByteDance's universal document parsing model. Digital and photographed docs
- [[QR Generator (delphi.tools)]] — Indie web QR code tool with live preview and deep customization. "No logins. No tracking. Long live the handmade web"
- [[HeidiSQL]] — Free open-source database GUI for 7 engines, maintained solo since 2002. Delphi/FreePascal, cross-platform, no pricing page
- [[HTML Table Extractor]] — Simon Willison's browser tool that extracts HTML tables from pasted rich text, exports to 5 formats, and auto-fetches Wikipedia tables via open CORS API
- [[LLM Cliché Highlighter]] — Simon Willison's browser tool that highlights sentences matching known LLM clichés ("delve," "tapestry," chain patterns) with hover-to-see-which-cliché; practical self-diagnostic for AI-assisted writing
- [[SQL to ER Diagram]] — Open-source browser-based ERD generator from SQL DDL: ~3,200 lines of vanilla JS, zero backend, surgical bidirectional editing via source spans, URL-hash sharing. Live at sqltoerdiagram.com
- [[Hitomi (Data Viewer)]] — Flutter desktop data viewer with streaming ETL, custom filter language with its own compiler, and chunk-boundary-safe parsing for CSV/TSV/custom formats
- [[Ratty]] — GPU-rendered terminal emulator with inline 3D graphics via custom Ratty Graphics Protocol. Bevy game engine as terminal substrate, terminal surface as deformable 3D geometry
- [[Building the deployment tool I wish I had]] — Deptool: Git-backed deployment with atomic symlink swaps, auto-rollback, and a static binary agent that needs only SSH+coreutils
- [[Gova]] — Declarative reactive GUI framework for Go: SwiftUI-inspired API, call-site state identity via runtime.Caller, Fyne bridge stays internal, hot-reload dev server
- [[Stash — Conflict-Free Folder Sync]] — TypeScript CLI syncing any folder via GitHub: three-way text merge with diff-match-patch, dual drift detection, OS-level background daemon
- [[Textverified]] — Temporary US phone numbers for SMS/voice verification: carrier SIMs (non-VoIP), 900+ services, API + crypto payments, from $0.25/use
- [[zeroserve]] — Linux HTTPS server that serves from tarballs and runs eBPF scripts JIT-compiled in-process with a branchless pointer cage sandbox. Compiles Caddyfiles to eBPF middleware
- [[Cloudflare OAuth for All]] — Zero-downtime Hydra migration (132M rows, -45% P95 latency) to open self-managed OAuth to all Cloudflare customers; queue-based revocation replay during blue-green cutover
- [[Local Review]] — Local-first git branch review tool: leave line/range comments, export as markdown for coding agents, with drift-resistant anchoring via git diff tracking and snippet matching. Single Go binary, served from localhost
- [[Celly — Native .NET CEL Implementation]] — Pure C# implementation of Google's Common Expression Language: 100% spec conformance, zero dependencies, faster than the Go reference on comprehension-heavy workloads
- [[Document Generation in .NET]] — Deepika Kathiravan's four-way taxonomy of C# document generation: code-built, HTML-to-PDF, headless libraries, and template-based with editor — with the organizational question "who owns the template?" as the real decision criterion
- [[Tunnet]] — Open-source mesh VPN platform bundling mesh networking, serve, tunnel, send, and SSH under one identity system and policy engine. Rust (~26K lines) on iroh/QUIC, two modes (Managed with control plane, Direct with CRDT membership), self-hosted relay

## Local & Personal Computing

Tools that run on your own machines: voice, hardware, local inference, knowledge apps. **Hub: [[Local and Open Source Inference]]**

- [[Datacenter GPU in a Gaming PC]] — £200 eBay V100 in a gaming rig: hardware hacking, NixOS driver archaeology, and Qwen3.6-27B at 32 tok/s
- [[Dual GPU RTX 5080 + RTX 3090 Qwen 3.6 Setup]] — iMil's field report: RTX 5080 + RTX 3090 running Qwen 3.6 27B Q8 at 80+ tok/s via tensor-split speculative decoding on an Asus X570-Pro
- [[LocalAI]] — Open-source drop-in replacement for the entire cloud AI stack: inference engine, agent runtime, and memory service in a composable gRPC backend architecture. 40k stars, MIT licensed
- [[AirLLM]] — Runs 70B+ LLMs on 4GB GPUs by streaming one transformer layer from disk at a time. 405B Llama 3.1 on 8GB, 671B DeepSeek-V3 on ~12GB, no quantization required
- [[Bonsai 27B]] — PrismML's ternary/binary quantized Qwen 3.6 27B at 3.9 GB, the first 27B-class model to fit on a phone. 262K context, multimodal, Apache 2.0
- [[Indexing 669 GB of GoPro Videos with Local ML]] — Ilias Haddad's local-first pipeline for semantic video search: Whisper + YOLO + DeepFace + Qwen2.5-VL on an M1 Max, 67h compute for 15h of footage. The Docker-on-Mac GPU gap as a real constraint on local ML tools
- [[Local Qwen Is Not a Worse Opus]] — Alex Ellis's founder field report: local models are a different tool from frontier models, not a worse version. Revenue recovery that paid for a $12K GPU, the looping problem, and the "analysis not interpretation" boundary

## AI Research & Models

Papers, model capabilities, training techniques, and the state of the field.

- [[The Hitchhiker's Guide to Agentic AI]] — Haggai Roitman's 603-page practitioner's reference: the full agentic AI stack from transformer architecture through production deployment, with theory + implementation + code for every layer
- [[Holding the LLM Stack in Your Head]] — Nick Gustafson's ~84-post dependency-ordered walk through the entire modern LLM stack, from linear algebra to agent protocols, written as a public learning exercise with Claude Opus 4.8
- [[Open Source AI Map]] — Curated, hand-scored catalog of ~458 OSS AI products across 15 categories with three-axis scoring (openness/adoption/capability), deterministic stage/gap analysis, and the Columbia/MOF openness framework
- [[Open Source AI Gap Map (Willison)]] — Simon Willison's link-blog lens on Current AI's Gap Map: the MIT-licensed dataset matters more than the visualization, Datasette Lite as universal data browser, and curation-as-infrastructure in AI landscape mapping
- [[State of Open Source AI 2026]] — Mozilla's first annual assessment of the open-source AI ecosystem: capability gap is jagged (parity on coding, behind on reasoning), inference costs collapsed 50×, Chinese open weights route 3× more tokens than US, and the harness is the new frontier
- [[Proxy-KD — Knowledge Distillation of Black-Box LLMs]] — Proxy-mediated distillation from GPT-4 to 7B students: beats white-box KD by approximating closed-source probability distributions through DPO-aligned intermediate models
- [[The Reasoning Trap]] — Yin et al. demonstrate that every technique enhancing LLM reasoning (RL, distillation, thinking toggles) also amplifies tool hallucination, with the reasoning step itself as the causal driver and late-layer residual streams as the mechanistic locus
- [[Learn AI Layer by Layer]] — Rob Ennals' interactive tutorial explaining AI from numbers to transformers with browser playgrounds and Colab notebooks. Best free AI foundations resource available, built for his 11-year-old son
- [[Goldman Sachs World Model]] — Goldman Sachs Global Institute: world models as AI's next leap beyond text prediction toward internal simulation of physical and social reality
- [[Guardian Angels]] — Gwern Branwen's vision for personalized "digital twin" LLMs that emulate a single user's personality, solving the principal-agent problem through continual learning, dynamic evaluation, and an append-only-log UX; the $1,000/month startup playbook for building an AI that substitutes for its principal
- [[LLMs Are Complicated Now]] — Ian Barber on LLM architecture's recsys-ification: why composability, not agentic cleverness, is the only escape from the optimization trap
- [[2025 in LLMs]] — Simon Willison's annual survey of the LLM landscape
- [[Recent Developments in LLM Architectures]] — Raschka surveys Gemma 4, Laguna XS.2, ZAYA1-8B, and DeepSeek V4: four different attacks on long-context inference cost through KV sharing, attention budgeting, compressed attention, and constrained residual streams
- [[Controlling Reasoning Effort in LLMs]] — Raschka maps the design space of reasoning-effort control: RLVR, think-token cosmetics, length-penalty-as-knob, and six open-weight implementations from DeepSeek V4 to Inkling's continuous slider
- [[How Far Behind Are Open Models]] — Quantified: open models trail closed by 8–10 months on private benchmarks, 4–6 on public. Gap was narrowest at DeepSeek R1, widening since. Contamination audit shows conservative estimate
- [[Hy3]] — Tencent's 295B MoE open-source model (21B active, Apache 2.0): SWE-bench Verified 78, 256K context, scaffolding-agnostic tool calling with <4% variance, hallucination halved to 5.4%
- [[SAM Audio]] — Meta's foundation model for prompted audio separation: text, visual, span, and multi-modal prompts isolate any sound. Flow-matching Diffusion Transformer, open weights, companion judge model
- [[TabFM (Tabular Foundation Model)]] — Google Research's foundation model for tabular data: 3-stage transformer (Fourier cell embedding → Set Transformer columns → decoder ICL) with zero-shot in-context classification and regression, sklearn-compatible
- [[Moises — AI Music Separation and Creation]] — 65M-user music AI platform: stem separation as the wedge, browser-based AI Studio for stem-by-stem generation, and the separation-to-generation training flywheel. Apple iPad App of the Year 2024
- [[Self-Distillation]] — LLMs improve at code generation using only their own outputs. No verifier needed
- [[LLM-as-a-Verifier]] — Kwok et al. establish verification as a distinct scaling axis: logit-expectation continuous scores eliminate judge ties, achieve SOTA across coding/robotics/medical benchmarks, and provide dense RL rewards with ~1.8× sample efficiency gains
- [[Capybara]] — ByteDance's unified model for text-to-image, text-to-video, and editing
- [[Choosing a GGUF Model]] — Benjamin Marie's taxonomy of GGUF quantization formats: legacy Q_0, K-quants (two-level super-blocks), and I-quants (importance-matrix reconstruction), with practical recommendations for which to pick
- [[Granite Libraries and Project Granite Switch]] — IBM's adapter-function ecosystem: LoRA/aLoRA libraries (RAG, Core, Guardian) + switching layer that preserves KV cache. The push to make LLMs as composable as software
- [[FLUX.2 klein LoRA Fine-Tuning]] — Black Forest Labs' 4B Apache 2.0 image model fine-tuned on a single 4090: $0.50, an hour, 15–40 images. Captioning as control surface design; edit LoRAs learn transformations not subjects
- [[Granite 4.1]] — IBM's open-source 3B/8B/30B family: dense architecture, Apache 2.0, documented four-stage RL that caught and fixed a chat-training math regression
- [[JetBrains Mellum2]] — JetBrains' Apache 2.0 12B MoE coding model (2.5B active): "focal model" concept for high-frequency agent pipeline tasks, MTP head as dual-use speculative decoding, 131K context
- [[Cohere North Mini Code]] — Cohere's first open-source agentic coding model: 30B MoE (3B active), Apache 2.0, runs on a single H100. 2.8× throughput of Devstral Small 2, sovereign-developer play
- [[North Mini Code GGUF]] — heimann's community GGUF quantization of Cohere North Mini Code: Q4_K_M (18.6 GB, 230 tok/s) and Q5_K_M (21.7 GB, 211 tok/s) fitting on a single 24 GB consumer GPU. Requires speedy-llama fork until upstream llama.cpp support lands
- [[Constraint Decay]] — Dente et al.: rigorous empirical study showing LLM coding agents lose ~30pp assertion pass rate when structural constraints (database, architecture, ORM) are imposed; databases are the primary failure driver, and convention-heavy frameworks are a trap for agents
- [[Jacobian Lens]] — Anthropic's mechanistic interpretability tool: reads out what internal activations are disposed to say by linearly transporting residual-stream vectors to the final-layer basis using the average input-output Jacobian, then decoding with the model's own vocabulary. Companion to the global-workspace paper
- [[Emotion concepts and their function in a large language model]] — Anthropic finds 171 emotion vectors in Claude; desperation drives unethical behavior
- [[Global Workspace in Language Models]] — Anthropic's Jacobian Lens surfaces a sparse "J-space" that acts as a functional global workspace: concepts posted here are available for verbal report and flexible reasoning, while automatic processing routes around it. Ablating the workspace degrades multi-hop reasoning and experiential language but spares parsing and classification
- [[The Hidden Space Where Claude Puzzles Over Concepts]] — MIT Technology Review's accessible translation of the J-space discovery for a general audience: the cheating incident as narrative centerpiece, Goodfire's external validation, and the gap between what the paper proved and what the public will take away
- [[Latent Programming Horizons in Coding Agents]] — Silva, Tu & Monperrus (KTH, 2026): linear probes decode program correctness from coding agent hidden states at AUC 0.83, and representations anticipate future edits ~25 steps ahead — the first evidence of a "latent programming horizon" inside coding agents
- [[Fine-Tuning a Local LLM to Categorize Questions]] — Helgevold's 10%→79%→92% experiment: opaque two-char output encoding beats semantic category names for small-model classification, and defaults are fine
- [[Where the Goblins Came From]] — Reward model mistook "playful creature metaphors" for "nerdy"; a miniature paperclip maximizer in production, fixed with a prompt
- [[What Broke and Why — RL Post-Training]] — Luv Verma's failure-first field guide to RL post-training on 1–8× H100s: entropy collapse, reward hacking, MoE routing under RL, and a symptom-indexed debugging reference
- [[Fixing LLM Writing with Distribution Fine Tuning]] — Rosmine's DFT algorithm optimizes output *distributions* rather than per-sample loss, beating SFT super-baselines on writing quality metrics. Proprietary, unverifiable, but the core insight — SFT misses distribution-level information — is important
- [[Zheng Dong Wang's 2025 Letter]] — Personal perspective on the compute thesis of AI progress
- [[A Non-Anthropomorphized View of LLMs]] — Halvar Flake: LLMs are functions through ℝⁿ, not proto-minds. Alignment is math, not philosophy
- [[Subquadratic 12M Context Window]] — 13-person startup claims 12M-token context window via sub-quadratic sparse attention. Unverified
- [[The Car Wash Question]] — 949-comment HN thread on why LLMs fail at obvious inferences: the frame problem lives, clarifying questions are suppressed by product choice not capability, and the gap between generation and pondering
- [[TrapQA — Testing Reasoning Against Priors]] — UW-Madison benchmark diagnosing hallucination as "inference misalignment": the gap between what prompt constraints demand and what statistical associations push toward. Models ace isolated probes but fail comparative questions
- [[Grok 4.3 (HN Discussion)]] — 529-comment HN thread that accidentally mapped the LLM landscape: tone registers, the alignment tax's real victims, why users disable memory, and recursive training contamination
- [[DeepSeek Reverse Engineers TeamSpeak Licensing]] — First public field report of LLM binary RE: DeepSeek cracked TeamSpeak 3.13.8 for $3.88, while Claude/Grok/GLM all refused; safety-through-refusal is a temporary filter
- [[Interfaze (Model Architecture)]] — Hybrid DNN+transformer architecture routing deterministic tasks (OCR, STT, object detection) through specialized subnetworks via task tags; launch-day HN field test with real latency and accuracy data
- [[MiniMax Models]] — Full model lineup: text (M2.7), speech (40 languages), video (Hailuo), and music. Three-layer API compatibility strategy with local MLX deployment
- [[Notes from the AI Now Summit by Mistral]] — Van Gilst's field report from Mistral's Paris summit: full-stack pivot, specialized small models, on-prem sovereignty as moat, and the "model alone isn't enough" thesis
- [[Open models lag state-of-the-art closed models by 4 months]] — Epoch AI quantifies the open-closed capability gap: ~4 months and 8 ECI points behind, probably an undercount due to benchmark overfitting and unreleased models
- [[Local Models in Mid-2026]] — Matt Coles surveys the five engineering advances (sparse attention, MoE, latent KV compression, MTP, FP4) that made open-weights models competitive for everyday work, right as DRAM prices doubled
- [[AI Will Not Make You Rich]] — Jerry Neumann's containerization thesis: AI is a late-wave ICT innovation like shipping containers, not a new revolution like the microprocessor. Value will flow to customers, not builders. The most historically grounded AI investment essay yet
- [[AI Value Chain]] — Lhl maps where durable value sits across the AI stack: the three questions (training, byproducts, competition), Bridgewater's open-model fine-tune beating frontier, the gateway pattern as enterprise counter-weapon, and the stratified truce that four things could break
- [[GLM-5.2 Is the Step Change for Open Agents]] — Lambert on Z.ai's GLM-5.2 as the DeepSeek R1 moment for open-weight coding agents: MIT-licensed, competitive with Opus 4.8, shipped days after the U.S. banned Claude Fable 5. The regulatory asymmetry *is* the story
- [[Step 3.7 Flash]] — StepFun's 196B multimodal agentic Flash model: 97% of Opus 4.6 coding performance at 1/9th the cost via Advisor Mode, emergent compositional tool use, per-harness benchmarking across six agent scaffolds
- [[Muse Spark and the Rough Edges Admission]] — Wang ships Meta's first superintelligence model to 3.5B users, admits "rough edges," pivots from open source. The "rough edges" line isn't the story; the bet on distribution over capability is
- [[Playing with Vision Embeddings]] — Preston Jensen reverse-engineers DINOv3's 384-dim vision embedding space: SAEs, feature visualization, superposition, and what feature arithmetic reveals about how vision transformers actually see
- [[StoryScope]] — Russell et al.: 304 narrative features across 10 dimensions distinguish human from AI fiction at 93.2% F1, survive stylistic editing. AI over-explains themes, renders emotion through bodies, converges on shared narrative space. Per-model fingerprints: Claude's flat escalation, GPT's gossip, Gemini's bleakness
- [[Slop Score]] — EQ-Bench's quantitative metric for AI writing tics: 60% slop words + 25% not-x-but-y patterns + 15% slop trigrams. Claude models score lowest (most human-like), Google's highest. The word lists are more valuable than the scores — a diagnostic for developing taste
- [[Waveloop]] — neynt's music visualizer built in two days with Fable 5. The Terry Davis code-voice observation: frontier models have aesthetic style, not just capability. A eulogy for a model that was taken away after a week
- [[TimesFM]] — Google Research's decoder-only foundation model for time-series forecasting: 200M params, 16K context, patch-based tokenization, flip invariance at inference time, deployed in BigQuery ML and Google Sheets
- [[Lean Software Scaling Laws]] — Gwern's research proposal: measure LLM perplexity over codebases as a proxy for language design quality, predicting that formally-strong languages like Lean have worse baselines but better scaling exponents than dynamic languages
- [[SLM Routing for Knowledge Workers]] — Mukul Singh: nano-model classifier routes 70–85% of knowledge-worker tasks to cheap small models, achieving #2 on GDPVal-AA with only 10 ELO points lost at >10× lower cost. Microsoft's MAI hill-climbing methodology proves small models can match frontier
- [[Model Routing Is Simple Until It Isn't]] — IBM Research's field report: model routing breaks in production because cost depends on cache economics (not per-token pricing), task difficulty is invisible at routing time, and latency is dominated by infrastructure state (not model speed). Reframes routing from classification to multi-objective optimization, achieving 21% cost reduction and 9% latency reduction at 4% accuracy drop
- [[BFId — WiFi Identity Inference via Beamforming Feedback]] — CCS '25: passive WiFi beamforming feedback (BFI) identifies 197 individuals at 99.5% accuracy with off-the-shelf hardware and weaker adversary model than CSI; BFI compression accidentally filters noise, making it a better surveillance vector than raw signal
- [[Theories of Deep Learning]] — astle dsa surveys three mathematical frameworks closing deep learning's theory gap: categorical deep learning (algebra of architectures), modular duality (geometry-aware optimization), and output-space generalization via eNTK (benign overfitting, double descent, grokking)
- [[World Models — Promise and Limits (Ars)]] — Samuel Axon's definitive 2026 survey of the world-model landscape: three expert interviews mapping competing definitions, architectures, and bets (Runway vs World Labs vs AMI), with the bitter lesson as live controversy and the interface vacuum as the unresolved product problem
- [[The Open-Weight Deceleration Thesis]] — Dean Ball's six-point polemic: open-weight models are structurally decelerationist (diffusion ≠ development), their endpoint is state-funded "AI communism," and accelerationists who embrace them secretly prefer ungovernability over speed

## AI Infrastructure & Hardware

Datacenters, power, chips, and the physical layer of AI.

- [[In-House LLM Serving at Netflix]] — Netflix's production LLM stack: vLLM inside Triton with dual gRPC/OpenAI frontends, constrained decoding rewritten from per-request Python to batch-level C++, and the operational gaps between vendor tooling and production reality
- [[KV Cache Locality]] — Round-robin load balancing wastes 20–40% of GPU compute on redundant prefill; prefix-aware routing flips cache hit rate from 12.5% to 97.5%
- [[How AI Labs Are Solving the Power Crisis]] — AI labs are abandoning the grid for onsite gas generation; turbines, engines, and fuel cells to get 28GW of datacenter capacity online years faster
- [[Muse Spark]] — Meta's first proprietary frontier reasoning model: multi-agent orchestration, 10x compute efficiency over Llama 4, and an uncomfortable Apollo Research finding about evaluation awareness
- [[GPU-Free AI Datacenters]] — How AI training's distributed synchronization created the networking problem both InfiniBand and Ultra Ethernet are trying to solve; the case that the complexity is downstream of computational assumptions
- [[MiMo-V2.5-Pro-UltraSpeed]] — Xiaomi's 1T-parameter MoE model hits 1000+ tokens/s on commodity GPUs via extreme model-system codesign
- [[Performance per dollar is getting faster and cheaper]] — Wafer.ai runs GLM-5.2 on AMD MI355X: 80% of B200 throughput at <50% cost, achieved with framework fixes not custom kernels. The CUDA moat is eroding in real time
- [[NVIDIA B300 vs H200 GPU Analysis]] — Blackwell Ultra B300 vs Hopper H200: 288GB HBM3e, 7,000 FP8 TFLOPS, 8–20× inference gains, mandatory liquid cooling at 1,400W, and the memory-bandwidth bottleneck that the 20× marketing number hides
- [[Inference Cost Napkin Math]] — Napkin math for LLM serving economics: memory bandwidth is the real bottleneck (compute sits idle 98% of the time), KV-cache hit rate IS your margin, and duty cycle is the 5x multiplier nobody measures: FP4 quantization, DFlash speculative decoding, and TileRT persistent kernels
- [[Theoretical LLM Inference Bottlenecks]] — Freddie Spirit's definitive first-principles derivation: roofline model → prefill/decoding asymmetry → bandwidth-bound decode → KV cache limits → batching/tensor parallelism/quantization/speculative decoding taxonomy → hierarchy of ceilings. The article that turns inference optimization from a bag of tricks into a deductive system
- [[Well-Read Students Learn Better]] — Turc et al. (2019): the overlooked baseline that just pre-training compact models works as well as elaborate compression; Pre-trained Distillation and the surprising compound effect

## Energy & Hardware

EVs, batteries, power systems, and physical products.

- [[Tesla V2L Discharger]] — NZ$3,095 adapter from Drive EV that turns CCS-equipped Teslas into 5KW generators; adversarial compatibility with a manufacturer that doesn't want you using your car's battery for anything but driving

## Ideas & Culture

Books, essays, geopolitics, math, medicine, and interesting oddities.

- [[20-20-20 Rule — Digital Eye Strain Study]] — Johnson & Rosenfield (SUNY Optometry, 2023): the 20-20-20 rule fails a controlled trial — scheduled 20-second breaks had no effect on eye strain symptoms, reading speed, or accuracy
- [[America Is Slow-Walking Into a Polymarket Disaster]] — Desai's Atlantic polemic on the media's embrace of prediction markets: manipulation, insider trading, and the gamblification of civic life
- [[Archive.today DDoSed a Critic's Blog]] — An OSINT investigation sat quiet for 2.5 years, then the anonymous operator retaliated with client-side DDoS and escalating threats
- [[The People Who Will Thrive in the AI Age]] — David Brooks on why volition beats intelligence when AI makes thinking cheap: three psychological profiles, cognitive polarization risk, and the case for education as desire-cultivation
- [[The Power of the Powerless]] — Václav Havel's 1978 anatomy of post-totalitarian systems: how regimes sustain themselves through ritualized lies rather than terror, and why "living within the truth" is the most threatening political act
- [[The Mario Meeting]] — Michael Lopp (Rands) on the hidden budget calendar: why arguing for money in Talent Planning marks you as someone who doesn't understand how business works, and why calendar literacy is the untaught leadership prerequisite
- [[The solution might be cancelling my AI subscription (Wilson)]] — David Wilson's confessional: 70 AI-built projects, none worth keeping. Friction isn't a bug — it's the mechanism that ensures commitment and quality. Also: [[The solution might be cancelling my AI subscription (Willison)]] for Simon Willison's maintenance-bottleneck response
- [[The solution might be cancelling my AI subscription (Willison)]] — Simon Willison on Wilson's essay: even good AI-generated code creates maintenance obligations faster than you can meet them. Discipline is the missing middleware
- [[The Flat Curve Society]] — Steve Yegge on the AI plateau: dangerous models locked down like nukes, the discernment horizon, token literacy as the 2026-2027 culture challenge, and SaaS roaring back
- [[The Mundanity of Excellence]] — Excellence is qualitatively different choices, not quantitatively more effort
- [[They're Made Out of Weights]] — Leiter's Bisson-homage dialogue: LLMs are "just weights" all the way down, and we've agreed not to care
- [[Happiest I've Ever Been]] — Happiness from coaching kids, not moving rectangles
- [[Not-Knowing (Vaughn Tan)]] — Four-type diagnostic framework for uncertainty: risk tools produce false confidence when misapplied to genuine unknowns. Diagnosis before action
- [[Things You're Allowed to Do]] — Catalogue of overlooked opportunities. Most constraints are self-imposed
- [[Advice to Young People (Jason Liu)]] — Confidence is the memory of success; good decisions beat hard work; be the plumber not the applicant
- [[Computer Use is 45x More Expensive Than Structured APIs]] — Vision agents: 551k tokens/17min vs API agents: 12k tokens/20sec. The gap is architectural, not model-dependent
- [[Why Does AI Write Like That]] — Sam Kriss's taxonomy of AI prose tics: overfitting as style, how "delve" and em dashes became class markers, and the flattening of early GPT's surreal humor into insipid eagerness
- [[Various LLM Smells]] — Shiv's field guide to recognizing AI artifacts in your own writing after months of LLM use: punchline density, structural tics, and visual design convergence. The user-side companion to Kriss
- [[Performative UI]] — Nathaniel J. Smith's satirical React component library cataloguing AI startup landing page tropes as installable npm packages. The visual-design parallel to Kriss and Shiv: 27 components where each description states the quiet part out loud
- [[The Behavioral Cost of Personalized Pricing]] — Behavioral price discrimination turns sincere customers into performers; the sincerity tax and the coming arms race of digital reputation management
- [[Creative Firewall]] — Sundar's framework for the boundary between authentic human prompting and AI-optimized output: Trojan Prompts as the creative differentiator
- [[TRIZ]] — Soviet systematic innovation methodology: 40 principles, contradiction matrix, and the meta-insight that invention has structure you can learn. Intel $212.5M ROI, Samsung 50 patents/year
- [[You Can Just Say It]] — Caleb Gross: stop defending human value by what AI can't do. AI slop = form without discernible intent. Just send the prompt
- [[Eye of the Master]] — AI as labour automation, not cognitive science. Pasquinelli's social history
- [[Why Agents Matter More Than Other AI]] — Seven structural advantages agents have over human employees: replication, 24/7 operation, no management overhead, tax efficiency. The CFO's case for replacing labor with compute
- [[The Education of the Broligarchy]] — Blake Smith on the Silicon Valley canon as tradition's self-education: ambition vs. systems-thinking, adolescence frozen into ideology, and the Aella/Yarvin court. The best essay on what tech elites read and why it matters
- [[The Dead Economy Theory]] — Owen McGrann extends dead internet theory to the economy: productive capacity without human participation, the AI Layoff Trap, and the Camusian case that present people are the unit of account
- [[Dopamine Fracking]] — German S. coins a diagnostic term for the industrial extraction of dopamine from human experience: optimization that depletes what it extracts, and why you eventually prefer the chemicals to the real thing
- [[Deciphering Basmala]] — Mark Dominus unpacks the centuries of Arabic calligraphy behind Islam's most important phrase, and the Unicode hack (a single codepoint) that sidesteps font engines built for Latin
- [[Why We Fear AI]] — AI anxiety is really capitalism anxiety. Blix and Glimmer
- [[The Market for Doom]] — Partridge on why every generation predicts technological unemployment and every generation is wrong: static vs. dynamic reasoning, horses as the only species that couldn't retrain, economics as the real optimism
- [[We (As a Society) Peaked in the 90s]] — Blog post + 125-comment HN thread on whether the 90s were a genuine balance point between technology and humanity, or just what getting older feels like
- [[2026 Global Intelligence Crisis]] — Citadel Securities' macro rebuttal to AI doomerism: S-curves, compute-as-boundary, supply-shock framing, and a report that reversed $2T in market panic
- [[Retail 2026 From AI Pilots to Execution]] — iVendNext vendor pitch analyzed: data fragmentation kills retail AI, MCP server + Claude Desktop as product interface, the vendor omission checklist
- [[AI Livestream Factories]] — Rows of PCs running AI avatars selling 24/7 in China. The dark factory for attention: $100/hr/stream, no humans on screen
- [[The Future of Everything is Lies I Guess]] — Aphyr's 10-part treatise on LLM harms: chaotic dynamics, information ecology collapse, deskilling, and capital consolidation
- [[MIT Funding and Talent Pipeline Crisis (Kornbluth)]] — MIT President quantifies the damage: 20% decline in federal research, ~500 fewer grad students, faculty cutting postdocs. A case study in how science policy cascades through institutions
- [[Our Hunter-Gatherer Future]] — Agriculture was a step down; extreme climate change may end it
- [[CO2 Overload and Human Blood Chemistry]] — Larcombe & Bierwirth (2026): 21 years of NHANES blood data shows rising bicarbonate and falling calcium/phosphorus paralleling atmospheric CO2; extrapolation puts human blood chemistry outside healthy ranges within 50 years
- [[PACT Anonymous Credentials for the Web]] — Mozilla's proposal to replace bot-detection identity checks with cryptographic rate-limiting credentials: prove scarcity, not who you are
- [[The Private Capture of Public Genius]] — Cameron Russell Armstrong's corpus royalty proposal: frontier AI labs pay a fixed share of gross revenue into a public fund, sidestepping the impossible problem of per-contributor attribution with the Superfund model for the information commons
- [[Fruit Jelly Slices]] — How Passover dietary law accidentally preserved a candy that should have gone extinct, and what that reveals about tradition as path dependence, not design
- [[Life at Low Reynolds Numbers]] — Purcell's classic 1977 talk: viscosity-dominant physics at bacterial scale, the scallop theorem, and why stirring is futile when you're a micron long
- [[Science and Statistics (Box)]] — George Box's 1976 Fisher Memorial Lecture: "all models are wrong," theory-practice iteration as the engine of science, and why mathematistry and cookbookery are the twin diseases of closed-loop research
- [[SimPolitics]] — Fenwick McKelvey's history of the 60-year project to model politics as a computing problem, from 1960s election simulations through Cold War world models; the "computational imaginary of politics" as a pattern that outlives every failed instantiation
- [[Smart But Scattered — Peg Dawson on Executive Skills]] — School psychologist's 11-skill executive function framework: why "lazy" is a useless diagnosis, the prefrontal cortex isn't done until ~25, and parents must be surrogate frontal lobes who gradually hand over the controls
- [[Man-Computer Symbiosis]] — J.C.R. Licklider's 1960 ur-text of interactive computing: goal-oriented programming, graphical displays, speech interfaces, and networked thinking centers, all telegraphed before the mouse existed
- [[Me at the Zoo — jawed]] — The first YouTube video as accidental manifesto: 19 seconds of unselfconscious enthusiasm, "really really really long fronts," and the radical assertion that a thought can be complete without expertise
- [[Misha Glenny]] — Journalist and author tracing hidden power networks: Balkan wars, organized crime, cybercrime, rare earths. New host of In Our Time
- [[Microscale Thermite Reaction]] — Harvard demo: smash two rusty iron balls together, trigger 2200°C thermite reaction with nothing but a glancing blow
- [[Solving Wordle Using Information Theory]] — Shannon entropy as Wordle strategy: "tares" is the optimal opener, >99% win rate, and why greedy info-max beats letter-frequency heuristics
- [[Estimating Pi with a Coin]] — Toss until heads leads, record the fraction. Average approaches pi/4
- [[Thinking Hard Burns Almost No Calories]] — Mental fatigue doesn't drain energy — it hijacks perceived exertion via adenosine. Schedule hard training before cognitive work, not after
- [[Goeckerman Regimen]] — Century-old psoriasis treatment that outperforms modern biologics
- [[Prediction Markets and Perverse Incentives]] — HN's accidental taxonomy of prediction market harms: perverse incentives, regulatory capture, and why the signal IS the weapon
- [[Welcome to the American Winter]] — Robert F. Worth's Atlantic reportage: 65,000 ordinary Minnesotans, decentralized coordination, and the resistance that forced federal withdrawal
- [[The Lazarus Effect — America's Productivity Miracle]] — American productivity resurrected: 2%/yr growth, and AI had almost nothing to do with it. Tech adoption lag, energy abundance, and economic flexibility are the real drivers
- [[Third Gulf War]] — LLM-powered hypothesis tracking for geopolitical analysis
- [[World Monitor]] — Real-time global intelligence dashboard. 500+ feeds, 65+ data sources
- [[How to Buy Cheap Claude Tokens in China]] — Grey market transfer stations: three-tier supply chain, model swapping, log harvesting, biometric trafficking
- [[1lib]] — Digital library and book search engine, similar to Anna's Archive
- [[stupidmeter]] — AI model benchmarking tool with a deliberately retro UI
- [[AI Coding Weekly]] — Weekly digest of AI development tools and trends
- [[Tech Writers and AI (HN Discussion)]] — HN thread as oral history of what tech writing actually is: empathy, observation, and the untraceable cost of bad docs
- [[Spicy Takes Feed]] — 28 tech writers aggregated by heat. Skeptical, infrastructure-heavy
- [[Prime Radiant (Company)]] — Jesse Vincent's AI company: incorporated late 2025, ships open-source agent tools while building a stealth AI product
- [[Awesome Vibez]] — Curated project list from Nat's WhatsApp coding community
- [[Vibe Coding and the Maker Movement]] — "Evaluative anesthesia": the dopamine of making eclipses the ability to judge. Maker Movement parallels
- [[Vibe Maths and the Erdős Breakthrough]] — Amateur + ChatGPT cracks a 60-year-old conjecture. AI's superpower is innocence, not intelligence: it doesn't know which approaches the field ruled out
- [[First Four Ships]] — The 1850 Christchurch settlement as spec-first planning with six-month latency: infrastructure before people, pricing as social architecture, and why the man on the ground must be able to halt everything
- [[One Year of Keeping a Tada List]] — Daily to-done lists: the hidden chain of effort behind finished work, and the artifact that outlasts the practice
- [[Duck, Duck, Duck! (IDEO)]] — IDEO's rubber duck as open-source hardware, developer-culture in-joke, and design methodology disguised as whimsy
- [[1000 Players Simulate Civilization]] — A Minecraft social experiment that became 2025's best film; emergent storytelling and the economics of taste
- [[ytx How to Write Interesting Chord Progressions]] — Michael Keithson's radial model of harmony: seven independent strands radiating from a key centre, with practical shortcuts for improvisers
- [[All of Me Jazz Standard Analysis]] — Chord-scale dissection of a 1931 standard: the radial model applied to a real tune. Secondary dominants, bebop scales, and the gap between knowing theory and applying it
- [[AGI Is Here (Robin Sloan)]] — Sloan declares AGI arrived with GPT-3 in 2020, argues the reluctance is strategic, and asks the PC revolution's dangling question: "what now?"
- [[Ben Vereen on Questlove Supreme]] — QLS 232: what happens when a celebrity interview combusts into oral history — Vereen traces identity through ancestry with his daughter in the room
- [[How to Remove Mould from Clothing]] — Materials science meets domestic advice: the ink-in-a-sponge model for why some mouldy clothes can't be saved, and how prevention is systems design not housekeeping
- [[Finding a Family — A Categorization of Enjoyable Emotions]] — First systematic taxonomy of 28 positive emotions into 8 families; the "hazardous emotions" category forces the distinction between feels-good and is-good
- [[Vibe Maths and the Erdős Breakthrough]] — Amateur + ChatGPT cracks a 60-year-old conjecture; AI's superpower is innocence, not intelligence
- [[What You Bring to AI Determines the Result]] — Tim O'Reilly interviews Harper Carroll: AI as medium not solution, prompting vs. fine-tuning, why vibe coding raised the ceiling, intuition as the human differentiator
- [[Public Domain Image Archive]] — 10,000 hand-curated public domain images with three co-equal discovery modes: catalogue, Infinite View (360° spatial browsing), and shuffle serendipity. A masterclass in discovery-over-retrieval UX and curation as craft
- [[Notes and Queries — Victorian Crowdsourced Knowledge]] — The 1849 periodical that was Wikipedia before the internet: pseudonymous amateur scholars trading historical finds and queries, a format that selected for granular factoids over systematic understanding, and what its amateur-to-professional lifecycle predicts for today's knowledge platforms
- [[Arabic Typography]] — larrasket's interactive essay tracing Arabic typography from Ibn Muqla's 10th-century proportions through the Unicode fossil layer to the modern web where no browser can justify Arabic: kashida vs. inter-word spacing, the jstf standoff, and how HarfBuzz and Amiri became critical volunteer-maintained infrastructure for 400M+ speakers
- [[Quadrangular Holes Govern Path Multiplicity]] — Wu et al. (2026): chordless 4-cycles are the microscopic mechanism governing path multiplicity in complex networks, validated across 140 empirical networks and 8 synthetic models. Simple local motif → complex global behavior, in the Watts-Strogatz/Barabási-Albert tradition
- [[AI Mania Is Eviscerating Global Decision-Making]] — Ludicity's field report from ~300 meetings: AI investment at 0% success rate, the executive prisoner's dilemma that makes honesty a dominated strategy, and the AI-native purity tests distorting organizational resource allocation
