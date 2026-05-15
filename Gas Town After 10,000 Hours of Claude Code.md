# Gas Town After 10,000 Hours of Claude Code

Simon Hartcher's skeptical field assessment of Steve Yegge's Gas Town multi-agent system, written after a year of intensive daily use of Claude Code in pair-programming mode. Hartcher finds Gas Town impressive as engineering but rejects its core premise: that the developer should delegate to agents rather than collaborate with them. His critique lands on three concrete objections — visibility loss, token-speed sluggishness, and beads state polluting git history — but the underlying tension is philosophical. He still wants to see the code.

---

## Key Quotes

> "I know that there are not 10,000 hours in a year. I've been living inside Claude Code and it feels like a lifetime."

The opening line discloses the depth of his practice. This isn't a casual user's hot take; it's earned skepticism from someone who's pushed the current tools to their limits.

> "I feel as though I have more agency when I work this way"

Hartcher describes pair programming with Claude Code — varying between driver and observer depending on the task — as giving him *more* agency than delegating to agents. This inverts the pitch of Gas Town, which promises agency through delegation. Hartcher's counterclaim: delegation *removes* agency by removing visibility.

> "it feels like deferring everything to agents, and I get almost no visibility of what's going on other than work was completed"

Requesting explanations from "the mayor" feels "very cumbersome" to him. This is the core UX complaint about multi-agent systems: the coordination layer becomes an opaque intermediary. You know work finished; you don't know what tradeoffs were made.

> "the whole process just seems really slow"

He attributes this to "the token speed of current agents," specifically Claude Opus 4.5. Multi-agent orchestration amplifies the latency problem: if one agent is slow, serial multi-agent chains are glacial. Gas Town's architecture turns individual agent latency into system-level slowness.

> "agents need contracts"

Hartcher agrees with Yegge on the diagnosis — agents lack human intuition for task ordering and need explicit dependency graphs. Beads represents dependencies as a DAG (A blocks B). This maps to the planner/worker/judge pattern that [[Scaling Long-Running Agents]] found critical for coordination at scale.

> "those changes pollute every PR you or an agent makes"

His sharpest technical criticism. Beads stores dependency state in git alongside application code — meaning every PR carries agent bookkeeping artifacts. Hartcher argues this should be separated: if git is the right medium, the state should live in a different repo or layer. This is [[Prefix Effects]] applied to agent infrastructure: naming decisions (where state lives) create gravity that shapes all downstream artifacts.

> "Even if I am vibe engineering, I still care about the code. I still look at it."

A direct rebuttal to Yegge's admission: "I've never seen the code, and I never care to, which might give you pause." Hartcher's line is the [[Write Only Code]] tension crystallized in one sentence. He's doing the thing Yegge champions (delegating code generation), but he refuses the logical endpoint Yegge embraces (not reading the output).

---

## Key Themes

#concept **Pair programming as the middle path.** Hartcher's workflow sits at Level 2-3 of the [[Five Levels from Spicy Autocomplete to the Dark Software Factory]] — AI generates, human reviews everything. Gas Town aims for Level 5. The disagreement isn't about whether Level 5 is possible; it's about whether it's desirable. Hartcher likes reading code.

#concept **Visibility as agency.** The core objection to Gas Town isn't technical — it's experiential. When you can't see how decisions were made, you lose the ability to override them. [[Agent Orchestration]] research confirms this: kanban boards emerged as coordination surfaces precisely because humans need to see what's happening.

#tool **Beads as git-polluting middleware.** Yegge's dependency tracker is clever — it gives agents a contract for what blocks what. But storing state in the same git repo as application code means agent bookkeeping artifacts leak into every PR. Hartcher's instinct that this should be "separated from the code" echoes best practices from [[Managing Agents via Kanban Boards]] and [[workgraph]], where task state lives in its own persistence layer.

#tool **Claude Opus 4.5 as both enabler and bottleneck.** Hartcher upgraded from $100 to $200/month Claude Max for access. The model is capable enough to make Gas Town work *in principle*, but slow enough to make it frustrating *in practice*. Multi-agent systems amplify model latency. The gap between what's architecturally possible and what's pragmatically enjoyable is still wide.

#person **Steve Yegge as the dark-factory evangelist.** Yegge's "never seen the code" stance is the purest expression of [[The Dark Factory is a DOT File]] philosophy — the code is disposable, the pipeline is the artifact. Hartcher's pushback ("I still look at it") represents the practitioner's resistance to that endpoint.

#person **Simon Hartcher as the skeptical practitioner.** He's not a Luddite — he's lived inside Claude Code for a year and calls it "a lifetime." His critique carries weight because it comes from deep usage, not abstract discomfort. He's willing to say that in mid-2025, the agentic future isn't ready, even if he can see where it's going.

---

## Critical Analysis

Hartcher's essay is valuable *because* it's a practitioner's reaction, not a theoretician's. He tries Gas Town — actually uses it — and reports what bothers him. That's rarer than it should be in this space.

But his objections may be temporary rather than fundamental. The speed complaint ("the whole process just seems really slow") is a function of current Opus 4.5 token rates, not an architectural flaw. If models get 10x faster (and they will), this objection evaporates. The visibility complaint is harder — multi-agent systems will always be less scrutable than single-agent ones — but better observability tooling could close the gap. [[How Intercom Uses Claude Code]] shows what happens when you add OpenTelemetry to agent workflows: visibility becomes tractable.

The beads-in-git complaint is the most durable objection. It's a real architectural question: should agent state live alongside application code? Hartcher says no. The kanban-based approaches ([[weft]], [[ralph-ban]]) say no too — they use separate SQLite databases or Cloudflare Durable Objects. But there's an argument for colocation: if agent state is versioned alongside code, you can replay exactly what an agent saw at any point in history. That's not nothing.

The most interesting thing Hartcher *doesn't* say: he doesn't claim pair programming produces better code than Gas Town. He claims it *feels better*. That's a UX argument, not a quality argument. If Gas Town ships better software but feels alienating, does that matter? For Hartcher, clearly yes. For a startup burning runway? Probably not. The right workflow depends on whether you optimize for developer experience or output velocity, and those two things have just started to diverge.

---

*Source: [[raw/gas-town-after-10000-hours-claude-code]]*
*Last updated: 2026-05-15*
