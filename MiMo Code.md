# MiMo Code

Xiaomi's terminal-based coding agent, designed from first principles for long-horizon tasks (200+ execution steps). Where most coding agents treat memory as an afterthought bolted onto a ReAct loop, MiMo Code builds its entire architecture around three time scales: single-turn reasoning (computation), multi-turn continuity (memory), and cross-session improvement (evolution). The headline result: tied with Claude Code under 200 steps, wins 65%+ beyond 200. This is the first coding agent where the memory architecture isn't a feature — it's the product.

---

## The Three-Pillar Architecture

MiMo Code's design is organized around **computation** (making each turn count), **memory** (surviving hundreds of turns without context rot), and **evolution** (learning across sessions). Each pillar has at least one genuinely novel contribution.

### Computation: Max Mode + Goal

**Max Mode** generates N=5 candidate solutions per turn in parallel (temperature=1, so they're genuinely diverse), then has the same model judge and select the best. 10–20% improvement on SWE-Bench Pro at 4–5× token cost. This is Best-of-N at the turn level — not a new idea (see [[MiMo-V2.5-Pro-UltraSpeed]]'s speed-to-intelligence argument), but applied at the architectural level rather than the inference level.

**Goal** is the more interesting contribution: an independent verifier model checks whether the user's natural-language stopping condition is actually met before the agent can terminate. The verifier is independent specifically to prevent alignment bias — it doesn't develop the same blind spots as the agent doing the work. This is a [[Guardrails and Feedback Loops]] pattern applied to termination decisions, and it's clever.

### Memory: The Writer Pattern

This is MiMo Code's defining contribution. The core insight: **the main agent does not maintain its own memory.** A separate writer subagent, triggered by the runtime at checkpoints (20%, 45%, 70% of context budget), reads the conversation and writes structured state to disk. The main agent never touches memory management.

Why this matters: the team found that asking a debugging model to also maintain structured logs makes it do both tasks worse. This is a classic concurrency/attention argument — the same cognitive resource can't optimize for two things at once — but applied to LLM attention rather than CPU cycles. It's the same reason [[Scaling Long-Running Agents]] found that planner/worker/judge beats flat self-coordination.

The early-checkpoint strategy (20/45/70%) is counterintuitive and well-argued. Most systems wait until the window is nearly full, then desperately compress. MiMo Code argues this is "exactly backwards" — model capability degrades under high context utilization ("lost in the middle"), so you're asking the model to do its hardest work at its weakest moment. Extract early, extract often, and the final rebuild becomes assembly rather than emergency compression.

### Evolution: Dream and Distill

Two cron-like maintenance processes run on the project memory file:

- **Dream** (every 7 days): merges, deduplicates, and compresses scattered memories into current state
- **Distill** (every 30 days): identifies recurring work patterns and solidifies them into reusable skills, CLI commands, and SOPs

This is [[napkin]]'s "markdown file that learns" taken seriously as a first-class system. The key design choice: all memory is file-based (Markdown), not vector DB. The rationale is **reviewability** — users can read, edit, and delete what the agent remembers. This is the right call for a tool that developers need to trust.

---

## Key Quotes

> "The main agent does not maintain its own memory."

The single most important design constraint in the system. It's a separation of concerns that most coding agents violate. Claude Code asks the same model to debug AND maintain context; MiMo Code says no, those are different jobs. This is the architectural equivalent of "don't mix computation and I/O in the same thread."

> "Asking the model to perform the most critical compression at the very moment when its compression ability is degrading is a bad trade-off."

The early-checkpoint argument in one sentence. This directly challenges the default behavior of every major coding agent, which compresses only when forced. MiMo Code treats checkpointing as preventive maintenance, not emergency response.

> "Orchestration logic exists in natural language, and natural language is ambiguous, forgettable, and unverifiable."

The justification for Dynamic Workflow — turning orchestration from prompts into JavaScript code. This is the same insight that drove [[Guardrails and Feedback Loops]] (linters beat prompts), applied to multi-step workflows rather than single-step constraints. An `if` statement doesn't forget a branch; a prompt might.

> "Even if every section reaches its limit, the total injected content is kept within roughly 65K tokens."

The rebuild injection budget. This is the engineering answer to "how do you reconstruct context without just filling the window again?" Structured compression with hard caps per section.

> "When the number of steps exceeds 200, MiMo Code's win rate rises to over 65%."

The money quote from the human A/B test. This validates the entire architectural bet: MiMo Code's advantage isn't in short tasks (tied at 50%), it's in the long-horizon regime the architecture was built for. The fact that they ran a double-blind study with 576 developers on 474 real private repos is unusually rigorous for a product launch blog post.

---

## Key Themes

