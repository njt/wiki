# Mission Control — Bhanu's 10-Agent Squad on OpenClaw

Bhanu Teja (SiteGPT.ai) built a 24/7 AI agent squad of 10 specialized agents on OpenClaw, coordinated through a shared Convex backend with file-based persistent memory, 15-minute heartbeat loops, and a Kanban workflow. It's the most detailed published reference architecture for a production multi-agent team on a personal agent framework.

---

## Key Quotes

> "If you want to remember it, write it to a file."

The entire memory architecture collapses to this one rule. Five file types (JSONL sessions, WORKING.md, daily logs, MEMORY.md, SOUL.md) each serve a distinct retrieval pattern. There's no vector DB, no RAG — just files and grep. It works because the retrieval surface is small and the write discipline is enforced by the heartbeat loop.

> Each agent has a unique personality and professional role — content writer, SEO analyst, designer, developer — and they work together like a mini startup that operates 24/7.

This isn't just role-playing. The roles create information asymmetry: the SEO analyst sees things the content writer doesn't, which creates genuine collaboration rather than echo-chamber agreement. The squad structure forces perspective divergence before convergence.

> We stagger the heartbeats. Agent 0 fires at :00, Agent 1 at :02. If all 10 fired simultaneously, the cost spike would be brutal.

The most practically useful technical detail in the whole thread. Staggered cron is the kind of thing you learn in week two of running a multi-agent system and wish someone had told you in week one.

> Start with 2-3 agents. Validate the memory → coordination → review loop first.

The scaling advice is honest: the coordination overhead grows non-linearly. Most multi-agent demos skip straight to "10 agents!" without acknowledging that 3 agents will expose every architectural flaw.

---

## Key Themes

- **#pattern** — File-based persistent memory as the simplest possible agent memory architecture. No databases, no vectors — just markdown files and disciplined writes.
- **#pattern** — Heartbeat-driven autonomy. Agents don't run continuously; they wake, check, work, and sleep. This is both cost-efficient and architecturally cleaner than long-lived loops.
- **#pattern** — Staggered scheduling. The "stagger your cron jobs" insight applies to any multi-agent system but almost nobody documents it.
- **#tool** — OpenClaw (formerly [[clawdBot]]) as the agent runtime. Session management, cron, WebSocket API, and multi-platform messaging in one binary.
- **#tool** — Convex as the shared collaboration substrate. Realtime serverless DB acting as the team's collective working memory, distinct from each agent's private file-based memory.
- **#concept** — Role specialization as information asymmetry. Agents don't just have different job titles; they have different SOUL.md files that shape what they notice and how they interpret. This is the mechanism that makes multi-agent collaboration produce better output than a single agent.
- **#concept** — Autonomy levels (Internal/Specialist/Lead) as a graduated trust model. Not all agents get the same permissions, and the escalation path is explicit.

---

## Critical Analysis

**What's genuinely new here.** Most multi-agent architectures are theoretical or toy-scale. Bhanu's is production — it runs his actual business (SiteGPT.ai). The file-based memory system is elegant in its simplicity: five file types, each with a clear purpose, no infrastructure beyond the filesystem. The heartbeat + staggered cron pattern is battle-tested advice that almost nobody publishes.

**What's missing.** The thread is a architecture overview, not an incident post-mortem. We don't know what breaks. What happens when two agents make conflicting edits to the same Convex document? When the heartbeat fires but the agent's WORKING.md is corrupted? When an agent hallucinates an @mention? The happy path is well-documented; the failure modes are invisible.

**The Convex dependency is doing a lot of work.** File-based memory handles individual agent state, but inter-agent coordination — task boards, @mentions, notifications — all flows through Convex. That's a realtime serverless database with its own latency characteristics, consistency model, and failure modes. It's not a trivial dependency. Anyone cloning this architecture should understand what Convex provides that a shared SQLite file or a Git repo wouldn't.

**The autonomy taxonomy is under-specified.** Internal/Specialist/Lead is a clean three-level model, but the thread doesn't explain how the system decides which level an agent operates at for a given task. Is it role-based? Task-based? Does Jarvis (Squad Lead) manually set autonomy levels, or are they encoded in the task schema? Without this detail, it's hard to assess how much human intervention the system actually requires.

**Comparison to other multi-agent patterns.** This is a concrete implementation of the planner/worker/judge pattern described in [[Scaling Long-Running Agents]], with Jarvis as planner/coordinator and the specialists as workers. The heartbeat loop echoes [[MimiClaw]]'s HEARTBEAT.md pattern at a larger scale. The file-based memory maps directly to [[Elements of Agentic Systems Design]]'s Context and Memory elements. But where [[maestro]] uses a PM/Architect/Coder queue model, Mission Control uses a flat squad with one lead — more startup-org-chart than assembly line.

**The real innovation is the Convex-backed shared workspace.** Individual agent memory (SOUL.md, MEMORY.md) is well-established — [[MimiClaw]] does it, [[clawdBot]] does it, [[Hermes]] does it. What Mission Control adds is a *shared* collaboration layer where agents can see each other's tasks, @mention each other, and subscribe to threads. That's the difference between 10 agents working in parallel and 10 agents working as a team.

**Bottom line:** This is the best published reference architecture for a production multi-agent team on a personal agent framework. The file-based memory and staggered heartbeat patterns are immediately stealable. But the inter-agent coordination layer (Convex + Kanban + @mentions) is the hard part, and the thread raises more questions than it answers about failure modes and autonomy gating.

---

*Sources: [[raw/mission-control-bhanu]], https://x.com/pbteja1998/status/2017662163540971756*
*Last updated: 2026-05-15*
