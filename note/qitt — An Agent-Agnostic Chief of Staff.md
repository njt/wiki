# qitt — An Agent-Agnostic Chief of Staff

Benoît Quenin's open-source qitt is a proactive chief of staff that watches his Slack and JIRA, triages everything before he sees it, spins up research workers, and leaves drafts for him to send. Its two design rules — never send on the human's behalf, and build only from T3 Code constructs — make it one of the cleanest published examples of a personal agent that is proactive about the work but never about the person.

---

## What it argues

The first qitt (built on OpenClaw, Discord-fronted, with long-term memory and indexes of Slack, JIRA, GitHub, Drive and email) failed for a reason worth generalising: it was reactive and in the wrong place. "A chief of staff shouldn't wait for me to come and ask. It should come to me." Two months and 280 commits produced a tool he almost never used.

The rebuild rests on two rules, both written down on day one:

**Proactive about the work, never about the voice.** The Slack send tools are *blocked*, so neither qitt nor its workers can post — a tool-level constraint, not a prompt-level one. "I'll be fully in control, qitt doesn't send reply for me, just be proactive and let me review." It also never settles a thread.

**Only T3 Code constructs.** The chief of staff is a single pinned thread with an AGENTS.md charter; workers are ordinary threads that land in the sidebar; triggers are scheduled tasks (Slack webhook, JIRA webhook, weekday sweep). Outside the harness there is only a small Slack connection bridge.

The vendor-agnosticism is the headline but almost a side effect of where it lives: worker threads run on Claude, Codex, or Cursor because those are what T3 Code drives — "switching is a setting, not a migration."

## Key quotes

> "A chief of staff shouldn't wait for me to come and ask. It should come to me."

The diagnosis of v1's failure. Most personal agents die of this: they are chat interfaces pretending to be assistants. Placement (where does it live relative to your actual work) and proactivity (does it act unprompted) are the two variables, and Quenin changed both.

> "Proactive about the work, never about my voice."

A sharper boundary than most agent designs draw. The distinction isn't autonomy-level, it's *what kind* of action: research, routing, drafting are all delegated; expression and closure are not. "It doesn't speak for me either" — and the enforcement is mechanical (blocked tools), not aspirational.

> "It doesn't do my job for me. It gets me to the part of my job that needs me, faster."

The honest framing missing from most proactive-agent pitches: he still read all 42 reports that Friday morning, decided on each, and wrote or edited every outgoing reply. What was automated was the pre-work — noticing, classifying, gathering context.

## Key themes

#pattern #personal-agents #guardrails #orchestration

**Draft-only autonomy as a general pattern.** Blocking send tools is the strongest form of the review gate: not "please ask permission" but "no path exists." This is the tool-enforcement end of the spectrum that [[You Shall Not Pass — Where Developers Draw the Line on AI Autonomy]] surveys, and it sidesteps the ratchet problem — there is no convenience gradient toward full autonomy because the send tool was never granted.

**The chat-location lesson.** v1's failure is a placement argument: the agent must live where the work streams in, not in a separate chat app. It also strengthens [[Chief of Staff]] (De Jesus), which reaches a compatible conclusion from the other direction — that agent's lesson was tiered processing and graduated trust; qitt's is placement and non-sending. Together they suggest the chief-of-staff pattern is converging on "watch everything, act early, never speak for the human."

**A pinned thread as control plane.** The architecture is all reuse: one pinned thread, ordinary worker threads, scheduled-task triggers. It complicates [[Mission Control — Bhanu's 10-Agent Squad on OpenClaw]], which needed a shared Convex backend, heartbeats and a Kanban board for a ten-agent squad; qitt coordinates its workers with nothing but the harness's native thread primitives and gets to ~940 triaged items in three weeks. The minimal version may be the more reproducible one.

**Numbers worth trusting.** Three weeks, every message sent by a human, ~15 items triaged per message he typed. The failure mode reporting is unusually candid (started duplicate threads, researched the wrong question for an afternoon, missed messages), and the maintenance loop is itself agentic: complain in the pinned thread, and qitt proposes or builds the fix.

## Critical analysis

The strongest thing here is the rule-one enforcement. Everyone writes "the agent never sends for me" into prompts; almost nobody blocks the tool. Quenin's version is falsifiable by construction — if a message goes out, a rule was broken in code, not in interpretation.

The weakest spot is the vendor-agnostic claim, which is really a T3 Code dependency wearing a general slogan. "Whatever T3 Code supports next" is a bet on one harness's adapter surface, and the article doesn't test the claim — everything today runs on Claude. The agnosticism that matters in practice is not model-agnosticism but *event-source* agnosticism (Slack + JIRA today, extensible later), which is where the interesting engineering actually lives.

There's also an unexamined asymmetry in the numbers: 367 items closed autonomously as "needing nothing" — mostly acks, but the criterion for *not* acking is exactly the judgment call a silent failure would hide. He acknowledges reports are "sometimes wrong, or answer a slightly different question"; a misclassification rate on the 367 would be the number I'd want next.

---
*Sources: [[raw/agent-agnostic-chief-of-staff]], [[summary/agent-agnostic-chief-of-staff]]*
*Last updated: 2026-11-11*
