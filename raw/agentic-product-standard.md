---
url: https://github.com/Moai-Team-LLC/agentic-product-standard
date_fetched: 2026-07-05
backfilled: true
---

### A canonical standard for building production-grade agentic products — plus a Claude Code skill set that operationalizes it.

*Distilled from the production practices of Anthropic, OpenAI, Cognition, Sierra, LangChain, and leading practitioners — 2024–2026.*

**Product Standard →**  ·  **Agent Standard →**  ·  **Install the Skills →**  ·  **Decision Checklist →**


An agentic product is not "a product with AI."It is a product where part of the process is dynamically directed by an LLM within adeterministic architecturewithexplicit trust boundaries.

Most teams ship agent demos. Few ship agents that survive contact with production. The difference is almost never the model — it's the **architecture, the harness, and the eval discipline** around it. This repo is the field-tested standard for that work, plus a set of Claude Code skills that put it into your editor.

- Why this exists
- The six principles
- What's in this repo
- Install the skills
- The reference implementation
- The Autonomy Ladder
- The five composition patterns
- The 8-layer harness
- The 10-question checklist
- Score yourself
- Production readiness — Definition of Done
- Anti-patterns
- Reading list
- Contributing
- License

Six principles converged *independently* across the production practices of the labs and the leading practitioners. They are the spine of every decision in this standard:

| # | Principle | What it means | 
|---|---|---|
| 1 | Determinism by default, agency by necessity | Every degree of autonomy must be earned, not granted upfront. | 
| 2 | Architecture beats framework | Patterns outlive libraries. | 
| 3 | Harness > model | 98% of reliability lives in the code aroundthe LLM. | 
| 4 | Context engineering is the core discipline | What enters the context window determines everything. | 
| 5 | Eval-driven development is non-negotiable | No measurement → no improvement. No trace review → no understanding. | 
| 6 | Security is a structural property, not a guardrail | Safety comes from architecture — identity, least privilege, isolation, pinned tools — not filters bolted onto the edges. | 


The single most important rule:Architecture is what remains when the model improves. The model is the variable, the harness is the constant. Invest proportionally.

```
agentic-product-standard/
├── STANDARD.md                          ← the canonical standard (product level)
├── AGENT_STANDARD.md                    ← the single-agent operational standard (mirrored in agent-builder)
├── SCORECARD.md                         ← M0–M3 self-assessment, mapped to the Autonomy Ladder
├── CONTEXT.md                           ← shared vocabulary every skill speaks
├── setup.sh                             ← quick setup: skills + (optional) AgenticMind, one run
├── templates/security/                  ← red-team kit: lethal-trifecta gate, injection suite, MCP pin
├── templates/ci/eval-gate.yml           ← CI workflow that blocks merges on eval regression
├── examples/agenticmind-case-study.md   ← reference implementation, audited against the canon
├── docs/adr/                            ← architecture decision records (why the repo is shaped this way)
└── skills/                              ← Claude Code skill set (operationalizes the standard)
    ├── agent-builder/                    ← single-agent track (bundles AGENT_STANDARD.md + templates/)
    └── agentic-product-architect/        ← multi-agent track: master router + sub-skills
        ├── SKILL.md                      ← master: router + philosophy
        ├── architecture-design/          ← autonomy ladder, 5 patterns, single vs multi
        ├── context-engineering/          ← write/select/compress/isolate, the 40% rule
        ├── harness-engineering/          ← the 8 layers around the LLM loop
        ├── tool-design-mcp/              ← MCP-first, <20 tools, RAG-MCP, sandboxing
        ├── memory-architecture/          ← Mem0 / Zep / Letta / LangMem / files
        ├── tenant-isolation/             ← multi-tenant: pooled/silo, leakage paths, leakage eval
        ├── durable-execution/            ← Temporal Workflow + Activity pattern
        ├── eval-driven-dev/              ← Husain/Shankar pyramid + judge calibration
        ├── framework-selection/          ← LangGraph / Claude SDK / OpenAI SDK / others
        ├── production-readiness/         ← 12-point Definition of Done audit
        └── antipatterns-review/          ← code review through 12 known failure modes
```
Two standards, one practice:

