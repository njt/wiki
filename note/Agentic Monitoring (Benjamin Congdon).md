# Agentic Monitoring (Benjamin Congdon)

Benjamin Congdon argues that the sleeper application of coding agents is production monitoring: agents that watch telemetry and logs like "an experienced engineer who has nothing better to do than to refresh charts", detecting subtle issues traditional alerts find one-dimensional. His change-point-monitoring case study shows agents dissolving a long-standing precision/latency tradeoff — provided the harness separates agent-written specs from deterministic execution.

---

## What it says

- Anthropic's blog post described "superfan" Claudes, one per component, with access to telemetry and logs. Congdon calls this "so obviously the path forward".
- Systems are increasingly "grown" than "designed"; human eyes-on-code knowledge per operator is shrinking, and observability is already expensive (his team's optimization saved several million dollars in annualized observability cost — a fraction of one team's spend).
- Change-point monitoring failed as platform engineering for structural reasons: targeting motivated teams yields toil reduction but no safety gain; targeting everything drowns in noisy metrics.
- Agents fix this. But a pure agent loop is wrong; agent-writes-spec + deterministic-executor is the right shape. Secret sauce: noise-reduction heuristics, good spec prompts, a labeled eval set.
- Agents' unfair advantages: exhaustive 24/7 coverage, parallel investigation, speed to understanding, elastic fleets of incident responders (mirroring the defender side of agent-vs-agent cybersecurity).
- Cost collapses: Haiku-class models make fleet-wide monitoring ~$100/day for a large company.
- The virtuous cycle: coding agents ship more, monitoring agents support more, humans "build the factory" and "tend the garden".

---

## Key quotes

> "our superfans can detect subtle and novel issues that make these alerts seem one-dimensional by comparison."

The framing Congdon builds the whole essay on — monitoring as attention, not alerting. A human operator's attention cutline is the real constraint; agents delete it.

> "a harness that allows the agent to write a spec of how to monitor, along with a deterministic system for running those specs, produces significantly cleaner signals with dramatically lower token usage."

The load-bearing architectural claim, and the reason this is interesting beyond ops: the agent plans, deterministic code executes — the same division of labor showing up everywhere in 2026.

> "There hasn't been a 'Claude Code moment' for monitoring yet."

His "we're so early" claim. Note it's slightly undercut by his own footnote that Anthropic's Claude Tag "evidently already does a good job at agentic monitoring" — the moment may have already happened without the practitioners noticing.

> "It will soon be reckless not to have agents monitoring your systems."

Strong version of the argument. The caution to read against: this is exactly the rhetoric that preceded other "obviously the path" trends that plateaued.

---

## Themes

#concept (agentic monitoring, superfans, attention cutline) #pattern (spec-writer + deterministic executor) #tool (Claude Tag, observability stacks) #person (Benjamin Congdon)

## Analysis

The genuinely novel move here is precision/recall economics. Observability has always been rationed by human attention: you could have precision (targeted, hand-tuned) or coverage (broad, noisy), never both. Congdon's argument that agents break the tradeoff by being *willing to do the tedious work per-change* is the sharpest formulation of "agents change what's worth automating" I've seen applied to ops. The spec-plus-deterministic-runner harness reinforces it: the agent's contribution is judgment at write time, not vigilance at run time.

Three honest caveats. First, the essay leans on prototypes and single anecdotes (the 50% heap savings) — impressive, but the eval-set sentence does a lot of quiet work: without labeled ground truth, "cleaner signals" is a feeling. Second, the security comparison (offensive agent fleets necessitating defensive ones) smuggles in an arms-race assumption that flatters investment. Third, the virtuous-cycle conclusion assumes monitoring quality stays high as agents watch agent-grown systems — the observability of systems nobody fully understands is precisely the case where false confidence is most dangerous. He acknowledges human advantage in tacit-knowledge intuition, which is where the interesting tension lives.

Still, as a practitioner position paper this is exactly the "writing from practitioners" he says is missing.

## Related pages

- Strengthens [[Cloud Software Factories]]' pipeline by giving its `monitor` stage a concrete, costed architecture — spec-writing agents plus deterministic runners — rather than a placeholder.
- Complicates [[Build the Factory, Let It Run, Go Kayaking (Warp, Fall 2026)]]'s SRE-shaped mandate: Congdon argues the monitoring half of that mandate is closer to solved than Warp's framing suggests, and that it's the missing write-up, not the missing capability.
- Nuances [[ZizkaDB — Causal Lineage for Agent Observability]]: observability is usually discussed as *observing agents*; here it's agents *doing the observing* of conventional systems — the mirror-image problem, sharing the same harness question of what the agent sees.
- Extends [[The AI-Native SDLC Playbook]]'s incident-record/autonomous-maintenance stage into a standalone discipline, with the same artifact-chaining logic: monitoring plans as specs the scaffold executes.

---
*Sources: [[raw/agentic-monitoring]], [[summary/agentic-monitoring]]*
*Last updated: 2026-10-20*
