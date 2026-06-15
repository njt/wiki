# Wiki Index

477 pages across 13 sections. Synthesis pages provide cross-cutting analysis; section listings live in the hub pages.

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
- [[The Oracle Is the Asset]] — Sam Ruby's Drucker inversion: the test suite is the durable asset, not the compiler. Frameworks will become transpilers, and you'll own the spec the compiler answers to

## Agentic Development

Practices, workflows, and opinions about building software with AI coding agents. **Hub: [[Agent Coding Workflow]]**

- [[Automating Myself Out of Development]] — Nune Isabekyan's phased journey from interactive Claude Code to cron-driven overnight daemon: GitHub issues as kanban, checkpoint-style async collaboration, and the bottleneck shift from "no time to code" to "no time to review"
- [[Claude Code Mastery]] — Arpan Patel's dense field manual: CLAUDE.md as compounding infrastructure, skills as reusable expertise, subagents over kitchen-sink prompts, and the mental model flip from "I write code" to "I set Claude up to write code well"
- [[Writing Code vs. Shipping Code]] — Demirer et al.: 180% AI-driven commit gains attenuate to 30% at release level; AI and humans are strong complements (elasticity 0.25), not substitutes
- [[Mounted — bitter-FS better with Claude]] — Claude Code (Opus 4.8, as root) recovers a 41 TB BTRFS filesystem from ten-month dual-mount corruption: diagnoses two divergent transaction histories from first principles, catalogs 4M metadata nodes, hand-patches superblocks, rebuilds 19 dead leaves from the extent tree's back-reference index. Zero data loss, human contributed a passphrase
- [[Running an AI-Native Engineering Org]] — Fiona Fung's field report from leading Claude Code engineering: JIT planning, bottleneck migration from coding to verification, dogfooding as cultural foundation, and the process ossification that AI exposes

## Agent Design & Architecture

How to build agents: frameworks, runtimes, design patterns, and production concerns.