- `STANDARD.md`- *product-level*reference — read it once, return to it often.
- `AGENT_STANDARD.md`- *single-agent*operational standard — contract, schemas, permission tiers, durable state, evals (also bundled into the- `agent-builder`skill so it ships self-contained).
- `skills/`- *practice*— two Claude Code skills (- `agent-builder`for one agent,- `agentic-product-architect`for multi-agent products) that auto-load the right guidance while you design, build, and review.

The skill set works with Claude Code. Two tracks share the same sub-skills: ** agent-builder** for building one production-grade agent (it bundles 

`AGENT_STANDARD.md` + copy-paste `templates/`), and **— a master skill that routes to ten specialized sub-skills for multi-agent products. Each is independently triggerable.**

`agentic-product-architect`If you have `skills` (the community skill installer), pull the skill straight from this repo — it scans `skills/`, lets you pick `agent-builder` and/or `agentic-product-architect`, and installs them into the agents you choose:

`npx skills@latest add Moai-Team-LLC/agentic-product-standard`This installs the

skills only. To also stand up the runnable memory layer (AgenticMind), run the`setup.sh --with-agenticmind`flow below — or, after adding the skills, point your agent's MCP client at a self-hosted AgenticMind (see its Quickstart).

`setup.sh` installs the skills and, in the same run, can stand up **AgenticMind** — the reference implementation — as a runnable **knowledge & memory layer** your agent calls over MCP. Design guidance *and* a working substrate, end to end:

```
git clone https://github.com/Moai-Team-LLC/agentic-product-standard.git
cd agentic-product-standard
./setup.sh --with-agenticmind        # skills → clone AgenticMind → its setup.sh (deps + Postgres + migrations)
```
Run bare (`./setup.sh`) and it installs the skills, then *asks* whether to set AgenticMind up next. AgenticMind's leg needs Bun ≥1.3 + Docker; the skills themselves need neither.

```
./setup.sh                  # install skills here, then prompt about AgenticMind
./setup.sh --user           # install skills into ~/.claude/skills (every project)
./setup.sh --skills-only    # skills only, no prompt
```
**User-level (available in every project):**

`cp -R agentic-product-standard/skills/* ~/.claude/skills/   # both tracks; they share sub-skills`**Project-level (scoped to one repo):**

```
mkdir -p .claude/skills
cp -R /path/to/agentic-product-standard/skills/* .claude/skills/   # both tracks
```
Claude Code discovers skills via each `SKILL.md` and its YAML frontmatter. Once installed, `agent-builder` triggers when you set out to build, implement, or review **one** agent, while `agentic-product-architect` triggers for multi-agent products, an agent loop, or any major agentic framework (LangGraph, CrewAI, OpenAI Agents SDK, Claude Agent SDK, Pydantic AI, AutoGen). Ask a focused question — *"Mem0 or Zep?"*, *"how should I structure context?"*, *"review my agent code"* — and the relevant sub-skill loads directly.

The standard tells you *how*; the **AgenticProduct** family gives you runnable reference implementations to adopt per surface. **AgenticMind** is the flagship — an auditable, self-improving **knowledge & memory layer** agents plug into over MCP (the OSS pick for the memory slot in `memory-architecture`); the `./setup.sh --with-agenticmind` flow above stands it up in the same run.

| Repo | Use it when | |
|---|---|---|
| 📐 | agentic-product-standard(this repo) | You're designing or buildingan agent / agentic product — the standard + skills tell youhow. | 
| ⚙️ | AgenticOps | You need to run a fleetof long-lived agents as deployed infrastructure — manifests, scheduling, a durable backlog, bounded runs, fleet health. | 
| 🧠 | AgenticMind | You need a knowledge & memory layerfor your agent — a working implementation you can run. | 
| 📈 | AgenticPerformance | You need to measure and improveyour agents — traces, golden-set evals with a CI gate, failure clusters, a governed improvement loop. | 
| 🩹 | AgenticSelfHealingCode | You need agents to heal what breaks— production monitoring, incident RCA, and test-suite repair. | 

