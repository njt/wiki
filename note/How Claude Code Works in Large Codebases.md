# How Claude Code Works in Large Codebases

Anthropic's Applied AI team opens a new blog series, *Claude Code at scale*, by mapping the patterns behind successful enterprise deployments across multi-million-line monorepos and thousands of developers. The thesis: the harness around the model matters more than the model itself. Seven components (CLAUDE.md, hooks, skills, plugins, LSP, MCP servers, subagents) form the extension surface; three configuration patterns (navigable codebases, model-aware maintenance, ownership) determine whether deployments plateau or scale.

---

## The Harness, Enumerated

The article defines seven extensibility points with a comparison table that doubles as a design-decision matrix. The core insight: each component has a "common confusion" — a way teams misuse it — and the entire table reads as a catalog of mistakes Anthropic has observed in the field.

> "The most successful Claude Code deployments share a set of recognizable patterns"

This is the article in one sentence. The patterns aren't secrets — they're the outcome of watching what breaks when you skip them.

### Agentic Search vs. RAG

Claude Code does not embed your codebase. It navigates the filesystem with grep, file reads, and reference-following — an approach Anthropic calls "agentic search."

> "those systems can fail because embedding pipelines can't keep up with active engineering teams"

This is a shot at the RAG-everything default. The argument is practical, not theoretical: embeddings go stale the moment someone commits. In a codebase with thousands of engineers committing continuously, an embedding pipeline is a distributed systems problem you didn't need to solve. Agentic search sidesteps it entirely — but at the cost of needing good starting context. The article is honest about this tradeoff.

### Component Comparison: What Goes Where

The table is the article's most useful artifact. Seven rows, each with "Common Confusion" — a column that reads like a postmortem taxonomy:

| Component | Common Confusion |
|-----------|-----------------|
| CLAUDE.md | Using it for reusable expertise that belongs in a skill |
| Hooks | Using prompts for things that should run automatically |
| Skills | Loading everything into CLAUDE.md instead |
| Plugins | Letting good setups stay tribal |
| LSP | Assuming it's automatic |
| MCP servers | Building MCP connections before basics work |
| Subagents | Running exploration and editing in same session |

The through-line: teams overstuff CLAUDE.md, underuse hooks and skills, and skip LSP because they assume Claude can "just read the code." Each confusion is a failure mode Anthropic has seen enough times to name.

---

## Three Patterns That Matter

### 1. Make the Codebase Navigable

The most actionable section. Six concrete practices:

- **Lean CLAUDE.md** at root — "pointers and critical gotchas only; everything else drifts into noise"
- **Layered CLAUDE.md** in subdirectories — Claude walks up the tree loading every one
- **Scoped commands** per subdirectory — running the full test suite for one service change "causes timeouts and wastes context"
- **Version-controlled `.ignore` files** — commit deny rules so exclusions survive across machines
- **Codebase maps** — a root markdown file listing folders with descriptions, like a table of contents
- **LSP running** — grep returns thousands of matches for common function names; LSP returns only references to the same symbol

> "Claude searches by symbol, not by string"

LSP is the silent differentiator. Without it, Claude is pattern-matching on text — good enough for unique names, disastrous for `get()` in a monorepo. With it, Claude has IDE-level symbol resolution. The article treats LSP as infrastructure, not a feature: you need it running, and it's not automatic.

### 2. Maintain CLAUDE.md as Models Evolve

> "Teams should expect to do a meaningful configuration review every three to six months"

This is the most underappreciated point. Instructions written for today's model can become constraints on tomorrow's. A rule that helped Claude 4.5 navigate a pattern might make Claude 4.7 *worse* at it. The article recommends reviews after major model releases and whenever performance plateaus — essentially, treat CLAUDE.md like a dependency that needs version bumps.

### 3. Assign Ownership

> "knowledge will stay tribal and adoption will plateau"

Without a DRI, the configuration drifts. The emerging role of **agent manager** — hybrid PM/engineer owning the ecosystem — is the minimum viable governance structure. For regulated orgs, the recipe is: approved skills, required code review, limited initial access, cross-functional working groups with engineering, infosec, and governance.

The fastest rollouts didn't start with broad access. They started with a small team wiring up tooling so Claude "already fit workflows on first use." Infrastructure before adoption.

---

## Critical Analysis

**What it gets right:** The article is unusually honest for a vendor blog. It admits edge cases (codebases with hundreds of thousands of folders, non-git version control, game engines with binary assets), names its own product's limitations, and doesn't pretend the harness is simple. The "common confusion" column in the component table is genuinely useful — it's not marketing, it's pattern recognition from field failures.

**What's missing:** The article is a map, not a manual. "Build codebase maps" is good advice, but there's no example. "Run LSP" is critical, but how do you verify Claude is actually using it? The three-to-six-month CLAUDE.md review cycle is right but vague — what does a review actually look like? The article names the patterns without showing the work.

**The subtext:** This is Anthropic telling enterprises "your setup matters more than our model." That's a remarkable claim from a model vendor. The implicit message: GPT-5 or Gemini 3 won't fix your deployment — only your harness will. The article reads as both honest engineering communication and a moat-building exercise. If the harness is the differentiator, switching models gets you less than switching harnesses.

**The gap between this and reality:** The "agent manager" role is interesting but aspirational. Most orgs are nowhere near needing a dedicated person — they have one enthusiastic engineer maintaining a sprawling CLAUDE.md that nobody else reads. The article describes the destination; the journey from tribal knowledge to managed ecosystem is the hard part, and it's left for future installments.

---

## Related Pages

- [[A Guide to Claude Code 2.0]] — Deep tour of the features this article configures
- [[How Intercom Uses Claude Code]] — The most comprehensive enterprise deployment: 13 plugins, 100+ skills, hooks, OpenTelemetry
- [[Components of a Coding Agent]] — The harness-matters-more-than-the-model taxonomy this article operationalizes
- [[CLAUDE.md (Universal)]] — Token-efficient CLAUDE.md rules for the "lean and layered" approach
- [[Writing a Good CLAUDE.md]] — HumanLayer's guide to brevity as an instruction budget
- [[Anatomy of the .claude/ Folder]] — Every directory and file, including the settings.json for `.ignore` rules
- [[Intent Layer]] — Hierarchical AGENTS.md at folder boundaries: the same layering concept
- [[Harness Engineering]] — Böckeler's feedforward/feedback framework: the theory behind "linters beat prompts"
- [[Harness Engineering (OpenAI)]] — The original experiment: 1M lines, zero handwritten code, 12 concrete practices
- [[Scaling LLMs to Larger Codebases]] — Gill's framework for where to invest engineering resources
- [[How Boris Uses Claude Code]] — The creator's own workflow
- [[claude-ctrl]] — Enforcement via hooks and SQLite, not prompts
- [[claude-code-config (Trail of Bits)]] — Security-conscious defaults
- [[Feedback Loop is All You Need]] — Linters beat prompts: the self-tightening loop
- [[Gas Town After 10,000 Hours of Claude Code]] — Hartcher's field experience with the tool this article configures
- [[Agent Memory and Context]] — Context management as the real engineering challenge
- [[Agent Orchestration]] — Multi-agent patterns including the planner/worker/judge topology
- [[Scaling Long-Running Agents]] — Cursor's planner/worker/judge finding: flat self-coordination fails

---
*Sources: [[summary/how-claude-code-works-in-large-codebases]]*
*Last updated: 2026-05-16*
