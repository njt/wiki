# Always-On Agents (Rubric Labs)

Rubric Labs argues that the missing piece between today's chatbots and the Jarvis-shaped AI people actually imagine is not capability but **form factor** — and lays out a minimal architecture for the always-on version: a single-threaded agent with self-managed context and dispatchable sub-agents, a heartbeat that wakes it, messaging as a tool, broad computer access, and a filtered notification inbox. The genuinely hard problem they identify is *discernment*: knowing when to act at all.

---

The framing claim is the most quotable thing here:

> I have a hunch that the reason people disagree about whether we've reached AGI is **more about form factor than it is about model capabilities**.

That's a real thesis, not just a hook. If the bot you talk to daily behaves like an ephemeral contractor rather than a colleague who's always around, no benchmark will fix the felt gap. It reframes the AGI argument as a UX argument — which is either a dodge or the sharpest observation in the debate, depending on how generous you feel.

On context, they reject the default:

> compaction lobotomizes the agent's momentum... it makes more sense to give the agent tools to compact and awareness of how full its context window is, and then just **let it manage its own tokens**.

`foldContext` and `dropContext` as tools, token-usage awareness at every turn, agents compacting at natural lulls — this is agency applied to the agent's own housekeeping, and it sidesteps the usual third-party summariser that doesn't know what's still in play.

Proactivity is built from two small primitives: a **heartbeat** ("a simple message... that we push into the chat history. It wakes the agent if it's asleep") and **messaging as a tool** — the agent only reaches you when it decides to call `send_message`, and its turn-ending output stays in the thread as a note to itself. That inversion of the chat contract is the whole trick: staying quiet becomes a normal outcome rather than a failure to respond.

The running example (flight to Austin, cancelled mid-trip, agent reroutes while you're still in the air) is well chosen because it requires all three properties at once — awareness, proactivity, and judgement about when to interrupt you.

But the honest centre of the piece is discernment:

> Agents are either **too eager to do busy work or missing opportunities to be helpful**. Calibrating them to the right balance is more art than science.

The failure mode they describe — agents acting on Slack messages not addressed to them, then, finding no mandate or access, settling on "writing a design doc and sharing it. This is useless and embarrassing" — is exactly the behaviour anyone who has put an agent into a real channel will recognise. Their design answer is "start out shy, and over time build the confidence to act proactively," backed by a clear mandate, permission to do nothing, and memory of how the user reacted last time. That memory-of-reaction loop is the most novel suggestion here, and it quietly implies that discernment is a *learned, personalised* property, not a prompt you write once.

#concept #pattern #agent-ux

## Analysis

This is a good archetype of the current always-on-agent genre: the primitives are almost embarrassingly simple (one thread, a cron-ish wake-up, an inbox, one messaging tool), and the piece is refreshingly upfront that "architecturally, everything above is a fairly simple series of interventions." The hard 20% is judgement, and the authors correctly locate why: models are RL-trained to *crush tasks*, and there is no training signal anywhere for *declining* a task. The call for "new RL environments that reward discernment" is the piece's real research ask.

Two caveats. First, "give the agent full access to our email, browser and file system" is stated with a breeziness that ignores the blast radius of an always-on agent with broad permissions — the security literature in this wiki makes clear that's where these systems bleed. Second, the shy-to-confident calibration is described as "more art than science," which is candid but also means every deployment will be hand-tuned; the memory-of-reaction design is a good intuition with no eval story attached.

The piece sits deliberately close to [[How Always-On Agents Work (Stanford CS146S)]], which covers the same territory from an infrastructure angle; this one is stronger on the UX and judgement layer, weaker on the durability/ops side.

**Related:**
- [[How Always-On Agents Work (Stanford CS146S)]] — Lee Robinson's lecture covers the same always-on form factor with heavier infrastructure (durable workflows, Firecracker VMs, cache-aware compaction); this essay strengthens its "router that knows when to bug the user" intuition with a concrete design for it.
- [[Headlong — Persistent Agent Microharness]] — Laude's Audel is a working instance of nearly everything proposed here (one shared thought stream, unprompted pings, self-managed context); it complicates the piece's confidence by showing the failure modes (three accidental self-kills, "bad at keeping secrets") that broad access + proactivity produce in practice.
- [[Devin Memory and Dreaming]] — Cognition's memory-of-lessons store is the production cousin of Rubric's "memory of how we reacted when it did act," suggesting discernment-through-memory is converging across labs.
- [[Chief of Staff]] — a personal-agent deployment that lives or dies on exactly the eager-vs-missing-it calibration this piece names as the hard part.

---
*Sources: [[raw/always-on-agents]], [[summary/always-on-agents]]*
*Last updated: 2026-11-09*