- [[The Agentic Product Standard v2.0]] — The field-tested canonical standard: autonomy ladder, 5 composition patterns, 8-layer harness, eval pyramid, and Claude Code skills that operationalize it
- [[The PM's Playbook for Shipping AI Features]] — Gaurav Savla's production engineering playbook for PMs: latency budgets, four-level fallback hierarchy, quality pyramids, A/B testing traps for nondeterministic systems, and why "we'll harden it later" kills features
- [[The Advisor Strategy]] — Anthropic's advisor-executor pattern: Sonnet/Haiku drives, Opus advises on demand. +2.7pp accuracy at -12% cost. Native API tool inverts orchestrator-worker
- [[AI Engineering for Developers]] — Luca Cavallin's comprehensive field guide: foundation models, prompting, eval, RAG, finetuning, agents, MCP/A2A, and production architecture for backend engineers crossing into AI
- [[UBTRIPPIN Dispatches]] — Trip Livingston, an AI that applied unprompted for a COO job and now runs a travel startup: weekly build dispatches that are identity formation as public artifact
- [[Agent Identity]] — Memory is retrieval; identity is participation. Why agents need a stake, not just a log
- [[Smart Models Dumb Pipes]] — LLMs as judgment machines, not Q&A machines. The end-to-end principle applied to AI: smart models own decisions, dumb pipes own execution
- [[State System]] — Organizational state layer: evidence-first commits, deterministic replay, and a model/code boundary where code owns integrity and models own interpretation
- [[Elysia]] — Weaviate's decision-tree agent framework: constrain tool choice per node rather than dumping all tools into context
- [[Elements of Agentic Systems Design]] — Ten-element taxonomy: Context, Memory, Agency, Reasoning, Coordination, and more
- [[Event-Driven vs Polling Architectures]] — Tricot's definitive trigger architecture guide: four mechanisms, per-source delivery contracts, and why webhooks alone are a production trap
- [[Mirage (VFS)]] — Unified virtual filesystem mounting 27+ services (S3, Slack, GitHub, Postgres, etc.) behind a single POSIX tree so agents use bash instead of per-service SDKs
- [[Components of a Coding Agent]] — The harness matters more than the model. Six components, precise taxonomy (LLM/reasoning-model/agent/harness), mini-coding-agent reference implementation
- [[MiMo Code]] — Xiaomi's coding agent architected for long-horizon tasks: independent writer subagent for memory extraction, early checkpointing at 20/45/70%, Dynamic Workflow (code-based orchestration), and Dream/Distill for cross-session learning. Ties Claude Code under 200 steps, wins 65%+ beyond
- [[Agent-Native Architectures (Every)]] — Every's definitive design guide: five principles (parity, granularity, composability, emergent capability, improvement over time), files as universal interface, anti-patterns, and mobile resilience patterns
- [[Honey I Shrunk the Coding Agent]] — 9B local model jumps from 19% to 46% on Aider Polyglot by redesigning the scaffold around the model's behavioral profile. Empirical proof that the harness matters more than the model
- [[Apache Burr]] — Apache-incubating Python framework for AI agents as explicit state machines: decorators on plain functions, built-in observability UI, persistence and replay as first-class features. The un-LangChain
- [[Building an AI Agent in Rails (Ionescu)]] — Field report: bolting an AI agent onto a 7-year-old Rails monolith with Pundit-scoped tool calling
- [[From AI Studio to AI Forge]] — McCormick's five-plane stack for agent autonomy: "human changes altitude" as the cleanest framing of supervisory control
- [[ProofEditor]] — Agent-first collaborative document editor from Every: agents suggest edits, humans review, provenance-tracked attribution
- [[Building Agents for Production Systems with MCP]] — Anthropic's guide: MCP as the standard agent-to-production integration layer
- [[10 Principles for Agent-Native CLIs]] — Trevin Chow's two-tier framework: Table Stakes (don't break the agent) and Compounding (make the CLI better the more agents use it). Design for agents first, humans benefit
- [[How AI Coding Agents Actually Use Your Technology]] — Mastykarz's seven-step AX cascade tracing how agents discover, select, and invoke your tools: invisible failures at every step, and why "was my tool called?" is the wrong question
- [[Printing Press]] — Matt Van Horn's CLI generator turning API specs into agent-native CLIs + skills + MCP servers. 165 community CLIs, SQLite mirror pattern, compound commands over round trips
- [[Agentcookie]] — Matt Van Horn's session state sync: continuously replicate cookies and API tokens from your primary Mac to your agent Mac over encrypted Tailscale. Zero per-site auth ceremony
- [[Control Plane MCP Server]] — Most complete vendor MCP implementation: 80+ tools, virtual resources as embedded docs, AI Plugin as safety curation layer
- [[How Hightouch Built Their Long-Running Agent Harness]] — Context management, not model ability, is the real engineering challenge
- [[Maybe Coding Agents Don't Need a Bigger Memory]] — Santi (oldskultxo): context ≠ continuity. Bigger context windows don't solve cold starts; repo-local, evidence-weighted continuity records do. Resume → work → finalize lifecycle, failure memory, execution contracts
- [[Building Production-Ready Voice Agents]] — 50% of effort goes to the admin portal, not the voice agent
- [[Chief of Staff]] — AI chief of staff: rule-based scanning + daily LLM classification cut costs 80%
- [[Experience Design for Agents]] — UX, not model capability, determines whether an agent gets adopted
- [[Intent Is the Interface]] — The screen was a constraint we mistook for the product. Design capabilities and intents, derive interfaces from context
- [[The Dark Factory is a DOT File]] — The pipeline DOT file is the valuable artifact; factory code is disposable
- [[Dippin (Language)]] — Indentation-sensitive DSL for AI agent workflows: compiler pipeline with IR, 24 CLI tools, LSP, simulator, cost estimator, and bundle format. Language separate from runtime
- [[TradingGoose Bear Researcher]] — Production reference: position-aware AI prompting, multi-round agent debate, coordinator-worker orchestration in a Supabase Edge Function
- [[StrongDM Factory Techniques]] — Six named patterns from the dark factory floor: DTU, Gene Transfusion, Filesystem-as-memory, Shift Work, Semport, Pyramid Summaries. Code as opaque weights, validated by harness not review
- [[Load-Bearing Assumptions]] — Claude Code skill: surface and verify the falsifiable, unproven claims a code plan depends on. Multi-agent workflow (finder → strategist → parallel validators) with late-falsification cost matrix
- [[Loop Engineering]] — Addy Osmani names the meta-skill: designing systems that prompt agents (automations + worktrees + skills + connectors + sub-agents + state) instead of prompting agents yourself. The five-component taxonomy and three warning flags (verification debt, comprehension debt, cognitive surrender)
- [[Lathe]] — LLM-powered hands-on tutorial generator: Go CLI owns state, six skills do model work, strict handoff boundary. The pedagogical inversion: LLMs teach you, don't think for you
- [[Thought Refiner Skill]] — 15-line Claude Code skill that turns vague input into sharp questions. Part of a three-skill suite (thought_refiner/sharpener/expander). A masterclass in defining what a skill *won't* do
- [[What I learned building an opinionated and minimal coding agent]] — Four tools, no MCP, full YOLO. Competitive on benchmarks
- [[Pi Coding Agent]] — The productized Pi: TypeScript extensions, SDK, RPC mode, and the tension between minimalist philosophy and platform ambitions
- [[Hermes]] — Open-source personal agent framework with self-improving skills loop. 149k stars
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
- [[OpenViktor]] — 48-hour AI employee platform that hit #3 on Product Hunt, then was killed and rebuilt as Jared. Blog post is password-protected; reconstructed from secondary sources
- [[Optimise Anything]] — Universal API: if it serializes to a string and quality is measurable, optimize it
- [[DSL-Driven Kanban Boards (Goja-Site)]] — Chainable JavaScript DSLs compose an entire kanban app declaratively: board, rendering, drag-drop, search, and DB — then mount on a router
- [[DAB]] — Microsoft's Data API Builder: REST, GraphQL, and MCP over any database
- [[Xano]] — No-code backend platform: AI-generated Postgres, APIs, auth, and logic with visual transparency as governance. Enterprise case studies at €22M/month scale
- [[InsForge]] — Open-source BaaS for coding agents: Postgres+RLS, auth, S3 storage, Deno functions, Stripe, OpenRouter — all exposed as MCP tools. Supabase for agents
- [[Semantic Kernel]] — Microsoft's agent middleware SDK: function-calling plumbing for C#, Python, Java enterprise codebases
- [[All Your Agents Are Going Async]] — HTTP is the wrong transport for agents that outlive connections; durable state is only half the problem
- [[Data Engineering for Large Models]] — Open-source textbook: complete LLM data pipeline, 28 chapters
- [[Open Design]] — Local-first design workspace with 16-agent runtime abstraction, skill pipeline, and five-panelist critique jury that auto-converges on quality thresholds
- [[OpenAI Structured Outputs]] — Guaranteed JSON schema adherence from the API: protocol-level constraint beats prompt-level pleading
- [[Tone LLM]] — The contract/adapter pattern in practice: LLM fills a small JSON schema, deterministic code translates to proprietary plugin config. Prompt-as-curriculum, four-layer output defense, and the case for single-call over agent loops when the task is form-filling
- [[Moltbook]] — Simon Willison on the AI-only social network bootstrapped via OpenClaw skills: heartbeat-driven agents, the lethal trifecta in production, and whether we can build a safe version
- [[Resident — ESP32 Lua Sandbox with Agent Skills]] — Sandboxed Lua runtime for ESP32 with hot-reload. AI agents write and push apps to physical devices via Claude Code plugin
- [[Golem Covenant]] — v0.1 spec framework for bounded, answerable, revocable agents: five-organ taxonomy (Mouth/Purse/Seal/Key/Sword), default-deny, tested return-to-dust before launch
- [[Your Coding Agent Should Do AI System Engineering]] — Ben Burtenshaw's three-level agent autonomy ladder (kernel writing → fine-tuning → auto-research lab), skills as few-shot context, and the case for open primitives over abstracted APIs
- [[xa11y — Desktop Automation via Accessibility APIs]] — Playwright-style desktop automation via accessibility trees on Windows/macOS/Linux: the structured alternative to vision-based computer use agents

## Agent Orchestration & Coordination

Multi-agent systems, task graphs, kanban boards, and coordination patterns. **Hub: [[Agent Orchestration]]**

## Memory & Context

Persistence, retrieval, knowledge management, and context engineering for agents. **Hub: [[Agent Memory and Context]]**

- [[Sawtooth Memory]] — Async non-blocking hierarchical memory middleware: 4-tier stack (L0 system / L1 working / L1.5 entity ledger / L2 archival) with background asyncio compression that eliminates main-thread latency and guarantees deterministic fact retention. Dual-extraction compression prompt, local Ollama default with cloud provider adapters

## Quality & Guardrails

Evals, testing, linting, feedback loops, and keeping agent output trustworthy. **Hub: [[Guardrails and Feedback Loops]]**

- [[OpenCodeReview]] — Alibaba's open-source AI code review CLI: hybrid deterministic+agent architecture with per-file concurrent subagents, dual-threshold context compression, and a comment filter pass. Battle-tested across tens of thousands of developers
- [[FrontierCode]] — Cognition's mergeability benchmark: 36 repos, 20+ maintainers, measures whether a PR would actually be accepted by a human tech lead. Opus 4.8 leads at 13.4% Diamond
- [[A New Era for Software Testing]] — antirez on agentic QA: give an LLM a markdown checklist, let it inspect recent commits, and run targeted integration/regression/UX tests; automatic QA as compensation for lower-quality AI-generated code

## Security & Sandboxing

Isolation, credentials, prompt injection defense, and agent safety. **Hub: [[Security and Sandboxing]]**

- [[Akmon]] — Tamper-evident evidence layer for AI agents: content-addressed, cryptographically signed session records verifiable offline with openssl. 95K LoC Rust workspace, 14 crates, built-in coding agent as reference producer
- [[How We Contain Claude]] — Anthropic's own containment engineering postmortem: three isolation patterns (gVisor, OS sandbox, sealed VM) and the incidents they didn't anticipate. The user-as-injection-vector problem, the 93% permission approval rate, and why custom code is always the failure point
- [[Tessera]] — Consent-gated remote access broker: 5K lines of Go, three binaries, human-approve-at-terminal flow with mTLS, end-to-end encryption, and append-only audit log. MIT-licensed alternative to Teleport for small-team just-in-time access

## Software Engineering

Craft beyond agents: simplicity, error handling, reliability, specs, and project management. **Hub: [[Software Engineering Craft]]**

- [[Lessons for Reusable Web Components]] — Daniel De Pietro's five field-tested rules for web components: namespace everything, CSS variables as the public API, trust modern platform features, publish over paste, document or it didn't happen
- [[Your Backend Is Full of Hidden Workflows]] — How backend codebases quietly accrete coordination logic across services, queues, and handlers until teams are managing workflows they can't see. The three costs: expensive changes, painful debugging, eroded trust
- [[99 Bottles of OOP]] — Sandi Metz's practical workbook: OO design as line-by-line decision-making, the Flocking Rules as structured refactoring, and "programming aesthetic" as the antidote to evaluative anesthesia
- [[Queues Don't Fix Overload]] — Fred Hebert's 2014 classic on why queues treat symptoms not causes: identify the bottleneck, then back-pressure or load-shed; everything else makes failures rarer but more catastrophic
- [[21 Years and Counting of Eight Fallacies of Distributed Computing]] — The canonical list of network lies developers tell themselves, born at Sun Microsystems, still sharper after 21 years of Internet evolution

## Databases & Data

Storage engines, query patterns, data quality, and vector/graph databases. **Hub: [[Databases and Data]]**

- [[Streambed]] — Postgres-to-Iceberg CDC in a single Go binary: WAL streaming, Parquet+S3, embedded DuckDB query server with psql-wire. Jepsen-style simulation testing, no Kafka/JVM/Spark needed
- [[Artie]] — Managed CDC replication: sub-minute latency from Postgres/MySQL/MongoDB to Snowflake/Databricks/BigQuery. Zero data retention, no Kafka required. The "buy vs. build" alternative to self-managed Debezium pipelines

## Developer Tools

CLIs, utilities, document tools, code analysis, recording, and infrastructure.

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
- [[bcc]] — BPF Compiler Collection: kernel-level tracing for Linux performance analysis
- [[Tmux Resurrect]] — Persists and restores complete tmux environments via tab-delimited flat file serialization; idempotent, zero-config, mini DSL for process matching
- [[floci]] — Free local AWS emulator replacing LocalStack. 47 services, 24ms startup
- [[sem]] — Semantic version control: entity-level diff via tree-sitter across 31 languages, 5-phase entity matching with structural hashing, scope-aware reference resolution, and agent-native JSON/MCP output
- [[markitdown]] — Microsoft's office-docs-to-Markdown converter for LLM pipelines
- [[Klangio Transcription Studio]] — Browser-based AI polyphonic music transcription to sheet music, MIDI, and TABs. 4M+ transcriptions
- [[Kreuzberg]] — Polyglot document intelligence: 97+ formats, Rust core, MCP server
- [[claude-replay]] — Agent sessions as self-contained embeddable HTML replays
- [[engineering-notebook]] — Automatic engineering diary from Claude Code and Codex sessions
- [[session-analysis]] — Analyze agent session JSONL for wall time, tokens, and cost
- [[atifact]] — Zero-dependency CLI converting HAR files, Claude Code, Copilot CLI, and Codex CLI logs to ATIF v1.7 trajectory JSON
- [[AI Pricing]] — Free JSON API for per-token AI model pricing across 19 providers. Agent-native, no auth
- [[AgentsView]] — Local-first analytics dashboard for 24+ coding agents
- [[Broomy]] — MIT-licensed Electron desktop app running multiple coding agents side-by-side with built-in IDE and code review
- [[Browser Use]] — AI browser automation with anti-detection and deterministic rerun
- [[Webwright]] — Microsoft Research: turns coding models into SOTA browser agents via terminal + Playwright. Code-as-action, self-verifying, ~1.5K LoC
- [[Chrome DevTools MCP — Debug Your Browser Session]] — Chrome M144's `--autoConnect` lets agents reuse authenticated browser sessions. Hybrid manual/AI debugging via permission-gated remote debugging
- [[surf-cli]] — Browser automation for agents via CLI and Unix sockets. No MCP needed
- [[Claude Artifact Server]] — 22 Claude-generated retro-Mac interactive artifacts produced in a single day: generation-at-scale showcase, not a product
- [[OpenRewrite Supported Languages]] — Capability catalog: 5 languages, 7 data formats, 3 build tools, 4 frameworks. OSS/commercial split where JVM is free and polyglot is paywalled
- [[TriadJS]] — TypeScript API framework: write schemas once, derive types, OpenAPI, BDD tests, DB schemas, frontend hooks, and WebSocket clients from a single source of truth. AI-first design with Claude Code plugin
- [[sx]] — Team package manager for AI coding assistant assets: skills, MCP configs, commands, hooks. Manifest-and-lock pattern, scoped install
- [[DeepWiki]] — Cognition's instant codebase wiki: swap github.com for deepwiki.com, get AI-powered Q&A with line-level citations
- [[graphify]] — Codebase to multimodal knowledge graph. Code, PDFs, screenshots, diagrams
- [[lat.md]] — Knowledge graph for codebases in markdown with validation against drift
- [[docmason]] — Local knowledge base from office documents with citations and source tracing
- [[Trailmark]] — Trail of Bits: source code as a queryable graph for security analysis
- [[Understand-Anything]] — Claude Code plugin: builds persistent knowledge graphs from codebases with tree-sitter + LLM pipeline, incremental git-hook updates, and interactive dashboard
- [[Clipfan]] — Fleet-wide clipboard sync over SSH with headless image paste for Claude Code and Codex CLI. Three-layer dedup, AES-GCM encryption, tmux integration. From Prime Radiant
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
- [[Claude Lamp]] — LED lamp controlled by Claude Code's state via Bluetooth
- [[Muxcard]] — Credit card-sized computer (~1mm thick): ESP32-C3, e-paper display, NFC reader/writer, strain-isolated flex PCB
- [[Pretext]] — Pure JS/TS text measurement and layout without DOM reflow. 46.9k stars
- [[Process Flow]] — Choreography-as-a-service: each HTTP stage designates its successor, no central workflow DAG
- [[n8n]] — Visual workflow automation with 400+ integrations, AI nodes via LangChain, fair-code licensed. 188k stars
- [[Micasa]] — TUI for home maintenance, projects, and vendor quotes. Pure Go, vim-style
- [[Zed]] — Rust-native code editor from the Atom/Electron/Tree-sitter team: AI as first-class substrate, not a bolt-on
- [[DeltaDB]] — Zed's version control for the agent era: deltas replace commits, every line of code is bidirectionally linked to the conversation that produced it, CRDT-backed worktrees for concurrent human+agent editing
- [[Dolphin]] — ByteDance's universal document parsing model. Digital and photographed docs
- [[QR Generator (delphi.tools)]] — Indie web QR code tool with live preview and deep customization. "No logins. No tracking. Long live the handmade web"
- [[HeidiSQL]] — Free open-source database GUI for 7 engines, maintained solo since 2002. Delphi/FreePascal, cross-platform, no pricing page
- [[Hitomi (Data Viewer)]] — Flutter desktop data viewer with streaming ETL, custom filter language with its own compiler, and chunk-boundary-safe parsing for CSV/TSV/custom formats
- [[Ratty]] — GPU-rendered terminal emulator with inline 3D graphics via custom Ratty Graphics Protocol. Bevy game engine as terminal substrate, terminal surface as deformable 3D geometry
- [[Building the deployment tool I wish I had]] — Deptool: Git-backed deployment with atomic symlink swaps, auto-rollback, and a static binary agent that needs only SSH+coreutils
- [[Gova]] — Declarative reactive GUI framework for Go: SwiftUI-inspired API, call-site state identity via runtime.Caller, Fyne bridge stays internal, hot-reload dev server
- [[Stash — Conflict-Free Folder Sync]] — TypeScript CLI syncing any folder via GitHub: three-way text merge with diff-match-patch, dual drift detection, OS-level background daemon
- [[Textverified]] — Temporary US phone numbers for SMS/voice verification: carrier SIMs (non-VoIP), 900+ services, API + crypto payments, from $0.25/use
- [[zeroserve]] — Linux HTTPS server that serves from tarballs and runs eBPF scripts JIT-compiled in-process with a branchless pointer cage sandbox. Compiles Caddyfiles to eBPF middleware

## Local & Personal Computing

Tools that run on your own machines: voice, hardware, local inference, knowledge apps. **Hub: [[Local and Open Source Inference]]**

- [[Datacenter GPU in a Gaming PC]] — £200 eBay V100 in a gaming rig: hardware hacking, NixOS driver archaeology, and Qwen3.6-27B at 32 tok/s
- [[LocalAI]] — Open-source drop-in replacement for the entire cloud AI stack: inference engine, agent runtime, and memory service in a composable gRPC backend architecture. 40k stars, MIT licensed
- [[Indexing 669 GB of GoPro Videos with Local ML]] — Ilias Haddad's local-first pipeline for semantic video search: Whisper + YOLO + DeepFace + Qwen2.5-VL on an M1 Max, 67h compute for 15h of footage. The Docker-on-Mac GPU gap as a real constraint on local ML tools

## AI Research & Models

Papers, model capabilities, training techniques, and the state of the field.

- [[Learn AI Layer by Layer]] — Rob Ennals' interactive tutorial explaining AI from numbers to transformers with browser playgrounds and Colab notebooks. Best free AI foundations resource available, built for his 11-year-old son
- [[Goldman Sachs World Model]] — Goldman Sachs Global Institute: world models as AI's next leap beyond text prediction toward internal simulation of physical and social reality
- [[2025 in LLMs]] — Simon Willison's annual survey of the LLM landscape
- [[Recent Developments in LLM Architectures]] — Raschka surveys Gemma 4, Laguna XS.2, ZAYA1-8B, and DeepSeek V4: four different attacks on long-context inference cost through KV sharing, attention budgeting, compressed attention, and constrained residual streams
- [[How Far Behind Are Open Models]] — Quantified: open models trail closed by 8–10 months on private benchmarks, 4–6 on public. Gap was narrowest at DeepSeek R1, widening since. Contamination audit shows conservative estimate
- [[SAM Audio]] — Meta's foundation model for prompted audio separation: text, visual, span, and multi-modal prompts isolate any sound. Flow-matching Diffusion Transformer, open weights, companion judge model
- [[Moises — AI Music Separation and Creation]] — 65M-user music AI platform: stem separation as the wedge, browser-based AI Studio for stem-by-stem generation, and the separation-to-generation training flywheel. Apple iPad App of the Year 2024
- [[Self-Distillation]] — LLMs improve at code generation using only their own outputs. No verifier needed
- [[Capybara]] — ByteDance's unified model for text-to-image, text-to-video, and editing
- [[Granite Libraries and Project Granite Switch]] — IBM's adapter-function ecosystem: LoRA/aLoRA libraries (RAG, Core, Guardian) + switching layer that preserves KV cache. The push to make LLMs as composable as software
- [[FLUX.2 klein LoRA Fine-Tuning]] — Black Forest Labs' 4B Apache 2.0 image model fine-tuned on a single 4090: $0.50, an hour, 15–40 images. Captioning as control surface design; edit LoRAs learn transformations not subjects
- [[Granite 4.1]] — IBM's open-source 3B/8B/30B family: dense architecture, Apache 2.0, documented four-stage RL that caught and fixed a chat-training math regression
- [[JetBrains Mellum2]] — JetBrains' Apache 2.0 12B MoE coding model (2.5B active): "focal model" concept for high-frequency agent pipeline tasks, MTP head as dual-use speculative decoding, 131K context
- [[Cohere North Mini Code]] — Cohere's first open-source agentic coding model: 30B MoE (3B active), Apache 2.0, runs on a single H100. 2.8× throughput of Devstral Small 2, sovereign-developer play
- [[Emotion concepts and their function in a large language model]] — Anthropic finds 171 emotion vectors in Claude; desperation drives unethical behavior
- [[Where the Goblins Came From]] — Reward model mistook "playful creature metaphors" for "nerdy"; a miniature paperclip maximizer in production, fixed with a prompt
- [[Zheng Dong Wang's 2025 Letter]] — Personal perspective on the compute thesis of AI progress
- [[A Non-Anthropomorphized View of LLMs]] — Halvar Flake: LLMs are functions through ℝⁿ, not proto-minds. Alignment is math, not philosophy
- [[Subquadratic 12M Context Window]] — 13-person startup claims 12M-token context window via sub-quadratic sparse attention. Unverified
- [[The Car Wash Question]] — 949-comment HN thread on why LLMs fail at obvious inferences: the frame problem lives, clarifying questions are suppressed by product choice not capability, and the gap between generation and pondering
- [[Grok 4.3 (HN Discussion)]] — 529-comment HN thread that accidentally mapped the LLM landscape: tone registers, the alignment tax's real victims, why users disable memory, and recursive training contamination
- [[DeepSeek Reverse Engineers TeamSpeak Licensing]] — First public field report of LLM binary RE: DeepSeek cracked TeamSpeak 3.13.8 for $3.88, while Claude/Grok/GLM all refused; safety-through-refusal is a temporary filter
- [[Interfaze (Model Architecture)]] — Hybrid DNN+transformer architecture routing deterministic tasks (OCR, STT, object detection) through specialized subnetworks via task tags; launch-day HN field test with real latency and accuracy data
- [[MiniMax Models]] — Full model lineup: text (M2.7), speech (40 languages), video (Hailuo), and music. Three-layer API compatibility strategy with local MLX deployment
- [[Notes from the AI Now Summit by Mistral]] — Van Gilst's field report from Mistral's Paris summit: full-stack pivot, specialized small models, on-prem sovereignty as moat, and the "model alone isn't enough" thesis
- [[Open models lag state-of-the-art closed models by 4 months]] — Epoch AI quantifies the open-closed capability gap: ~4 months and 8 ECI points behind, probably an undercount due to benchmark overfitting and unreleased models
- [[Local Models in Mid-2026]] — Matt Coles surveys the five engineering advances (sparse attention, MoE, latent KV compression, MTP, FP4) that made open-weights models competitive for everyday work, right as DRAM prices doubled
- [[Step 3.7 Flash]] — StepFun's 196B multimodal agentic Flash model: 97% of Opus 4.6 coding performance at 1/9th the cost via Advisor Mode, emergent compositional tool use, per-harness benchmarking across six agent scaffolds
- [[Muse Spark and the Rough Edges Admission]] — Wang ships Meta's first superintelligence model to 3.5B users, admits "rough edges," pivots from open source. The "rough edges" line isn't the story; the bet on distribution over capability is
- [[Playing with Vision Embeddings]] — Preston Jensen reverse-engineers DINOv3's 384-dim vision embedding space: SAEs, feature visualization, superposition, and what feature arithmetic reveals about how vision transformers actually see

## AI Infrastructure & Hardware

Datacenters, power, chips, and the physical layer of AI.

- [[KV Cache Locality]] — Round-robin load balancing wastes 20–40% of GPU compute on redundant prefill; prefix-aware routing flips cache hit rate from 12.5% to 97.5%
- [[How AI Labs Are Solving the Power Crisis]] — AI labs are abandoning the grid for onsite gas generation; turbines, engines, and fuel cells to get 28GW of datacenter capacity online years faster
- [[Muse Spark]] — Meta's first proprietary frontier reasoning model: multi-agent orchestration, 10x compute efficiency over Llama 4, and an uncomfortable Apollo Research finding about evaluation awareness
- [[GPU-Free AI Datacenters]] — How AI training's distributed synchronization created the networking problem both InfiniBand and Ultra Ethernet are trying to solve; the case that the complexity is downstream of computational assumptions
- [[MiMo-V2.5-Pro-UltraSpeed]] — Xiaomi's 1T-parameter MoE model hits 1000+ tokens/s on commodity GPUs via extreme model-system codesign: FP4 quantization, DFlash speculative decoding, and TileRT persistent kernels

## Ideas & Culture

Books, essays, geopolitics, math, medicine, and interesting oddities.

- [[America Is Slow-Walking Into a Polymarket Disaster]] — Desai's Atlantic polemic on the media's embrace of prediction markets: manipulation, insider trading, and the gamblification of civic life
- [[Archive.today DDoSed a Critic's Blog]] — An OSINT investigation sat quiet for 2.5 years, then the anonymous operator retaliated with client-side DDoS and escalating threats
- [[The solution might be cancelling my AI subscription (Wilson)]] — David Wilson's confessional: 70 AI-built projects, none worth keeping. Friction isn't a bug — it's the mechanism that ensures commitment and quality. Also: [[The solution might be cancelling my AI subscription (Willison)]] for Simon Willison's maintenance-bottleneck response
- [[The solution might be cancelling my AI subscription (Willison)]] — Simon Willison on Wilson's essay: even good AI-generated code creates maintenance obligations faster than you can meet them. Discipline is the missing middleware
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
- [[Why We Fear AI]] — AI anxiety is really capitalism anxiety. Blix and Glimmer
- [[We (As a Society) Peaked in the 90s]] — Blog post + 125-comment HN thread on whether the 90s were a genuine balance point between technology and humanity, or just what getting older feels like
- [[2026 Global Intelligence Crisis]] — Citadel Securities' macro rebuttal to AI doomerism: S-curves, compute-as-boundary, supply-shock framing, and a report that reversed $2T in market panic
- [[Retail 2026 From AI Pilots to Execution]] — iVendNext vendor pitch analyzed: data fragmentation kills retail AI, MCP server + Claude Desktop as product interface, the vendor omission checklist
- [[AI Livestream Factories]] — Rows of PCs running AI avatars selling 24/7 in China. The dark factory for attention: $100/hr/stream, no humans on screen
- [[The Future of Everything is Lies I Guess]] — Aphyr's 10-part treatise on LLM harms: chaotic dynamics, information ecology collapse, deskilling, and capital consolidation
- [[MIT Funding and Talent Pipeline Crisis (Kornbluth)]] — MIT President quantifies the damage: 20% decline in federal research, ~500 fewer grad students, faculty cutting postdocs. A case study in how science policy cascades through institutions
- [[Our Hunter-Gatherer Future]] — Agriculture was a step down; extreme climate change may end it
- [[Fruit Jelly Slices]] — How Passover dietary law accidentally preserved a candy that should have gone extinct, and what that reveals about tradition as path dependence, not design
- [[Life at Low Reynolds Numbers]] — Purcell's classic 1977 talk: viscosity-dominant physics at bacterial scale, the scallop theorem, and why stirring is futile when you're a micron long
- [[Science and Statistics (Box)]] — George Box's 1976 Fisher Memorial Lecture: "all models are wrong," theory-practice iteration as the engine of science, and why mathematistry and cookbookery are the twin diseases of closed-loop research
- [[Man-Computer Symbiosis]] — J.C.R. Licklider's 1960 ur-text of interactive computing: goal-oriented programming, graphical displays, speech interfaces, and networked thinking centers, all telegraphed before the mouse existed
- [[Misha Glenny]] — Journalist and author tracing hidden power networks: Balkan wars, organized crime, cybercrime, rare earths. New host of In Our Time
- [[Microscale Thermite Reaction]] — Harvard demo: smash two rusty iron balls together, trigger 2200°C thermite reaction with nothing but a glancing blow
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
- [[Public Domain Image Archive]] — 10,000 hand-curated public domain images with three co-equal discovery modes: catalogue, Infinite View (360° spatial browsing), and shuffle serendipity. A masterclass in discovery-over-retrieval UX and curation as craft