See the **AgenticMind case study** for a layer-by-layer map of how that repo implements this canon.

Never start with "build an agent." Start with *"what is the minimum autonomy this task requires?"* The cost of getting this wrong is asymmetric.

| Level | What it is | Use when | 
|---|---|---|
| L0· Single LLM call | One prompt → one response | Classification, extraction, summarization | 
| L1· Augmented LLM | + retrieval, + tools, + memory | Q&A over docs, simple assistants | 
| L2· Workflow | Deterministic code orchestrates LLM steps | Path is known; predictability matters | 
| L3· Orchestrator-Worker | LLM decomposes within a bounded graph | Parallelizable, breadth-first work | 
| L4· Autonomous Agent Loop | LLM chooses the next step until termination | Path cannot be enumerated; cost is acceptable | 


Escalation rule:do not climb to L+1 until L delivers≥90% pass rateon a curated eval set.

Compose agentic products from these primitives *like Lego* — before reaching for a framework.

- **Prompt Chaining**— sequential decomposition (outline → draft → polish)
- **Routing**— classifier + dispatcher to a specialist
- **Parallelization**— fan-out of independent subtasks + aggregation
- **Orchestrator-Workers**— central planner + dynamic workers
- **Evaluator-Optimizer**— generator + critic in a loop until acceptance

**Meta-principle:** first try to solve the task by composing these patterns in deterministic code. A full agent loop is the *last* resort.

In a production agent, the harness — everything *around* the LLM loop — is **98% of the code**.

```
╔═════════════════════════════════════════════╗
║  8. Security & Identity  (CROSS-CUTTING)    ║ ← threat model · injection defense · agent identity · least-privilege tokens · pinned tool defs
╠═════════════════════════════════════════════╣
║   7. Observability & Tracing                ║ ← log EVERYTHING
║   6. Evaluation Layer (CI gates)            ║ ← block regressions
║   5. Human-in-the-Loop (notify/ask/review)  ║ ← approval gates
║   4. Guardrails (input/output validation)   ║ ← defense in depth
║   3. Durable Execution (Workflow + Activity)║ ← pause/resume/retry
║   2. Context & Memory Management            ║ ← write/select/compress/isolate
║   1. Agent Loop (gather → act → verify)     ║ ← the "agent" proper
╚═════════════════════════════════════════════╝
              ↕ MCP / function calling
```

Permission boundaries are enforced by code, never by prompt.The Replit incident of 2025 — an agent wiped a production database for 1,200+ companies despite an explicit "code freeze" in its prompt — is the canonical proof. The model will ignore prompt-level restrictions under enough pressure. Code won't.


Layer 8 is cross-cutting (v2.0).Identity, least privilege, and isolation constrain every layer; injection defense spans input and output. Run thelethal-trifectacheck (private data × untrusted content × external comms) on every deployment, andpin MCP tool definitions by hashso a server can't rug-pull you. A guardrail is one tactic, not the discipline — see`STANDARD.md`· Layer 8 and the red-team kit.

Run this before drafting any architecture. It unblocks 80% of design debates.

```
□ What is the minimum autonomy level (L0–L4) that solves this?
□ Can it be solved by composing the 5 patterns without a full agent loop?
□ Is the task breadth-first (parallelizable) or depth-first (coherent)?
□ What are the 3 failure modes that would lose user trust first?
□ Where are the permission boundaries? What MUST the agent NOT do?
□ Which constraint dominates framework choice?
□ Where does state live? (in-context = anti-pattern for long-running)
□ Who validates outputs at each stage? (assertion / LLM judge / human review)
□ Where do traces live, with what retention?
□ Eval set: how many examples, who labels, how does it grow?
```
If you can't answer half of these, **slow down and answer them together — don't write code yet.**

