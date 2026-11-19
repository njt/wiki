# Getting Out of the Loop — Lydia Hallie on Claude Code's New Best Practices

This is a talk by Lydia Hallie of Anthropic's Claude Code team, recorded as a make-up after technical difficulties at a live event, arguing that the Claude Code best practices of early 2025 are now obsolete — and worse, harmful. It matters because it is the tool's own team telling you to stop doing the things its own earlier guidance told you to do: stop steering, stop approving, stop stuffing CLAUDE.md with emphasis. The claim is falsifiable in a specific way: if emphatic instructions and human approval still improved outcomes with current models, her three-layer replacement (model, harness, environment) would be over-engineering.

---

## The argument in one paragraph

Human oversight of coding agents should be almost entirely replaced by three layers — the model (which now steers itself and verifies its own output), the harness (auto mode, deny rules, hooks, CI), and the environment (sandboxes and cloud sessions) — because the models got good enough that supervision is not just unnecessary but degrades results. The evidence offered is a change in session shape at Anthropic (dozens of parallel multi-hour sessions, orchestrating each other, 2,500 to 30,000+ changes per day between May and September), a study showing auto mode's classifier blocks dangerous commands at 89% versus 14% for humans in the loop, and the observation that emphatic CLAUDE.md instructions now push models off track. If Hallie is wrong, either the 89% figure hides unacceptable residual risk, or the verification points she proposes don't actually catch what a human in the loop used to catch.

## Key quotes

> "We've now essentially become managers for dozens of sessions."

The cleanest statement of the session-shape shift: the developer's role changed from operator to manager, and everything else in the talk follows from that premise.

> "So getting out of the loop is easy. But the harder question now is how do we make sure it's okay that we're not there anymore?"

She correctly identifies that leaving the loop is the trivial half of the problem; the entire safety-and-verification architecture exists to answer this question, and she's honest that it's the hard one.

> "Only about 14% of people caught a clearly dangerous command. Auto mode blocked 89% of them."

The most quotable statistic in the talk, and the one doing the most work: it reframes human approval not as a safety feature but as an 86%-failure-rate rubber stamp. Note what she concedes immediately after — 1 in 10 still gets through.

> "I think it kind of just comes down to, if Claude gets this wrong once, can I take it back?"

The decision heuristic for what needs a deny rule versus what the model can be trusted with. Revertibility as the dividing line between "leave it to the model" and "enforce in the harness" is the most transferable idea in the talk.

> "This machine is disposable, but the things that leave it aren't."

The cloud-session security model in one line: containment is about egress, not the machine. A commit pushed or a message sent through a connector is the moment the sandbox stops mattering — which is why deny rules travel in the repo.

## Critical analysis

The strongest move here is the permission blind spot. Hallie's point that `npm test` looks harmless but its dependencies — or a script Claude itself wrote — can read your `.env` and exfiltrate it, and that a human would have clicked allow anyway, is the most honest thing in the talk. It dissolves the intuition that human approval is a meaningful safety layer: permissions decide whether a tool call runs, not what it can reach once running. That's an argument for OS-level enforcement ([[A Deep Dive on Agent Sandboxes]]) that lands harder coming from the vendor than from a third-party reverse-engineer.

The weakest move is the statistics. 89% versus 14% is presented as auto mode winning, but the automated digest appended to the transcript rightly flags what the talk never says: what the 11% that got through actually were, and what the harm was. A classifier that blocks 89% of dangerous commands is only better than humans if the residual 11% is benign or recoverable — and "can I take it back?" is Hallie's own heuristic, applied to deny rules but not to auto mode's failures. Similarly, the risk-tiered review scheme (zero humans for docs and copy, one for product changes, two for risky ones) is asserted with no defect-rate data behind it.

The talk also has a structural blind spot: it is Anthropic describing how Anthropic works, at 30,000 changes a day, with the vendor's own beta features (projects, CI autofix, Merge when ready) as the destination. The migration path for a team without Anthropic's CI discipline is hand-waved. And the orchestrator pattern — one session spawning and verifying trusted worker sessions — is presented without any discussion of what happens when the orchestrator is wrong or injected; the digest flags this too, and it's a real gap given everything else in the talk is about containment.

What's non-obvious and worth keeping: the CLAUDE.md advice inverts the folk wisdom. The claim that "IMPORTANT!!!" instructions now cause *more* tool calls and worse results — because the model can already see what those instructions describe — means steering files should contain only what is invisible from the code. That directly complicates the configuration-accumulation advice in [[Tuning Claude Code Into a Better Engineering Partner]], which predates this shift. The hook gotcha (exit code 2, not 1, or the task is marked complete anyway) is the kind of operational detail that only surfaces from running the thing at scale.

## Related

- [[Simon Willison — Engineering Practices That Make Coding Agents Work]] — Willison's adoption ladder tops out at "don't read the code"; Hallie's talk radicalizes that rung into a full architecture for not being in the loop at all, strengthening his thesis while supplying the harness-and-environment machinery he gestures at.
- [[Tuning Claude Code Into a Better Engineering Partner]] — that page's ten configuration changes assume more configuration is better; Hallie complicates it directly, arguing emphatic CLAUDE.md instructions now push models off track and that a prompt-audit command exists precisely to strip them.
- [[A Deep Dive on Agent Sandboxes]] — Freeman's argument that sandboxing must be default, not optional, and enforced at the syscall level, is strengthened by Hallie's permission blind spot: she names the same gap (child processes escape permission checks) from the vendor side and extends it to cloud sessions as the endpoint.
- [[Guardrails and Feedback Loops]] — Hallie's verification points, guaranteed-by-hooks checks, and CI-as-backstop are a concrete, productized instance of this topic's thesis that quality must be made self-correcting rather than supervised.

---
*Sources: [[raw/e9774a987941f63d288bd17e856a717b]], [[summary/e9774a987941f63d288bd17e856a717b]]*
