# recursive-mode

An installable skill package that gives coding agents a file-backed, phase-gated development workflow -- requirements through closeout -- where every decision, plan, and outcome lives in numbered markdown documents inside the repo, not in ephemeral chat history. It directly attacks [[Context Rot]] by making the filesystem, not the context window, the source of truth.

---

## Key Quotes

> "Long-running agent work has a common failure mode: requirements, decisions, and plans live in the conversation. Once that session ends or the context window overflows, the agent loses track of what was decided, what was implemented, and why. This is context rot."

The clearest naming of the problem that [[Planning With Files]], [[Claude-Mem]], and [[Agent Memory and Context]] all circle around. recursive-mode's contribution is not diagnosing this -- everyone knows it -- but proposing a rigid, phased cure.

> "Each development phase produces one locked output document. Each phase uses the previous phases' output as its input."

This is the core mechanism. Locked outputs prevent the drift that plagues looser approaches. Compare with [[Trycycle]], which also uses phase separation but resets context at each stage rather than building on locked artifacts.

> "Audited phases loop through draft -> audit -> repair -> re-audit until the work is genuinely ready. Closeout phases feed validated lessons back into decisions, state, and memory so future runs start from better context."

The recursion is not metaphorical. Each phase consumes earlier artifacts, and the closeout feeds lessons back into the system's memory for the next run. This is the [[Compound Engineering]] loop made explicit and file-backed -- the "Compound" step that Klaassen describes as the differentiator, here enforced by structure rather than discipline.

> "The run docs together with the code diff in the worktrees become a rich dataset for auto-training or finetuning a model against your codebase."

A provocative secondary benefit. If every development decision is documented alongside the code it produced, you have a natural training corpus. Connects to [[Self-Distillation]] -- the model improving from its own structured outputs.

> "Chat is used the way it should be, for commands only. Keep valuable information out of chat and in docs."

The philosophical core. Chat is CLI; docs are memory. This is [[Planning With Files]]'s "context window = RAM, filesystem = disk" metaphor taken to its logical conclusion and enforced through workflow structure.

## Key Themes

#tool #agentic-coding #workflow #context-management #traceability #spec-driven

**Phase-gated development.** The numbered document structure (00-requirements through 05-manual-qa) creates a linear progression with exit criteria at each gate. This is closer to traditional software engineering phase gates than to the freeform agent coding most tools enable. The `.recursive/` folder structure:

```
.recursive/
├── memory/              # Structured memory bank
├── RECURSIVE.md         # Canonical workflow spec
├── STATE.md             # Current repository state
├── DECISIONS.md         # Decisions ledger
├── run/00-my-first-requirements/
    ├── 00-requirements.md
    ├── 01-as-is.md
    ├── 02-to-be.md
    ├── 03-implementation-summary.md
    ├── 04-test-summary.md
    └── 05-manual-qa.md
```

**Skill decomposition.** Ships six skills: core workflow orchestration, git worktree isolation, structured debugging, TDD with RED/GREEN evidence, review bundling for delegated reviews, and subagent handoff contracts. Each is independently installable via `npx skills add`. The worktree skill connects directly to [[Agent of Empires]]'s git worktree integration pattern. The subagent skill addresses the control problem that [[Scaling Long-Running Agents]] identified -- hierarchy over flat coordination.

**Missions alternative.** Positions itself as a free, open-source alternative to Factory.ai's Missions feature, predating it by several months. Works across IDEs, CLIs, agents, and models -- not locked to a single platform.

## Critical Analysis

**What's strong:** recursive-mode is the most opinionated and complete file-backed workflow for agent development I've seen. Where [[Planning With Files]] gives you three files and hooks, recursive-mode gives you a full phase-gated pipeline with numbered artifacts, exit criteria, and a memory layer that persists across runs. The DECISIONS.md ledger is particularly smart -- it's the institutional memory that [[Specifications as the Product]] identifies as missing from most agent workflows. The explicit "draft -> audit -> repair -> re-audit" loop is the kind of mechanical enforcement that [[Guardrails and Feedback Loops]] argues beats prompts every time.

**What's missing:** The introduction sells the philosophy well but is light on evidence. No benchmarks, no case studies, no "we ran recursive-mode on project X and here's what happened." Compare with [[Building low-level software with only coding agents]] (38K lines, 900+ tests, $2,871) or [[Scaling Long-Running Agents]] (1M+ lines in a week). Without concrete results, recursive-mode is a compelling design document, not yet a validated methodology.

The rigidity is a double-edged sword. Six numbered phases with exit criteria works for greenfield features but could be suffocating for bug fixes, spikes, or exploratory work. The page mentions "addenda" documents but doesn't explain when you'd break the linear flow. Real development is messier than requirements-through-QA suggests.

There's also a tension between "works with any model" universality and the deep structural assumptions about how agents interact with files. An agent that doesn't naturally work with file-based state (some chat-first tools) would need significant adaptation.

**How it connects:** recursive-mode sits at the intersection of several wiki threads. It's [[Planning With Files]] grown up into a full methodology. It's [[Compound Engineering]]'s compounding loop made file-explicit. It addresses [[Context Rot]] by design. It's what [[Specifications as the Product]] implies when it says the spec is the durable artifact -- here, every phase produces a durable artifact. The phase-gated approach echoes [[Verbose Deployment]]'s composable pipeline philosophy, applied to development rather than deployment.

The most interesting comparison is with [[Trycycle]]. Both use staged phases with review loops. But Trycycle deliberately wipes context between stages ("fresh eyes"), while recursive-mode deliberately preserves it (each phase reads the previous phase's locked output). These are opposing theories of how to handle accumulated context -- and both claim to solve the same problem. A head-to-head comparison would be illuminating.

---
*Sources: [[raw/recursive-mode]]*
*Last updated: 2026-05-14*
