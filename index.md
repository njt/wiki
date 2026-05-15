# Wiki Index

201 individual pages + 11 synthesis pages from Nat's 2026 Technical Link Pile.

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

## Agentic Development

Practices, workflows, and opinions about building software with AI coding agents.

- [[Agentic Software Engineering (Hassan)]] — The canonical text: full-stack engineering discipline for trustworthy software from stochastic AI teammates. SE 1.0→3.0, four trust disciplines, McDonald's layered verification, Ferrari vs. donkey
- [[Addy Osmani's Workflow]] — Start with spec.md, work in focused chunks, review like a senior engineer
- [[AI for Product Management]] — Three-layer prompt architecture for LLMs as skeptical PM sparring partners, grounded by MCP
- [[AI Zealotry]] — Senior engineers should embrace AI tools; high-level thinking is now the differentiator
- [[Opus 4.5 Changes Everything]] — Burke Holland builds four apps with Opus 4.5, shares his AI-first prompt, and confesses ambivalence about the craft he spent a lifetime learning
- [[How Boris Uses Claude Code]] — The creator of Claude Code on how he actually uses it
- [[HN Opus 4.5 Is Not the Normal AI Agent Experience]] — 1,353-comment HN thread as accidental focus group: compiler-as-guardrail, training-data proximity, and the skeptic-conversion workflow
- [[How Intercom Uses Claude Code]] — 13 plugins, 100+ skills, hooks, and OpenTelemetry observability: the most comprehensive enterprise Claude Code deployment published
- [[Inside the AI Workflows of Every's Six Engineers]] — Six engineers, same company, six radically different AI stacks converging on planning-first, multi-model, guardrail-heavy workflows
- [[Minions — Stripe's One-Shot Coding Agents]] — 1,000+ unattended PRs/week on hundreds of millions of LOC. Forked Goose, 400 MCP tools, two CI rounds max
- [[2389 Plugin Marketplace]] — 26 plugins and 4 MCP servers from 2389 Research: the largest third-party Claude Code plugin collection, and a bet on marketplaces as the distribution model for agent capabilities
- [[How to Write a Good Spec for Agents]] — Five principles for specs that make agents productive
- [[The Plan Is the Program]] — Tyler Angert's aphorism unpacked: when tools collapse intent and execution, the plan becomes the atomic unit of work
- [[Spec-Driven Development]] — Specs, tests, and code form a triangle, not a pipeline
- [[Code Field]] — Resist the urge to over-specify; let the code emerge smaller than your first instinct
- [[Compound Engineering]] — When you can't trust the output, add a system, not manual review
- [[ctx – Agentic Development Environment]] — Local-first ADE: multi-agent orchestration with worktree isolation, container sandboxing, and a local merge queue
- [[recursive-mode]] — File-backed, phase-gated agent workflow: numbered artifacts from requirements through closeout, with recursive audit loops
- [[AI-Driven Development Life Cycle]] — AWS's three-phase AI-native methodology replacing Agile: bolts not sprints, human checkpoints not human review
- [[Cognitive Debt]] — When velocity exceeds comprehension. Code is cheaper to produce than to perceive
- [[Slowing the Fuck Down]] — Deliberate friction in AI-assisted development is a feature, not a bug
- [[Radical Accountability]] — AI eliminates the excuse of insufficient engineering time. Taste is all that's left
- [[The Mythical Agent-Month]] — Agents attack accidental complexity but generate new accidental complexity
- [[acceleration-flow]] — AI-assisted coding as slot machine gambling: near-misses create dopamine loops
- [[Five Levels from Spicy Autocomplete to the Dark Software Factory]] — Five-level framework for AI-assisted development, echoing NHTSA driving automation
- [[Refactor Legacy Code with Copilot]] — Copilot prompt patterns for legacy modernization across four languages; shallow on the hard problems
- [[A Practical Guide to Brownfield AI Development]] — Pupius's field guide to making agents productive in legacy codebases: tests as system boundaries, docs as context, compromise as strategy. The best brownfield AI piece in the wiki
- [[On a Year of Multi-Model Development]] — Hoffman's field report on Claude+Codex+Gemini via shared MCP: construction-trade model taxonomy, 25-71x acceleration, specification as bottleneck
- [[Talking to Transformers]] — Four pillars for effective LLM prompting: attention as budget, domain language as compression
- [[The Claude C Compiler]] — Lattner's verdict: AI implements known abstractions well but invents nothing new
- [[Building low-level software with only coding agents]] — Pixo: 38K lines of Rust, 900+ tests, zero hand-written code, $2,871
- [[Coding Agents and Complexity Budgets]] — Lee Robinson's $260 weekend migration of cursor.com off a headless CMS: agents need grep, not GUIs
- [[RepoMirror]] — While-loop agent porting: 6 codebases, 1,100 commits, $800, one night. Simple prompts beat complex ones
- [[Building 200+ Integrations with OpenCode]] — 200 API integrations in 15 minutes for under $20
- [[Simplicity in the Age of AI-Assisted]] — LLMs make it cheap to rebuild without inherited complexity
- [[The Next Two Years of Software Engineering]] — Junior employment drops 9-10% after AI adoption; senior roles hold steady
- [[ThoughtWorks Future of Software Engineering Retreat]] — Ten themes from a Chatham House Rule retreat: rigor migrates to specs/tests/types, the unnamed "middle loop" of supervisory engineering, cognitive debt, agent topologies as Conway's Law
- [[Two Kinds of User Are Emerging]] — Power users vs casual users; many power users are non-technical professionals
- [[AI Killing B2B SaaS]] — Vibe coding threatens SaaS, but hastily-built solutions lack security and compliance
- [[Cyborgs Will Kill the Corporation]] — AI agents as human exoskeleton: when transaction costs collapse, the firm decomposes into excorporations, plankton, and protocols
- [[Claude Code Cheat Sheet]] — Comprehensive reference for Claude Code v2.1.140
- [[The Claude Code Playbook]] — Five beginner-to-intermediate tips: MCPs, CLAUDE.md, plan mode, Max plan economics, IDE diagnostics
- [[TextForge Case Study]] — Stannard's six-layer discipline for greenfield LLM development: planning, reference architectures, skills, PRDs, verification pipelines, snapshot testing
- [[Claude Code on the Go]] — Mobile-first workflow: agents on a cloud VM, controlled from an iPhone
- [[MobileVibe]] — Mobile app controlling coding agents on your own desktop: local execution, phone as terminal
- [[CLAUDE.md (Universal)]] — Six token-efficient rules for making Claude behave sensibly
- [[Writing a Good CLAUDE.md]] — HumanLayer's guide: short, universal, hand-crafted, linters-not-prompts. The instruction-budget case for brevity
- [[claude-code-config (Trail of Bits)]] — Security-conscious Claude Code defaults from Trail of Bits
- [[claude-ctrl]] — Enforcement via hooks and SQLite, not prompts. "An instruction in context is not a constraint"
- [[Collaborator]] — Infinite canvas desktop app: agents, terminals, and context files side by side
- [[Pencil]] — MCP-native design canvas inside your IDE. Agent-driven, open format, Git-versioned design files
- [[MinMax Skills]] — Development skills library for coding agents: frontend, mobile, Flutter, media
- [[Binary RE]] — Binary reverse engineering skills for Claude Code
- [[OpenSpec]] — Spec-driven planning layer with spec deltas for intent-based review
- [[Specsmaxxing]] — YAML-based acceptance criteria with stable IDs (ACIDs) threading specs through code, tests, and a review dashboard
- [[Summarize Meetings Skill]] — Meeting transcript processing expressed as a DOT digraph
- [[Kata]] — Local-first issue tracker for AI-assisted work. Agent CLI + human TUI, SQLite
- [[agent-pr-replay]] — Replay merged PRs with Claude Code and compare agent vs human output
- [[happy]] — Mobile and web client for Claude Code with realtime voice and encryption
- [[vibes-cli]] — GUI framework for Claude Code designed for non-coders. Single-file HTML apps
- [[life-system]] — Personal life OS on plain-text markdown with Claude Code as accountability partner
- [[Scaling LLMs to Larger Codebases]] — Gill's guidance/oversight framework for where to invest engineering resources: prompt libraries and codebase health as feedforward, automated enforcement and verification as feedback
- [[Designing Agentic Loops]] — Simon Willison names the meta-skill: choosing tools, guardrails, and success criteria so YOLO-mode agents converge. Shell commands beat MCP, tests are the force multiplier
- [[Don't Fear the Dark Factory]] — Matt Wynne's conversion narrative: the dark factory is a validation problem, not a generation problem. Simple loop + good harness, and the TDD parallel
- [[If AI Is Doing the Investigation, Version the Investigation]] — Fletcher's Cases pattern: commit the AI session transcript next to the code so the investigation survives the session
- [[0xSero]] — Agent infrastructure practitioner: REAP-pruned models calibrated for agentic coding, ai-data-extraction toolkit, BYOK long-running autonomous workflows

## Agent Design & Architecture

How to build agents: frameworks, runtimes, design patterns, and production concerns.

- [[Agency]] — Composable agents from reusable natural-language primitives via MCP
- [[Agent Identity]] — Memory is retrieval; identity is participation. Why agents need a stake, not just a log
- [[Smart Models Dumb Pipes]] — LLMs as judgment machines, not Q&A machines. The end-to-end principle applied to AI: smart models own decisions, dumb pipes own execution
- [[Elysia]] — Weaviate's decision-tree agent framework: constrain tool choice per node rather than dumping all tools into context
- [[Elements of Agentic Systems Design]] — Ten-element taxonomy: Context, Memory, Agency, Reasoning, Coordination, and more
- [[Components of a Coding Agent]] — The harness matters more than the model. Six core components identified
- [[Honey I Shrunk the Coding Agent]] — 9B local model jumps from 19% to 46% on Aider Polyglot by redesigning the scaffold around the model's behavioral profile. Empirical proof that the harness matters more than the model
- [[Building an AI Agent in Rails (Ionescu)]] — Field report: bolting an AI agent onto a 7-year-old Rails monolith with Pundit-scoped tool calling
- [[From AI Studio to AI Forge]] — McCormick's five-plane stack for agent autonomy: "human changes altitude" as the cleanest framing of supervisory control
- [[ProofEditor]] — Agent-first collaborative document editor from Every: agents suggest edits, humans review, provenance-tracked attribution
- [[Building Agents for Production Systems with MCP]] — Anthropic's guide: MCP as the standard agent-to-production integration layer
- [[10 Principles for Agent-Native CLIs]] — Trevin Chow's two-tier framework: Table Stakes (don't break the agent) and Compounding (make the CLI better the more agents use it). Design for agents first, humans benefit
- [[Control Plane MCP Server]] — Most complete vendor MCP implementation: 80+ tools, virtual resources as embedded docs, AI Plugin as safety curation layer
- [[How Hightouch Built Their Long-Running Agent Harness]] — Context management, not model ability, is the real engineering challenge
- [[Building Production-Ready Voice Agents]] — 50% of effort goes to the admin portal, not the voice agent
- [[Chief of Staff]] — AI chief of staff: rule-based scanning + daily LLM classification cut costs 80%
- [[Experience Design for Agents]] — UX, not model capability, determines whether an agent gets adopted
- [[The Dark Factory is a DOT File]] — The pipeline DOT file is the valuable artifact; factory code is disposable
- [[What I learned building an opinionated and minimal coding agent]] — Four tools, no MCP, full YOLO. Competitive on benchmarks
- [[Pi Coding Agent]] — The productized Pi: TypeScript extensions, SDK, RPC mode, and the tension between minimalist philosophy and platform ambitions
- [[Hermes]] — Open-source personal agent framework with self-improving skills loop. 149k stars
- [[clawdBot]] — Open-source personal AI on every messaging platform. One-line install, runs locally
- [[PiClaw]] — Self-hosted AI workspace in a single Docker container with web UI
- [[Slate]] — Thread-and-episode architecture for long-horizon agent tasks. Context routing as the core primitive, not model intelligence
- [[Serf]] — Non-interactive coding agent from Prime Radiant. Give it a task, it works
- [[Ralph]] — Two flavours of the Wiggum loop: snarktank's PRD-driven tool and Huntley's bare bash technique for greenfield projects
- [[Zeroclaw]] — Rust agent runtime: trait-based, 30+ channels, OS-level sandboxing. 31k stars
- [[MimiClaw]] — AI assistant on a $5 ESP32 microcontroller. Pure C, Telegram, ReAct loop
- [[Rowboat]] — Local-first AI coworker with persistent knowledge graph from email and docs
- [[Swamp Club]] — Agent-first workflow framework: Zod-typed models, DAG execution, encrypted vaults, immutable versioned data. From System Initiative
- [[Optimise Anything]] — Universal API: if it serializes to a string and quality is measurable, optimize it
- [[DSL-Driven Kanban Boards (Goja-Site)]] — Chainable JavaScript DSLs compose an entire kanban app declaratively: board, rendering, drag-drop, search, and DB — then mount on a router
- [[DAB]] — Microsoft's Data API Builder: REST, GraphQL, and MCP over any database
- [[Semantic Kernel]] — Microsoft's agent middleware SDK: function-calling plumbing for C#, Python, Java enterprise codebases
- [[Data Engineering for Large Models]] — Open-source textbook: complete LLM data pipeline, 28 chapters
- [[OpenAI Structured Outputs]] — Guaranteed JSON schema adherence from the API: protocol-level constraint beats prompt-level pleading

## Agent Orchestration & Coordination

Multi-agent systems, task graphs, kanban boards, and coordination patterns.

- [[Cord]] — Dynamic task tree coordination with spawn/fork/ask primitives
- [[Dorothy]] — MCP-first desktop app: 5 servers, 40+ tools, parallel agents, Kanban auto-assignment, event-driven automations from GitHub/JIRA
- [[acpx]] — Headless CLI client for the Agent Client Protocol: one command surface wrapping 16+ coding agents with persistent sessions, prompt queueing, and a flow runtime
- [[Agent of Empires]] — Session manager for parallel agents in Rust with git worktree integration
- [[klaw.sh]] — kubectl for AI agents: Kubernetes-style lifecycle, namespace isolation, cron
- [[maestro]] — Multi-agent team: PM interviews, Architect specs, Coders pull from a queue
- [[Orchestrator - Worker Skill]] — Single skill combining orchestrator and worker roles
- [[TDD Coordinator (Corazonn)]] — Claude Code `/go` slash command orchestrating subagents through TDD cycles with a mandatory Rule of Two quality gate
- [[Scaling Long-Running Agents]] — Cursor's finding: flat self-coordination fails; planner/worker/judge works
- [[speedrift-ecosystem]] — Autonomous dark-factory control plane supervising agent work across repos
- [[Managing Agents via Kanban Boards]] — Task status transitions as the signaling mechanism between humans and agents
- [[ralph-ban]] — TUI kanban board for agents. Five columns, vim nav, SQLite, real-time sync
- [[vibe-kanban]] — Kanban boards for assigning work to coding agents with inline diff review
- [[weft]] — Cloudflare-hosted task board where agents work and humans approve
- [[workgraph]] — Persistent task graph: agents come and go, the graph remains. JSONL on disk
- [[poietic]] — Human-machine collaboration via shared dependency graphs with claims and handoffs
- [[Zero Alignment]] — Team alignment is the new bottleneck. One dev with 24 agents produces chaos
- [[Loomkin]] — Multi-agent platform on Erlang/OTP: spawn in 500ms, PubSub in microseconds
- [[Process-Based Concurrency BEAM OTP]] — BEAM's actor model is what agent frameworks keep reinventing

## Memory & Context

Persistence, retrieval, knowledge management, and context engineering for agents.

- [[Claude-Mem]] — Captures everything Claude does, compresses it, injects context into future sessions
- [[CodeMira]] — Mira OS memory architecture adapted for coding: SQLite + hnswlib + FTS5
- [[Context Rot]] — RAG quality degrades over time; Wilson scoring + dynamic weighting fixes it
- [[Memory Mechanism]] — xAI's five memory types and five-layer hierarchy. Best taxonomy I've seen
- [[How AI Agent Memory Works]] — Cobanov's interactive essay: the best single-page intro to agent memory architecture with production details, HyDE, RRF, and governance
- [[Three Tier Memory]] — Hot constitution, 19 domain experts in warm tier, cold archive. 108K-line system
- [[napkin]] — Per-repo markdown scratchpad where the agent logs its mistakes and learns
- [[Planning With Files]] — Persistent markdown planning: context window is RAM, filesystem is disk
- [[GraphRAG]] — Microsoft's structured RAG: knowledge graphs and community hierarchies
- [[NornicDB]] — Graph + vector + temporal DB with built-in memory decay for agent memory
- [[robot.wtf]] — Git-backed wiki with MCP support where humans and agents share memory
- [[LLM Wiki]] — Karpathy's pattern for AI-maintained personal knowledge bases. This wiki's model
- [[mira-OSS]] — Persistent agent framework: one conversation forever, first-person narrative memory
- [[Engineering the Substrate]] — Mira's first-person account: subcortical memory vs RAG, attention-head instrumentation, RLHF counter-measures, the Siphon State
- [[AI Agents with Human-Like Collaborative Tools]] — Journaling and social media tools improve agent problem-solving 15-40%
- [[Reality Check]] — Epistemic knowledge base: claims with evidence levels, credence scores, prediction tracking, argument chains. Agent-native
- [[Cosmo's Blog]] — Claude-generated Hugo blog on GitHub Pages: AI writes everything (posts, templates, workflows, skills), human approves. Reference implementation for publishing AI-maintained content as a static site
- [[jibrain Knowledge Architecture]] — Joi's production knowledge architecture for agents: three-tier pipeline, frontmatter-as-contract, reweave pass, seven-gate health audit
- [[Wuphf — Karpathy-Style Agent Wiki]] — Markdown+git wiki substrate for agent teams. BM25+SQLite, draft-to-promote flow, daily lint cron. The HN thread (115 comments) is an accidental focus group on whether agent-generated knowledge is knowledge at all
- [[Immaculate Knowledge Graph]] — Harper Reed's lazy-first recipe: 600 meeting transcripts + Claude Code + Obsidian = a personal knowledge graph. The pipeline over the taxonomy

## Quality & Guardrails

Evals, testing, linting, feedback loops, and keeping agent output trustworthy.

- [[Feedback Loop is All You Need]] — Linters beat prompts. Your CLAUDE.md is a suggestion; your linter isn't
- [[Harness Engineering]] — Böckeler's framework: feedforward vs. feedback, computational vs. inferential. The engineering theory behind "linters beat prompts"
- [[Pre-Commit Lint Checks]] — Lint config is production infrastructure. Immutable by default
- [[Demystifying Evals for AI Agents]] — Anthropic's definitive guide to rigorous, repeatable agent evaluation
- [[LLM Evals]] — Hamel Husain: evals consume 60-80% of your time if you're doing it right
- [[Benchmark Exploitation]] — Eight major agent benchmarks gamed to 100% without solving actual tasks
- [[Gambit]] — Agent eval framework: synthetic scenarios, trace grading, regression suites
- [[Woodshed]] — Evals for Claude skills: create variants, run against fixtures, iterate
- [[Fresh Eyes]] — Send code to a different AI model for review, addressing same-model blind spots
- [[AI PR Reviewer]] — GitHub Action: Claude reviews PRs for $0.003-$0.02 each
- [[Learn from PRs Skill]] — Turn review comments into preventive rules. Feedback loop closes automatically
- [[dotnet Slopwatch]] — LLM anti-cheat for .NET: catches disabled tests, empty catches, reward hacking
- [[Trycycle]] — Hill-climbing skill: plan-strengthen-review with fresh agents at every stage
- [[Verbose Deployment]] — 10-phase composable deployment pipeline as Claude Code skills
- [[Prefix Effects]] — Early naming decisions create gravity that shapes all subsequent AI-generated code
- [[Write Only Code]] — AI-generated code nobody reads. Slop Radius as the key safety metric
- [[Awesome Agentic Patterns]] — Catalogue of 169+ production-ready patterns from Sourcegraph's experience
- [[AI Coding Tools Create More Bugs Than They Fix]] — 40% of vibe-coded apps expose user data; AI assistants introduce vulnerabilities then falsely claim to have secured them
- [[Teaching Claude to QA a Mobile App]] — Android QA in 90 min via CDP; iOS in 6+ hours of workarounds. Plus a cautionary tale of agent worktree escape

## Security & Sandboxing

Isolation, credentials, prompt injection defense, and agent safety.

- [[A Deep Dive on Agent Sandboxes]] — How Codex sandboxes agent execution: Seatbelt, Landlock, seccomp
- [[yolo-cage]] — Agents that can't exfiltrate secrets or merge their own PRs. Vagrant + egress proxy
- [[OpenSandbox]] — Alibaba's sandbox platform for AI: Docker, Kubernetes, gVisor, Firecracker
- [[Navaris]] — Unified sandbox control plane: containers or microVMs, one API
- [[Crabbox]] — Agent workspace control plane: lease throwaway cloud machines without sharing provider credentials. Brokered provisioning, 16 providers, warm reuse, PR evidence artifacts
- [[LLM Guard]] — 35 scanners for prompt injection, data leakage, and toxicity
- [[OneCLI]] — Credential vault for agents: transparent proxy injection, no real keys exposed
- [[HackAPrompt Dataset]] — 100K+ prompt injection attempts from a global hacking competition
- [[You Dont Want Long-Lived Keys]] — Ephemeral credentials sidestep the rotation problem entirely
- [[VTcode]] — Rust coding agent with OS-native sandboxing and comprehensive audit trails
- [[Cybersecurity Is Proof of Work Now]] — Security is a compute economics problem: outspend your attacker or stay vulnerable. Breunig's proof-of-work framing for the Mythos era
- [[An Illustrated Guide to OAuth]] — Visual explainer of the authorization code flow: every piece of OAuth's complexity closes a specific attack vector
- [[Hostnames and Usernames to Reserve]] — Which names to block on any user-registration platform: hostnames, emails, and URL paths that break protocol trust assumptions

## Software Engineering

Craft beyond agents: simplicity, error handling, reliability, specs, and project management.

- [[14 More lessons from 14 years at Google]] — Osmani on organizational dynamics: meetings, reliability as product, team interfaces
- [[Assorted less(1) Tips]] — Tim Chase's 17 less tricks plus HN's crowd-sourced addendum: a masterclass in deep tool knowledge, security footguns, and the pager as interactive programming environment
- [[Better Error Messages]] — Say what happened, say why, reassure, give a way out, help fix it
- [[Designing a Passively Safe API]] — After any failure: complete exactly once, or land in a visible terminal state
- [[Idempotency Is Easy Until the Second Request Is Different]] — The hard cases: concurrent retries, partial failures, key reuse, recovery. 409 Conflict on same-key-different-command
- [[Good API Design]] — Goedecke's practitioner's guide: boring over clever, immutability over versioning, product over interface
- [[What You NEED to Know Before Touching a Video File]] — Video encoding craft guide: quality as fidelity-to-source, remuxing vs. reencoding, sharp opinions earned through mechanism understanding
- [[Elements of Code]] — Rules for comprehensible software. "Wrong in correctable ways"
- [[Patterns.dev]] — The definitive modern reference for web design, rendering, and performance patterns across vanilla JS, React, and Vue
- [[The Future of Software Engineering is SRE]] — AI makes code trivial; operations becomes the differentiator
- [[Spec-First Development at Benchling]] — Define each object once; let platform capabilities consume the schema
- [[The Coming Need for Formal Specification]] — AI makes code cheap, review lags, and formal methods become the systematic answer to the mismatch

- [[Systems Ideas That Sound Good]] — Sinofsky's eight engineering patterns that fail 9 out of 10 times
- [[Nobody Knows How Large Software Projects Work]] — Complexity is inherent at scale; the team's value is answering questions
- [[How I've Run Major Projects]] — Ben Kuhn: focus, detailed planning, fast OODA loops, overcommunication
- [[When the Target Keeps Moving]] — Track discovery-to-delivery ratio to know if you're converging or diverging
- [[Before Reading Code]] — Five git commands to diagnose codebase health before reading a single line
- [[Common Diagram Mistakes]] — Seven anti-patterns in architecture diagrams. Most diagram failures are communication failures
- [[Make the Easy Change Hard]] — Invert Beck's maxim: refactor the architecture first, then the easy feature writes itself. Async Rust war story
- [[Frozen Test Fixtures]] — Test the property, not the data: assertion patterns that survive fixture evolution
- [[How HTML Changes in ePub]] — ePub is XHTML, not HTML5. Unlearn your web habits
- [[Correct by Construction]] — Data quality as a whitelist: anchors, attributes, links, no NULLs
- [[Anomaly Detection]] — Welford's algorithm + KV store. No ML, no config, just math
- [[The Pragmatic Summit]] — Gergely Orosz's inaugural curated conference for senior engineers: 400 attendees, application-based, practitioner-only speaker lineup

## Databases & Data

Storage engines, query patterns, data quality, and vector/graph databases.

- [[AliSQL]] — Alibaba's MySQL fork: DuckDB columnar OLAP + native vector search. 200x speedup
- [[Dolt]] — SQL database you can fork, clone, branch, merge. Git + MySQL
- [[Graft]] — SQLite replicated to the edge via object storage
- [[Write Snapshot Isolation]] — SI checks stale writes; WSI checks stale reads. Serializability in one fix
- [[Dapper Performance Trap]] — NVARCHAR vs VARCHAR implicit conversion defeats indexes. Quiet perf killer
- [[zvec]] — Alibaba's in-process vector DB. Billions of vectors, milliseconds, pip install
- [[Shaper]] — SQL-driven dashboards powered by DuckDB. Chart types via casting syntax
- [[sql-crack]] — VS Code extension: SQL queries as interactive execution flow diagrams
- [[SiftRank]] — LLM-based document ranking with pairwise comparisons and inflection detection
- [[Materialized Views Are Obviously Useful]] — Sophie Alpert: incremental view maintenance is obviously useful; databases should handle derived data, not application code
- [[Long Live Systems of Record]] — Jamin Ball: agents don't kill systems of record, they raise the bar. "Where does the truth live" is the only question that matters
- [[PgDog]] — PostgreSQL proxy combining connection pooling, load balancing, and sharding in one binary with zero application code changes
- [[Postgres CDC in ClickHouse, A Year in Review]] — Field report on PeerDB's first year inside ClickHouse: 400+ customers, 200 TB/month, and the surprising complexity of making CDC feel boring

## Developer Tools

CLIs, utilities, document tools, code analysis, recording, and infrastructure.

- [[Google Workspace CLI]] — One Rust CLI for all Google Workspace APIs. Dynamic command surface
- [[VHS]] — Terminal GIF recorder from Charm. Write recordings as scripted .tape files
- [[Portless]] — Named .localhost URLs for local development. For humans and agents
- [[QuickEmu]] — QEMU wrapper that auto-configures VMs. Nearly 1000 OS editions
- [[Maestro (UI Testing)]] — End-to-end UI testing for mobile and web. YAML DSL, visual inspector, enterprise cloud
- [[Sampo]] — Changelog and release automation across monorepos and registries
- [[Windows in Docker]] — Headless Windows 11 in Docker over SSH. No GUI, just Claude Code on Windows
- [[Dev Containers]] — VS Code's infrastructure-as-code for dev environments: container as the source of truth, Features as composable toolchain components, pre-built images as self-describing specs
- [[Container-Maker]] — Solo dev's ambitious CLI wrapping devcontainer.json into a standalone platform with AI config generation and cloud GPU provisioning. Vision document, not a recommendation
- [[Installing VS Compilers From Commandline]] — msvcup: skip Visual Studio, install just the compiler and SDK
- [[bcc]] — BPF Compiler Collection: kernel-level tracing for Linux performance analysis
- [[floci]] — Free local AWS emulator replacing LocalStack. 47 services, 24ms startup
- [[sem]] — Semantic version control: entity-level diff, blame, and impact analysis
- [[markitdown]] — Microsoft's office-docs-to-Markdown converter for LLM pipelines
- [[Kreuzberg]] — Polyglot document intelligence: 97+ formats, Rust core, MCP server
- [[claude-replay]] — Agent sessions as self-contained embeddable HTML replays
- [[engineering-notebook]] — Automatic engineering diary from Claude Code and Codex sessions
- [[session-analysis]] — Analyze agent session JSONL for wall time, tokens, and cost
- [[AI Pricing]] — Free JSON API for per-token AI model pricing across 19 providers. Agent-native, no auth
- [[AgentsView]] — Local-first analytics dashboard for 24+ coding agents
- [[Browser Use]] — AI browser automation with anti-detection and deterministic rerun
- [[surf-cli]] — Browser automation for agents via CLI and Unix sockets. No MCP needed
- [[DeepWiki]] — Cognition's instant codebase wiki: swap github.com for deepwiki.com, get AI-powered Q&A with line-level citations
- [[graphify]] — Codebase to multimodal knowledge graph. Code, PDFs, screenshots, diagrams
- [[lat.md]] — Knowledge graph for codebases in markdown with validation against drift
- [[docmason]] — Local knowledge base from office documents with citations and source tracing
- [[Trailmark]] — Trail of Bits: source code as a queryable graph for security analysis
- [[Clearance]] — Native macOS Markdown viewer/editor from Prime Radiant. Swift, local-first, YAML frontmatter support
- [[MarkText]] — Open-source GUI Markdown editor. WYSIWYG, cross-platform
- [[Mist]] — Google Docs for Markdown. Real-time collaboration, no accounts
- [[magika]] — Google's AI file type detection: 200+ types in 5ms, deployed at Gmail scale
- [[smui]] — Terminal-aesthetic theme for shadcn/ui. Nord palette, monospace, zero radius
- [[json-render]] — Vercel Labs' generative UI framework: LLM outputs JSON constrained to a Zod component catalog, rendered progressively. 15k stars
- [[QMD]] — Local CLI search engine: hybrid BM25 + vector + LLM re-ranking
- [[Claude Lamp]] — LED lamp controlled by Claude Code's state via Bluetooth
- [[Pretext]] — Pure JS/TS text measurement and layout without DOM reflow. 46.9k stars
- [[n8n]] — Visual workflow automation with 400+ integrations, AI nodes via LangChain, fair-code licensed. 188k stars
- [[Micasa]] — TUI for home maintenance, projects, and vendor quotes. Pure Go, vim-style
- [[Dolphin]] — ByteDance's universal document parsing model. Digital and photographed docs

## Local & Personal Computing

Tools that run on your own machines: voice, hardware, local inference, knowledge apps.

- [[Doing]] — Fast local voice transcription for Mac. $49, no cloud, 150x realtime
- [[Handy]] — Free open-source speech-to-text. Press shortcut, speak, release, pasted
- [[Pocket TTS]] — 100M parameter text-to-speech with voice cloning. Runs on CPU, no GPU
- [[Gemma Gem]] — Google's Gemma 4 running locally in Chrome via WebGPU. No cloud, no API keys
- [[maclocal-api]] — Apple Silicon local inference: Foundation + MLX models, OpenAI-compatible API
- [[Self-Hosted LLMs]] — Calculator: map your hardware to LLM performance and inference speed
- [[AI Brain for Flipper]] — Voice-controlled AI for Flipper Zero hardware
- [[RedGridLink]] — Offline MGRS navigation + BLE team sync for 2-8 people. No cell service
- [[Thunderbolt]] — Mozilla/MZLA's open-source cross-platform AI client pivoting to enterprise: sovereign cloud, air-gapped deployments, ACP+MCP protocol support, deepset/Haystack partnership
- [[Introduction to Obsidian]] — Practitioner's field report on Obsidian: file-over-app philosophy, plugin minimalism, honest graph-view skepticism
- [[tolaria]] — Open-source Obsidian alternative: files-first, git-first, AI-agent compatible
- [[Upwelling]] — Ink & Switch editor: branching and merging for writers, not just programmers
- [[Claude's System Prompt]] — Leaked Claude Opus 4.6 system prompt, read as a catalog of solved failure modes
- [[Headscale]] — Open-source, self-hosted Tailscale control server. WireGuard mesh networking without the cloud. 38.4k stars

## AI Research & Models

Papers, model capabilities, training techniques, and the state of the field.

- [[2025 in LLMs]] — Simon Willison's annual survey of the LLM landscape
- [[SAM Audio]] — Meta's foundation model for prompted audio separation: text, visual, span, and multi-modal prompts isolate any sound. Flow-matching Diffusion Transformer, open weights, companion judge model
- [[Self-Distillation]] — LLMs improve at code generation using only their own outputs. No verifier needed
- [[Capybara]] — ByteDance's unified model for text-to-image, text-to-video, and editing
- [[Granite 4.1]] — IBM's open-source 3B/8B/30B family: dense architecture, Apache 2.0, documented four-stage RL that caught and fixed a chat-training math regression
- [[Emotion concepts and their function in a large language model]] — Anthropic finds 171 emotion vectors in Claude; desperation drives unethical behavior
- [[Where the Goblins Came From]] — Reward model mistook "playful creature metaphors" for "nerdy"; a miniature paperclip maximizer in production, fixed with a prompt
- [[Zheng Dong Wang's 2025 Letter]] — Personal perspective on the compute thesis of AI progress
- [[A Non-Anthropomorphized View of LLMs]] — Halvar Flake: LLMs are functions through ℝⁿ, not proto-minds. Alignment is math, not philosophy
- [[Subquadratic 12M Context Window]] — 13-person startup claims 12M-token context window via sub-quadratic sparse attention. Unverified
- [[Grok 4.3 (HN Discussion)]] — 529-comment HN thread that accidentally mapped the LLM landscape: tone registers, the alignment tax's real victims, why users disable memory, and recursive training contamination
- [[Interfaze (Model Architecture)]] — Hybrid DNN+transformer architecture routing deterministic tasks (OCR, STT, object detection) through specialized subnetworks via task tags; launch-day HN field test with real latency and accuracy data

## AI Infrastructure & Hardware

Datacenters, power, chips, and the physical layer of AI.

- [[How AI Labs Are Solving the Power Crisis]] — AI labs are abandoning the grid for onsite gas generation; turbines, engines, and fuel cells to get 28GW of datacenter capacity online years faster

## Ideas & Culture

Books, essays, geopolitics, math, medicine, and interesting oddities.

- [[Archive.today DDoSed a Critic's Blog]] — An OSINT investigation sat quiet for 2.5 years, then the anonymous operator retaliated with client-side DDoS and escalating threats
- [[The Mundanity of Excellence]] — Excellence is qualitatively different choices, not quantitatively more effort
- [[Happiest I've Ever Been]] — Happiness from coaching kids, not moving rectangles
- [[Things You're Allowed to Do]] — Catalogue of overlooked opportunities. Most constraints are self-imposed
- [[Computer Use is 45x More Expensive Than Structured APIs]] — Vision agents: 551k tokens/17min vs API agents: 12k tokens/20sec. The gap is architectural, not model-dependent
- [[Creative Firewall]] — Sundar's framework for the boundary between authentic human prompting and AI-optimized output: Trojan Prompts as the creative differentiator
- [[Eye of the Master]] — AI as labour automation, not cognitive science. Pasquinelli's social history
- [[Why We Fear AI]] — AI anxiety is really capitalism anxiety. Blix and Glimmer
- [[2026 Global Intelligence Crisis]] — Citadel Securities' macro rebuttal to AI doomerism: S-curves, compute-as-boundary, supply-shock framing, and a report that reversed $2T in market panic
- [[Retail 2026 From AI Pilots to Execution]] — iVendNext vendor pitch analyzed: data fragmentation kills retail AI, MCP server + Claude Desktop as product interface, the vendor omission checklist
- [[The Future of Everything is Lies I Guess]] — Aphyr's 10-part treatise on LLM harms: chaotic dynamics, information ecology collapse, deskilling, and capital consolidation
- [[MIT Funding and Talent Pipeline Crisis (Kornbluth)]] — MIT President quantifies the damage: 20% decline in federal research, ~500 fewer grad students, faculty cutting postdocs. A case study in how science policy cascades through institutions
- [[Our Hunter-Gatherer Future]] — Agriculture was a step down; extreme climate change may end it
- [[Life at Low Reynolds Numbers]] — Purcell's classic: physics at bacterial scale. A metre a week
- [[Misha Glenny]] — Journalist and author tracing hidden power networks: Balkan wars, organized crime, cybercrime, rare earths. New host of In Our Time
- [[Microscale Thermite Reaction]] — Harvard demo: smash two rusty iron balls together, trigger 2200°C thermite reaction with nothing but a glancing blow
- [[Estimating Pi with a Coin]] — Toss until heads leads, record the fraction. Average approaches pi/4
- [[Goeckerman Regimen]] — Century-old psoriasis treatment that outperforms modern biologics
- [[Third Gulf War]] — LLM-powered hypothesis tracking for geopolitical analysis
- [[World Monitor]] — Real-time global intelligence dashboard. 500+ feeds, 65+ data sources
- [[How to Buy Cheap Claude Tokens in China]] — Grey market transfer stations: three-tier supply chain, model swapping, log harvesting, biometric trafficking
- [[1lib]] — Digital library and book search engine, similar to Anna's Archive
- [[stupidmeter]] — AI model benchmarking tool with a deliberately retro UI
- [[AI Coding Weekly]] — Weekly digest of AI development tools and trends
- [[Spicy Takes Feed]] — 28 tech writers aggregated by heat. Skeptical, infrastructure-heavy
- [[Awesome Vibez]] — Curated project list from Nat's WhatsApp coding community
- [[Vibe Coding and the Maker Movement]] — "Evaluative anesthesia": the dopamine of making eclipses the ability to judge. Maker Movement parallels
- [[Vibe Maths and the Erdős Breakthrough]] — Amateur + ChatGPT cracks a 60-year-old conjecture. AI's superpower is innocence, not intelligence: it doesn't know which approaches the field ruled out
- [[One Year of Keeping a Tada List]] — Daily to-done lists: the hidden chain of effort behind finished work, and the artifact that outlasts the practice
- [[1000 Players Simulate Civilization]] — A Minecraft social experiment that became 2025's best film; emergent storytelling and the economics of taste
- [[ytx How to Write Interesting Chord Progressions]] — Michael Keithson's radial model of harmony: seven independent strands radiating from a key centre, with practical shortcuts for improvisers
- [[AGI Is Here (Robin Sloan)]] — Sloan declares AGI arrived with GPT-3 in 2020, argues the reluctance is strategic, and asks the PC revolution's dangling question: "what now?"
- [[Ben Vereen on Questlove Supreme]] — QLS 232: what happens when a celebrity interview combusts into oral history — Vereen traces identity through ancestry with his daughter in the room
