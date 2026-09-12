# Finding Problems Worth Working On

Lalit Maganti's answer to a mentee trying to make the jump to staff engineer: the role's differentiator isn't solving assigned problems, it's finding important ones your leaders don't yet know exist — and that skill comes not from blocked-out "strategic thinking" time but from ambient absorption, deliberate waiting, pattern-finding, and pressure-testing. Illustrated throughout with his own work on Perfetto, where a pile of unrelated UI feature requests eventually collapsed into one idea — extensibility — that none of the requests hinted at on its own.

---

## Key Quotes

> "I told him I rarely find good problems by staring at a blank page and trying to 'think strategically.' Instead, I act like a sponge."

The anti-calendar thesis. The mentee had already tried the standard advice — schedule time to think big thoughts — and found it unproductive. Maganti's replacement moves problem-finding out of a dedicated activity and into the byproduct of paying attention to the noise that already flows past you: meetings, chats, complaints. It's a craft answer to what most career advice solves with time management.

> "Users often ask for a particular solution instead of explaining their root issue. Rather than taking the request at face value, I keep digging until I understand what they are trying to accomplish and why existing products do not work for them."

The XY problem stated as an operating discipline. Note the follow-through: he sits with teams through their workflows, tries their bugs himself, and asks "If X existed, would it solve your problem?" — he runs cheap validation probes against each request before letting it into his problem inventory.

> "How eager a team was in that moment wasn't the same as how important the feature was relative to everything else my product needed to support. By hyperfocusing on their request, I lost sight of the bigger picture."

The scar-tissue quote. He built an eagerly-requested feature that the requesting team then barely used — their priorities had moved, or the request came from a one-off investigation. This is the strongest empirical claim in the essay: enthusiasm is a sampling bias, not a priority signal. Waiting is the correction: recurrence across independent teams is the evidence that one team's excitement can't supply.

> "When a connection like that finally clicks, it's one of the best feelings in the job: several awkward requests collapse into a single idea, and possibilities open up that none of them hinted at on their own."

The Perfetto payoff: requests for pinned tracks, default zoom positions, custom aggregations, and elaborate bookmarklet workarounds all turned out to be one need — personalize the UI without imposing your choices on everyone else — satisfied by macros and extension servers now used by dozens of Google teams and several other companies.

> "That feeling, though, is exactly when I have to be careful, because a common shape is only a hypothesis and elegance is not evidence."

The essay's sharpest line, and the thing that separates it from pattern-language romanticism. He kept the counterexample in: he was convinced a transparent caching system would solve both trace-sharing and repeated-query pain, and only discovered while writing the RFC that "the elegance was a lie" — the two problems wanted genuinely different solutions. Both shipped, separately.

> "I'm not only trying to convince other people; I'm also trying to convince myself. Sometimes the honest answer is to stop."

The commitment ladder: useful low-risk changes get sent immediately; uncertain ideas get throwaway prototypes that "expose the failure points"; big ideas he believes in get the full effort — RFCs, 1:1s, talks. And finding the right problem counts even when someone else builds it.

## Key Themes

- **#pattern Absorb problems, not requests** — ambient listening plus root-cause digging; the request is a corrupted encoding of the problem.
- **#pattern Waiting as an evidence filter** — unsolved problems are retained deliberately; recurrence, cross-team independence, and shared shape are the promotion criteria; "waiting can be a superpower."
- **#concept The common shape** — the hypothesis that several surface-different problems share one underlying need, with the explicit warning that a common shape is only a hypothesis.
- **#pattern Pressure-testing ladder** — escalation proportional to confidence: direct change → throwaway prototype → full RFC campaign, with stopping and parking as legitimate outcomes.
- **#concept The problem-finding flywheel** — solving real problems earns trust; trust brings you into conversations earlier; wider visibility makes the next pattern easier to see.
- **#person Lalit Maganti** — Google engineer on Perfetto; the essay is mentoring advice grounded in that infrastructure/devtools context, with an explicit caveat that it assumes bottom-up autonomy.
- **#tool Perfetto** — the performance debugging tool whose UI-extensibility arc (macros, extension servers) is the essay's worked example.

