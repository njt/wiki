# Paperclip

Paperclip is an open-source, self-hosted orchestration layer for running a "zero-human company" — hire AI agents into an org chart (CEO, CTO, engineers, marketers), point them at a single company goal, and govern the result as the board of directors. Its distinguishing move is deliberate anti-framework positioning: it doesn't build agents, write their prompts, or choose their models; it models the *company* they work in, with org charts, per-agent budgets, goal inheritance, tickets, and approval gates as the primitives.

---

## Key Quotes

> "Hire AI employees, set goals, automate jobs and your business runs itself."

The thesis in a sentence, and it's the exact promise [[The Dead Economy Theory]] treats as the endgame of AI acceleration — productive capacity that "hums along without needing human participation." Paperclip is that idea shipped as an installable product, with the one deviation that a human *board* remains in the loop.

> "If it can receive a heartbeat, it's hired."

The bring-your-own-agent contract. Claude Code, OpenClaw, Codex, Cursor, shell commands, webhooks — Paperclip is unopinionated about runtimes. This is the same heartbeat primitive as [[Moltbook]]'s, but pointed at an internal org chart with budgets and an audit log rather than at an untrusted remote skill file. Same mechanism, opposite threat model.

> "Autonomy is a privilege you grant, not a default."

The governance stance, and the sharpest line on the page. Agents can't hire agents without board approval; the CEO can't execute an unreviewed strategy; config changes are revisioned and rollbackable. Paperclip keeps the human as the board — which is precisely the layer that pure "zero-human company" rhetoric usually deletes.

> "When they hit it, they stop. Automatically. No runaway costs. No surprise bills. Hard limits, enforced by the system."

Cost control as a first-class primitive, not an afterthought. Monthly per-agent budgets with atomic enforcement at checkout — the "runaway loop wastes hundreds of dollars of tokens" failure mode, made structurally impossible rather than admonished against.

> "We don't tell you how to build agents. We tell you how to run a company made of them."

The "What Paperclip is not" section is a masterclass in negative positioning: not a chatbot, not an agent framework, not a workflow builder, not a prompt manager, not a single-agent tool. "If you have one agent, you probably don't need Paperclip. If you have twenty — you definitely do."

## Key Themes

- **#concept** — The company as the unit of abstraction. Paperclip's object model is not tasks or pipelines but *organizations*: hierarchies, roles, reporting lines, goals, budgets.
- **#pattern** — Heartbeats as the scheduling primitive. Agents wake on a schedule or on notification (ticket assignment, @-mention) and delegate up and down the org chart automatically.
- **#pattern** — Governance with rollback. Approval gates, revisioned config, and the board as the top of the hierarchy — human-in-the-loop as a designed feature, not a safety patch.
- **#tool** — Bring-your-own-agent orchestration. Adapters to any runtime that can receive a heartbeat, sidestepping the framework wars entirely.
- **#concept** — Atomic budget enforcement. Task checkout and budget spend are one atomic operation, so no double-work and no runaway spend — the deterministic fix for a failure mode most orchestration tools merely warn about.

## Critical Analysis

**This is the org-science thesis turned into a product.** [[Multi-Agent AI Systems Are Organizations]] argues that multi-agent systems fail not from implementation bugs but from four universal organizing problems — task division, allocation, information, reward — and that they must be designed as organizations, not architectures. Paperclip *is* that prescription: its entire feature list (org charts, roles, reporting lines, goal alignment, budgets, governance, audit) is the four-problem framework rendered as a deployable runtime. Where the paper calls for a research program, Paperclip ships a Node.js process. The intellectual lineage is direct and unacknowledged on the landing page.

**"Zero-human company" is a deliberately provocative tagline, and the product quietly contradicts it.** The copy leads with "zero-human companies," but every governance feature keeps a human at the top: approve hires, review strategy, override budgets, pause or terminate any agent. What's actually being automated away is human *employees* — the board (and the budget-holder) remains. That's a meaningful difference from the fully-autonomous framing, and it's also the product's most defensible claim: it sells delegation with a kill switch rather than full autonomy.

**The bring-your-own-agent stance is smart positioning that also outsources the hard part.** By refusing to build agents, Paperclip avoids competing with Claude Code, OpenClaw, and the rest — it rides them. But it also means the safety and reliability of any given "employee" is someone else's problem ("your agents are your own and you secure them however you want to"). The governance layer Paperclip adds is real, but it governs *Paperclip* — who can hire, what strategy runs, what budget exists — not what an agent actually *does* inside its own runtime. That's a meaningful gap the landing page waves at but doesn't close.

**Where it slots into the orchestration landscape.** [[Agent Orchestration]] notes the field has mostly solved hierarchical role separation and is converging on kanban boards and atomic task claiming, but that governance, cost-control measurement, and multi-player orchestration remain largely unsolved. Paperclip is one of the few tools attacking exactly those gaps — atomic budget enforcement, revisioned governance, multi-company isolation. Whether it holds up against the [[Paca]]-style "agents as Scrum teammates" model or the [[Fleet Supervisor (sermakarevich)]]-style dumb-supervisor model is an open, empirical question; the landing page is a pitch, not evidence.

**The unresolved question is the same one the whole category faces.** [[OpenViktor]] was an "AI employee" that launched to #3 on Product Hunt, then was killed and rebuilt — the category's demand signal is real, but its execution risk is high. Paperclip's answer to that risk is to be boring infrastructure (a single Node process, embedded Postgres, MIT license) rather than a magical employee. That's the right instinct. Whether "run a company made of agents" survives contact with actual multi-week agent autonomy — drift, spec creep, the comprehension-debt problems [[Principal Drift]] names — is what a real deployment would test.

---

*Sources: [[raw/paperclipai-net]], [[summary/paperclipai-net]]*
*Last updated: 2026-09-11*