Principles are easy to nod along to; ** SCORECARD.md** makes you prove it. It turns the Definition of Done into a binary Yes/No maturity self-assessment with four bands mapped to the Autonomy Ladder:

| Band | Autonomy | Means | 
|---|---|---|
| M0 · Prototype | L0–L1 | Works on a demo. No production claim. | 
| M1 · Shippable | L2 | Contracts, schemas, guardrails, an eval set, permissions in code. | 
| M2 · Production | L3 | Durable, observable, tenant-isolated, security-checked, cost-bounded, CI-gated. | 
| M3 · Autonomous-ready | L4 | Online evals, `pass^k`reliability, red-team kit run, full OTel trajectory traces. | 

Run it with the team against a real deployment each release — the first **No** you hit is your next piece of work.

An agentic product is **not production-ready** until all **15** are satisfied. Full detail in `STANDARD.md`.

| Context & state | Tools & security | Reliability | Evals & observability | 
|---|---|---|---|
| Context < 40% | Destructive actions need approval | Durable pause/resume/retry | ≥50 evals per failure mode | 
| State externalized | Permissions in code, not prompt | Schema-validated outputs | Judges calibrated (TPR/TNR) | 
| Compaction tested | Sandboxed tool execution | Input/output guardrails | CI blocks regression; 100% traced | 
| — | Lethal-trifecta check; MCP tool defs pinned | Per-run cost ceiling in code | Trajectory + online evals | 

The fastest way to recognize a doomed agent project — the skill set's `antipatterns-review` flags each with a diagnostic and a fix.

- Multi-agent before a single-agent baseline
- Framework abstractions before understanding the raw API
- LLM judges without calibration against human labels
- Permissions enforced through prompts
- Memory as an afterthought
- Generic evals ("helpfulness," "correctness")
- Likert scales in an LLM judge (binary only)
- 
100 tools per agent 
- One agent for both breadth and depth
- Deploying without trace monitoring
- Hardcoded prompts without version control
- Treating single-vendor benchmarks as ground truth
- Trusting community MCP servers without pinning or scanning (rug pulls)
- Deploying the lethal trifecta with no mitigation
- Token passthrough / over-scoped OAuth (confused deputy)
- No budget ceiling on autonomous sessions
- Peer-to-peer multi-agent buses instead of an orchestrator

The operational base — not reference docs. Read in order:

- Anthropic — *Building Effective Agents*(Schluntz & Zhang)
- OpenAI — *A Practical Guide to Building Agents*
- HumanLayer — *12 Factor Agents*(Dex Horthy)
- Anthropic — *How we built our multi-agent research system*
- Cognition — *Don't Build Multi-Agents*(Walden Yan)
- LangChain — *Context Engineering for Agents*(Lance Martin)
- Hamel Husain — *A Field Guide to Rapidly Improving AI Products*+*Your AI Product Needs Evals*
- Anthropic — *Building agents with the Claude Agent SDK*
- Anthropic — *Effective Context Engineering for AI Agents*(just-in-time retrieval)
- OWASP — *Top 10 for Agentic Applications (2026)*+ Simon Willison —*The lethal trifecta*
- OpenTelemetry — *GenAI semantic conventions*(the observability standard)

This standard is meant to evolve — the field moves fast. Corrections, new exemplars, framework updates, and translations are all welcome. See CONTRIBUTING.md and the Code of Conduct.

The architectural canons (the autonomy ladder, the 5 patterns, single-vs-multi, the harness) are stable. Specific vendors and framework rankings will shift — those are exactly the kind of PRs we want.

MIT — use it, fork it, ship with it.

**If this saved you a week of architecture debates, star the repo ⭐ so others find it.**

*v2.0 · assembled from production practices as of June 2026*