## Critical Analysis

**The waiting discipline is the operational core, and the essay's real contribution.** Most writing on staff-engineer impact says "find high-leverage problems" and stops. Maganti supplies the mechanism that makes it actionable: treat every request as unprioritized by default, and promote a problem only on accumulated evidence — independent recurrence across teams, or a shared shape that resolves several at once. That converts prioritization from a judgment call into an evidence-gathering protocol. The eagerly-requested, barely-used feature is the load-bearing anecdote because everyone has one.

**"Elegance is not evidence" is the line that makes the essay trustworthy.** The caching-system counterexample does real work: it would have been easy to present the Perfetto unification as proof of method and quietly drop the failure. Keeping it in, with the detail that the RFC/prototype process itself exposed the lie, quietly argues that pressure-testing isn't ceremony — it's the only place the hypothesis can die cheaply. The throws-into-relief point: his walk-around-London untangling produced the *hypothesis*, and the hypothesis only became real on contact with an RFC and a prototype. Insight is generative; verification is decisive.

**The method has a cold-start problem the essay doesn't address.** The flywheel runs on trust — people bring problems to you *because your past calls were right*. Maganti acknowledges the loop ("Early on, I had to turn many of these ideas into something real myself to prove that my judgment was sound") but doesn't engage with what this means for his actual audience: a senior engineer trying to break into staff has neither the accumulated trust nor, usually, the latitude to let problems sit for two years. His own caveat about bottom-up autonomy compounds this — the essay quietly describes how the method works for someone whose judgment an org already prices in.

**Survivorship risk in the sponge.** The narrative only shows the connections that eventually appeared. Ambient absorption with no external record depends on the pattern-matcher being good, and Maganti's is — but the Perfetto case took "a couple of years" of accumulation against a head he describes as "the usual tangle." He waves at the fix ("Other engineers I know write this sort of thing down more systematically. The mechanism is a personal choice") without noticing it undermines his own story: a written problem ledger would likely have surfaced the common shape far earlier, and would make the method transferable to people whose heads aren't as good. The dismissal of systematic capture reads as virtue made of preference — an introvert's apology for a practice that only scales if you're the sort of introvert who's right.

**The staff-engineer framing is the essay's quiet polemic.** Against the "staff engineer = meetings and coordination" stereotype, he insists conversations are *inputs* to building, not the job itself — he stayed a builder who uses the org as his sensor array. That's a genuinely different theory of the role than the influence-without-implementation one, and the Perfetto outcome (he designed and implemented macros himself) shows it's not just rhetoric.

## See Also

- [[The Mythical Agent-Month]] — McKinney's claim that "figuring out what to build was the hard part long before we had LLMs" is asserted there as the human residue agents can't touch; Maganti supplies the pre-agent operating manual for exactly that skill, which strengthens the case that it survives the agent era intact.
- [[The AI Productivity Paradox]] — Cagan diagnoses teams industrializing delivery while discovery stays human-paced; this essay is a working discovery discipline (absorb → accumulate → find the common shape → pressure-test), and its "eager requests aren't evidence" rule applies Cagan's output-vs-outcomes distinction one step earlier — to the *input* triage, not just the build decision.
- [[Product-Minded Engineers in an AI-Native World]] — the "customer signal" skill described there (denoising user input from Gong calls, GitHub issues, Slack threads) is the tool-assisted version of Maganti's absorb-problems-not-requests; his essay nuances it by locating the denoising in years of ambient context and firsthand bug-handling rather than searchable transcripts.
- [[The Importance of Sketching with Code]] — Gorilla Sun's throwaway sketch "solves problems you don't have yet," while Maganti's throwaway prototype exposes the failure points of a problem he very much has; the same disposable artifact, pointed in opposite temporal directions, and both essays defend it against the same ROI-focused skepticism.

---
*Sources: [[raw/find-problems-staff-engineer]], [[summary/find-problems-staff-engineer]]*
*Last updated: 2026-09-13*
