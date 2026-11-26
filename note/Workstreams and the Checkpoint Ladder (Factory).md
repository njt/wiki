# Workstreams and the Checkpoint Ladder (Factory)

Luke Alvoeiro, head of Factory's core agent team, describes how Factory rebuilt its multi-agent offering after "Missions" stalled: a software-factory architecture organised around **workstreams** — standing goals with identity, environment, and shared memory — driven by an orchestrator that works like an OS scheduler, with autonomy ramped through human checkpoints (L1→L4) gated by a measurable "regret-free merge rate."

---

## What the talk argues

- Hand over work **gradually**, not on day one: "the teams that are getting the most out of coding agents are not actually the ones that are handing over like the most work on day one."
- Standardize the **process, not the output**. Every signal (Sentry alert, Slack complaint) goes through the same stages: triage → spec → PR → validation → merge, and learns from production.
- Missions failed three ways: it demanded full up-front planning with no interactive surface; adversarial review at every milestone exploded scope from ~60 to 200+ features; and context stayed trapped with one person.
- Running hundreds of agents recreates **operating-system problems**: scheduling, safe concurrency, shared state. The orchestrator delegates by "areas of ownership" rather than a rigid DAG — which they tried first and found too rigid and expensive.
- Programmatic tool calling (code mode) cut a 150-ticket Linear migration from ~30 inference requests / 1M tokens to one generated program / ~60k tokens.
- Autonomy levels L1 (human review at spec, PR, merge) to L4 (automerge); the owner raises the level only when observed quality earns it.

## Key quotes

> "You don't need to decide on day one whether you trust a coding agent to own, you know, your like CI health workstream off the bat. You just need to trust it to to like unlock the next checkpoint."

The checkpoint ladder reframes trust as incremental and evidence-based — a much more sellable adoption story than "give the agent your whole backlog."

> "We measure it for droid ridden and humanridden PRs side by side. And it's the same way that we let like self-driving cars onto the road... once AI written changes break things less often than human ones and you can actually measure that then you can let them merge."

The **regret-free merge rate** is the most interesting artifact here: a single comparative metric that turns the autonomy dial from a policy debate into an empirical gate.

> "The judge gets the exact if and a check out of the branch so it can like build and test it. Uh but ultimately it reads its policy from our mainline branch so that the change can't rewrite the rules that it's being judged by."

A quiet but crucial governance detail: the reviewing policy lives outside the reviewed change, closing the classic self-judging loophole.

## Key themes

#concept (workstream, regret-free merge rate, autonomy levels) #pattern (OS-as-mental-model, shared filesystem memory) #tool (Factory/Droid, code mode) #person (Luke Alvoeiro)

## Take

This is vendor material — "higher quality at a lower cost than every other coding agent on the market" is a claim, not a finding — but the architecture is concrete and the production numbers are the strongest part: 800 PRs in a weekend, 99% droid-written, 52% of PRs now originating from workstreams, with some fully autonomous streams merging 40–53 PRs/week. The self-driving-car analogy for the merge gate is rhetorically neat but slightly dodges a difference: car crash rates have a base rate everyone understands; "regret" for a PR is defined by Factory's own instrumentation. The honest test is whether customers can replicate the measurement independently. The most transferable idea isn't the product at all — it's the last-minute ask: write down your signals, pick the one you waste the most time on, give it to an agent at L1.

## How it relates

- Strengthens [[Droid 1.0 — Building a Frontier Harness (Factory)]] with Factory's own account of why the harness was rebuilt — code mode and model-jaggedness tuning are the concrete sequel to that source's harness story.
- Complicates [[Crawl, Walk, Run to the Software Factory]] by supplying a specific, metric-gated version of the gradual-autonomy ladder it advocates.
- Nuances [[Self-Improving Software Factories]] with the loop made concrete: droids mining review comments into guidelines daily and tracing reverts back to PRs.
- Shares the vendor-measuring-itself framing with [[Using LLM-as-a-Judge Scoring to Measure Your Software Factory]] — both ask the factory's operator to grade its own output.

---
*Sources: [[raw/luke-alvoeiro]], [[summary/luke-alvoeiro]]*
*Last updated: 2026-11-26*
