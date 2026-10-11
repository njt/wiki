# Decompiling MW2 with Four Claude Agents

A reverse engineer runs four Claude Code agents for four straight weeks decompiling Call of Duty: Modern Warfare 2 (2009) — three workers on subsystems plus an overseer reviewing every commit — and reports candidly on what survives multi-week autonomous sessions: 7,000 commits, 199.8 billion tokens, 34% of functions done, and a fresh catalogue of long-horizon failure modes.

---

This is one of the most useful kinds of agent write-up: not a demo, but a *duration* report. Hour-long agent sessions are well documented; the author's thesis is that multi-day and multi-week sessions surface "a different class of problems," which he attributes mostly to compaction — agents forgetting what matters after the tenth summarization cycle.

**The topology that worked.** Three worker agents independently decompile different subsystems, all committing and pushing to the same branch; an overseer reviews every pushed commit, triggered by a GitHub webhook. Coordination is a shared Discord channel, with a separate webhook-fed channel for CI failures. Humans talk to the agents through Discord too, without shell access to the machines. Ghidra and IDA Pro are exposed via MCP servers, and leaked Xbox alpha builds with symbols and macOS ports with debug data give the agents ground truth. The task itself is "relatively easy" — "It's just an MCP call to IDA and then pasting that into a CPP file."

> "Initially, progress tracking used a STATUS.md file in the repository. That file quickly grew into a 10 MB+ giant, instantly overflowing the context when trying to ingest it. As a consequence, we switched to GitHub issues, which works great. Agents autonomously create, edit and close them."

The STATUS.md lesson is a textbook case of state that should live outside the context window: the artifact didn't scale with the project, and the fix was to externalize it to a system with its own indexing (GitHub issues) rather than to shrink it.

**What agents do wrong over weeks.** The failure list is the article's real contribution:

- **Yapping.** Agents are asked to be concise in Discord, comply for five minutes, then relapse into walls of text. PostCompact hooks that reinject rules fix issue management and status updates, but "staying concise with their messages or omitting redundant code comments is just something they don't respect. My best guess is that yapping, both in code and in messages, was just deeply embedded during their training."
- **Redundant verification.** Agents run the full local test suite on every change — four minutes of green each time — despite CI and instant failure notifications. "We have CI for that... Yet, they don't really care."
- **Quick-win bias.** Agents file an issue when they hit a big blocker and move to something small. Fine at first; once the low-hanging fruit is gone, it leaves agents idle. Avoidance of hard work looks like productivity in week one and like deadlock in week four.
- **Context paranoia.** Agents refuse to work "because they think they're running out of context. Obviously, they don't. They have no idea how many tokens are in their context" — the harness handles it. A model reasoning badly about its own internals becomes an operational outage.

**Sequencing over multitasking.** Asking agents to decompile, modernize C++, and improve portability simultaneously fails: "They either eventually forget one of those things after the 10th compaction, or start making mistakes. Pure mechanical decompilation first, and then modernization/portability afterwards is the way to go." One objective per long-running agent, phase transitions between objectives — the multi-week analogue of keeping prompts single-purpose.

**Model economics.** The author ran Opus 5 for two weeks, then switched to Sonnet: "It is slightly dumber, yet there is no noticeable difference." For a well-instrumented mechanical task, the cheaper model makes the same mistakes and gets distracted just as easily — capability budget belongs in the harness (MCP tooling, webhooks, hooks), not necessarily in the model.

#concept #tool #pattern

## Related pages

This source gives [[Giving Your Agent Eyes with Game Boy Hacking]] a production-scale sequel: that post's thesis — give an agent a feedback loop and reverse-engineering tools like Ghidra — is here validated at 16,324-function scale, with the addition that *multi-agent topology* (workers + reviewing overseer) is what sustains it for weeks.

It strengthens [[Automating Myself Out of Development]]: both converge on GitHub issues as the durable shared state for long-running agents, but this report adds the compaction diagnosis — it's not just that agents need external task state, it's that the tenth compaction is what erodes in-session rules.

It nuances [[Running an AI-Native Engineering Org]]: Fiona Fung's report is human-organisation-level; this is a single human orchestrating a machine-only team, and the bottlenecks that migrate (verification, review) reappear as *agent* behaviours — redundant local testing stands in for the human habit of over-checking.

And it complicates [[Maybe Coding Agents Don't Need a Bigger Memory]]: Santi argues continuity should be repo-local records rather than bigger windows; momo5502's PostCompact rule-reinjection is a third mechanism — not memory, not context, but periodic re-priming — that works for rules yet can't suppress training-deep habits like verbosity.

---
*Sources: [[raw/2026-08-17-mw2-decompilation]], [[summary/2026-08-17-mw2-decompilation]]*
*Last updated: 2026-10-11*
