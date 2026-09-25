# How We Made Claude AI Faster

Anthropic's engineering team made claude.ai and the desktop app ~3x faster in a two-week sprint run entirely from a Slack channel where Claude (via Claude Tag, on an Opus-5.5-class internal model) found bottlenecks, built benchmarks, shipped fixes, and watched deploys — over 3,000 merged changes with zero customer-facing incidents. The essay's real contribution is a methodology: measurement as the first step of agent hill-climbing, ratchets as the safety mechanism, and a precise taxonomy of what humans still do (ambition, taste, direction).

---

## The Method

Four journeys cover 95% of user activity; Claude analysed Datadog data via MCP and turned them into thirteen directly comparable measurements. Twelve of thirteen targets were hit by day three, so the team kept re-raising them. The loop settled into: measure → prototype in the lab → prove against wall-clock → land as a ratchet → let telemetry spawn the next thread.

## Key Quotes

> "Once Claude can measure something, it can make it faster. So we kept finding more things to measure."

The compressed thesis. The inversion of classical performance work is the interesting part: "Measurement used to be step zero… With Claude, it's step one of the climb." When optimisation is cheap, instrumentation becomes the scarce resource — the highest-leverage human act is manufacturing new countable things.

> "We treated every new benchmark with some skepticism… If a benchmark was flaky, or if it didn't actually correlate with user latency, we threw it out rather than let Claude climb the wrong hill."

Goodhart discipline, operationalised. Every benchmark had two jobs — a lab metric Claude could move, and a CI guardrail that "could only ratchet down" — and candidates had to prove wall-clock correlation or be unshipped. This is the same insight as [[Ratchets in Software Development]] and [[Goodhart's Law Comes for Every Benchmark You Trust]], but here it is load-bearing at 150-thread scale rather than merely asserted.

> "Shelley, one of the engineers in the channel, observed, '[This model] is a numbers demon.'"

Threads didn't close when their request was fulfilled; Claude kept going, opening its own threads, fifty to a hundred PRs each. The demon is only tameable because the environment is: instruction-count ratchets, a nightly 120 Hz frame-budget job, an integration test that went red 20/20 on main and green 20/20 on the PR.

> "The loop was productive, but it wasn't autonomous."

The honest sentence the essay could have skipped and didn't. Humans supplied three things: **Ambition** (agents default to scope-hedging; "the targets are not the stopping point"), **Taste** (ruling on whether a table fills cell-by-cell, whether a fade is worth a fifth of the frame budget), and **Direction** (sequencing 150 hammers; gavelling down a 900-line PR because "2ms per send is not worth the complexity of maintaining this build plugin"). This is the sharpest field confirmation of [[Own the Outer Loop]] and [[Human-in-the-Loop is Tired]] at once: the human loop shrank to three verbs but did not disappear.

> "We introduced nearly two hundred flags, more than half of which were already cleaned up by the end."

Guardrails got their own agent thread — Claude classified every flag as kill switch or ramp and retired them. Even hygiene is delegated; even hygiene is supervised.

## The Weirdest Finds

A leftover `location.reload()` causing half a million hidden reloads a day invisible to load metrics; em dashes forcing V8 strings onto the UTF-16 path and slowing syntax-highlighting regexes (fixed in twenty lines); a `:root:has()` selector costing 24ms on every DOM change; 6,900 React hooks re-rendering in the composer's typing path; Chrome's speculative prerender on managed browsers causing a layout shift nobody's dashboards caught. Each find is an argument for exhaustive small-measurement over heroic big-profiling.

## Opinionated Take

This is the strongest published data point yet for the "feedback loops over intelligence" school — [[Feedback Loop is All You Need]] and [[Harness Engineering (OpenAI)]] made the argument, this one supplies the receipts: 3,000 changes, zero incidents. But note what made it safe: everything touched was a *hot path with an unambiguous number*. That's the friendliest possible domain for an agent — wall-clock latency is the rare quality property that is simultaneously user-legible, deterministic-ish in proxies, and CI-ratchetable. The essay's method does not transfer to taste-heavy or design-heavy work, and the authors tacitly admit it by reserving taste for humans.

Two things deserve more scrutiny than the victory lap gives them. First, "not a single rollback" over 3,000 changes strains belief unless most changes were trivially small — the 200-feature-flag strategy is really a blast-radius-splitting scheme, and the essay is vague about how much user-visible risk was actually absorbed by the 1% canaries. Second, the human role is described as three verbs but the load-bearing one is *taste at scale*: every user-perceptible change demanded a screenshot ruling, and with 200 changes/day the named owners were doing a very high-frequency judgment mill. "Ambition, taste, direction" is the vision; "someone reviewed 200 diffs a day" is the reality.

The organisational detail that will age best: everything ran in one channel in the open, other teams started bringing their work in for performance review, and new code was written in "subtly more performant ways" because the guardrails and skills were ambient. The channel became culture. That — not the 3x number — is the durable claim.

## Related

- [[108 PRs in Eight Days — Accidentally Discovering Loop Engineering]] — the small-scale indie version of the same loop; this source strengthens its loop-engineering thesis with industrial-grade numbers and adds the ratchet layer Ellich's setup lacked.
- [[Ratchets in Software Development]] — this is the ratchet pattern fully weaponised: instruction counts as CI gates plus a daily job that lowers the ceiling.
- [[Harness Engineering (OpenAI)]] — OpenAI's zero-hand-written-code report from the same genre; both converge on "humans steer, agents execute," but Anthropic's adds measurement-first as the missing step.
- [[Introducing Claude Tag]] — this sprint is the proof-of-concept for Claude Tag as team infrastructure: one channel, one persistent agent, 150 threads; the containment questions raised there stay open at this scale.

---
*Sources: [[raw/how-we-made-claude-ai-faster]], [[summary/how-we-made-claude-ai-faster]]*
*Last updated: 2026-09-25*