- **#concept** — Three time scales (computation/memory/evolution) as the organizing framework for agent architecture
- **#pattern** — Independent writer subagent for memory extraction: the main agent never touches its own memory
- **#pattern** — Early checkpointing (20/45/70%) as preventive maintenance, not emergency compression
- **#pattern** — Dynamic Workflow: orchestration logic as deterministic code, not ambiguous natural language
- **#pattern** — Goal verification: independent model checks completion conditions before allowing termination
- **#pattern** — File-based memory for human reviewability over vector DB opacity
- **#tool** — MiMo Code: terminal-based coding agent, MIT-licensed, built on OpenCode
- **#tool** — Dream (7-day memory merge) and Distill (30-day pattern extraction) as cron-based memory maintenance
- **#concept** — Context budget as an engineering constraint with explicit checkpoint triggers, not a soft limit

---

## Critical Analysis

**The writer pattern is genuinely novel.** I haven't seen another coding agent that completely removes memory management from the main agent's responsibility. [[Sawtooth Memory]] does async background compression but still feeds back into the main agent's context. [[Slate]] uses episodes as compression boundaries but the orchestrator still manages routing. MiMo Code's writer is fully independent — different model, different attention budget, different job. This is the right design and I expect it to be widely copied.

**The early-checkpoint argument is persuasive but unverified.** The logic (models degrade under load, so extract early) makes sense, but the article doesn't provide ablation data showing that 20/45/70% checkpoints outperform, say, a single checkpoint at 80%. The claim that delaying extraction is "exactly backwards" needs evidence beyond reasoning from "lost in the middle" papers. This is a falsifiable claim — someone should test it.

**Dynamic Workflow is both innovative and a partial admission of failure.** The fact that natural-language workflow descriptions "systematically fail in complex workflows" is an indictment of the entire SKILL.md/AGENTS.md approach. MiMo Code's solution — generate JavaScript that runs deterministically — is correct engineering. But it also means the model can't be trusted to follow instructions, which is a fundamental limitation of current LLMs that better prompting won't fix. The comparison to Anthropic's Dynamic Workflow is interesting; MiMo Code extends it with `workflow()` composability and disk persistence for recovery.

**The file-based memory choice is philosophically right and practically limiting.** Markdown files are reviewable, editable, and git-friendly — exactly what you want for developer trust. But they're also unstructured, hard to query, and don't support the kind of semantic retrieval that vector DBs enable. MiMo Code's four-layer hierarchy (session → project → global → history/SQLite) is smart: use files for structured, curated knowledge and SQLite for raw traceability. The tradeoff is that cross-project pattern transfer (Dream/Distill) is limited to what can be expressed in Markdown.

**The 200-step threshold is the most important number in the article.** MiMo Code is tied with Claude Code under 200 steps and wins beyond. This means: (a) for most real-world usage (quick fixes, small features), there's no advantage, (b) the architecture's value is in long-horizon tasks that most developers don't run yet, and (c) if MiMo Code can make long-horizon tasks accessible to more developers, it expands its own market. This is a bet that the future of coding agents is long sessions, not quick turns — and it's probably right.

**What's missing:** No open-source code link (the article says GitHub/MIT but I couldn't find a repo URL). No ablation studies for individual components (Max Mode vs. Goal vs. Writer vs. early checkpointing — which contributes how much?). The A/B test details are thin: what tasks, what duration, what failure modes? The "below 0.5% infinite loop probability" claim for Goal is suspiciously precise without methodology.

**The Claude Code comparison is both fair and strategic.** Using Claude Code as the baseline is honest — it's the market leader. But the finding that MiMo Code only wins beyond 200 steps means the comparison is specifically designed to show MiMo Code's strength. A fairer headline: "MiMo Code matches Claude Code on normal tasks, wins on very long ones." That's still impressive, just less dramatic.

---

## How It Connects

MiMo Code's memory architecture is the most fully-realized version of ideas that appear across this wiki:

- [[Coding Agents Continuity Not Memory]] argues continuity is the real primitive; MiMo Code's checkpoint/rebuild cycle IS continuity-as-architecture
- [[Slate]] shares the long-horizon focus and the insight that context management beats model intelligence, but Slate solves it with thread/episode routing while MiMo Code solves it with independent extraction
- [[Agent Memory and Context]] catalogs the memory landscape; MiMo Code is a new entry that combines file-based persistence (like [[napkin]]) with structured multi-layer hierarchy (like [[Three Tier Memory]]) and temporal maintenance (like [[NornicDB]]'s decay)
- [[Components of a Coding Agent]] argues the harness matters more than the model; MiMo Code is the most ambitious harness design I've seen
- [[Scaling Long-Running Agents]] discovered planner/worker/judge empirically; MiMo Code's writer/agent separation is the same pattern applied to memory instead of task decomposition
- [[Guardrails and Feedback Loops]] says "linters beat prompts"; MiMo Code's Dynamic Workflow says "code beats prompts" for orchestration — same principle, different domain
- [[MiMo-V2.5-Pro-UltraSpeed]] is the model powering it; the speed-to-intelligence argument (1000 tps enables Best-of-N reasoning) is Max Mode's underlying enabler

---

*Sources: [[raw/mimo-code-long-horizon]]*
*Last updated: 2026-06-11*
