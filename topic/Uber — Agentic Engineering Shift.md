# Uber — Agentic Engineering Shift

Uber's engineering leadership on their internal transformation to AI-augmented development: the infrastructure they built, the workflow changes they observed, the adoption friction they underestimated, and the measurement gap they still haven't closed. A rare look inside a large company wrestling with the agentic transition — honest about what's hard, evasive about what's unproven.

---

## The Core Shift: From Pair Programming to Peer Programming

Anshu frames the evolution in two phases. Phase one was **pair programming**: synchronous tab completion and IDE chat, yielding a ~10–15% bump in diff velocity. Phase two is **peer programming**: agents as asynchronous collaborators that developers direct like a tech lead.

> "We imagine developers acting as their own tech leads … directing AI agents."

This isn't a metaphor — it's a workflow prescription. The developer plans, assigns, reviews, and course-corrects. Agents execute. This maps cleanly onto the [[Agent Coding Workflow]] maturity spectrum: it sits at the "compound engineering" level, where the human owns architecture and verification while agents own implementation.

The practical manifestation: engineers run **multiple background agents in parallel**, spending more time in planning and code review because so much more code is being generated. This is the bottleneck migration pattern seen in [[Automating Myself Out of Development]] and [[Running an AI-Native Engineering Org]] — the constraint moves from "I can't write code fast enough" to "I can't review code fast enough."

## Toil Is the Sweet Spot

> About 70% of workloads pushed to the system are toil: upgrades, migrations, bug fixes, dead code cleanup.

This isn't surprising, but it's validating. Toil tasks have clear success criteria, bounded scope, and deterministic verification — everything agents need to succeed. The virtuous cycle Anshu describes is real: well-defined tasks → higher accuracy → more trust → more adoption → more tasks.

The implication: **don't start with creative work**. Start with toil. The boring stuff builds the confidence loop. This is the same pattern in [[StrongDM Factory Techniques]] — the DTU (Do The Unpleasant) pattern, where automation's first win is always the thing nobody wants to do.

## Infrastructure Is the Real Differentiator

Uber built a **layered proprietary stack** atop their existing Michelangelo ML platform:

| Layer | Component | What It Does |
|-------|-----------|--------------|
| Access | **MCP Gateway** | Proxies internal/external MCPs with auth, telemetry, sandbox |
| Provisioning | **AIFX CLI** | One command to configure any agent client + install MCPs |
| Execution | **Minions** | Background agents on Uber infra, not vendor infra |
| Review | **CodeInbox + UReview** | Manages PR flood, filters low-value AI comments |
| Testing | **AutoCover** | Custom test gen with critic engine, 3x generic agent quality |
| Scale | **AutoMigrate / Shepherd** | Large-scale change across hundreds of PRs |

This is the most interesting slide of the talk. It's not a demo — it's a **platform architecture diagram** that reveals Uber's bet: the moat is in the integration layer, not any single model or vendor. They're building the [[Smart Models Dumb Pipes]] pattern at enterprise scale.

Sierra's [[The MCP Gateway Iceberg]] provides the engineering companion to Uber's strategy slide: seven concrete lessons from building an equivalent gateway, including the "grab the lock" coordination pattern, multi-pass cross-customer data guards, and the dual identity model (interactive-as-user, scheduled-as-service-account) that Uber's architecture implies but doesn't detail.

> "The tech we're building will likely be replaced with something better."

Anshu is refreshingly unsentimental about this. If Cursor's test coverage makes AutoCover obsolete? Fine. The platform's job is to deliver impact, not preserve itself. This is the right posture — and exactly the counterargument to [[The Case Against Building Your Own Agent Platform]]. Uber's scale justifies the build; the platform's modularity means individual components can be swapped when vendors catch up.

## The Measurement Problem Nobody Has Solved

This is where the talk gets genuinely interesting — and evasive.

> "NPS is at an all-time high. Developer satisfaction all-time high. Self-reported productivity all-time high. Number of diffs landed all-time high."

And then:

> "I need to show him what's the impact on revenue."

The CFO doesn't care about diffs. Neither should anyone else. [[Writing Code vs. Shipping Code]] demonstrated this empirically: 180% AI-driven commit gains attenuate to 30% at release level. Activity metrics are vanity. Uber's plan to instrument the **feature pipeline** (design → experiment → production) and measure speed improvements is the right move, but Anshu admits they haven't done it yet.

This is the most important open question in the talk. If the biggest, best-resourced agentic deployment in industry can't yet demonstrate revenue impact, what does that say about everyone else's claims?

## Cost Explosion

> "The cost of AI is too damn high. Since 2024 our costs have gone up at least 6x."

No absolute numbers provided — a notable omission. But the directional signal matters: what was once fundable from Anshu's dev platform budget now requires **CFO approval**. This changes everything about how AI tooling gets procured and justified.

The mitigation strategy is [[The Advisor Strategy]] / [[Thrifty (Tiered Delegation for Claude Code)]] pattern applied at enterprise scale: expensive models for planning, cheap models for execution, infrastructure routing the decision. This is cost-aware model routing as infrastructure, not personal preference.

The broader implication: **AI engineering cost is becoming a boardroom topic.** The [[Why Agents Matter More Than Other AI]] CFO's spreadsheet math — agents are cheaper than humans at scale — only works if token costs stay manageable. The 6x trajectory suggests they don't.

## Adoption: Reality Check

> Adoption has been "relatively slow" despite the technology being "magic" in demos.

This is the most honest thing in the talk. Despite executive sponsorship (Dara made AI one of Uber's six strategic shifts), despite best-in-class infrastructure, despite demos that look like magic — engineers resist. The reasons are familiar: **ingrained IDE habits die hard**, and context-switching to a new workflow has real cognitive cost.

The tactic that works: **peer win-sharing**, not top-down mandates. Engineers trust other engineers, not directors. This aligns with [[Experience Design for Agents]] — adoption is a UX problem, not a capability problem. The tool can be perfect and still lose to muscle memory.

Also notable: Anshu got **four VPs to land code in 24 minutes** during a demo. The implication is that non-engineers may actually adopt faster than engineers because they have no existing workflow to unlearn.

## What's Missing

The talk is unusually frank for a corporate presentation, but it still has a **survivorship bias problem**. Ten major omissions:

1. **No revenue impact data** — only a plan to measure it
2. **No absolute costs** — 6x of what?
3. **No discussion of code quality** — technical debt, architectural coherence, maintainability
4. **No correctness validation beyond human review** — how do you know agent code works?
5. **No junior engineer development path** — what happens when toil is offloaded?
6. **No specific adoption numbers** — just "relatively slow"
7. **No build-vs-buy framework** — when should you build vs. buy?
8. **No solutions for "archaic code"** — admitted as a problem, no remedy
9. **No discussion of job displacement, team structure, or performance eval changes**
10. **No failures or counterarguments** — the narrative is overwhelmingly positive

The last point is the most telling. Every large-scale technology deployment has failures, reversals, and things they'd do differently. None are mentioned. The talk is a **recruitment pitch and industry positioning** as much as it is a technical report.

## Bottom Line

Uber's agentic transformation is real, well-architected, and more honest than most. But it's also incomplete. They've built impressive infrastructure and seen genuine workflow changes. They haven't proven it moves revenue. The 6x cost increase with no revenue-attribution data is the talk's central tension — and the industry's.

The infrastructure lessons (MCP gateway, background agents on your own infra, cost-aware model routing, large-scale change platform) are valuable and transferable. The measurement gap is the warning: if Uber can't close it, neither can you.

---

## Key Themes

#agentic-engineering #developer-productivity #peer-programming #toil #code-review #mcp #large-scale-change #build-vs-buy #adoption #cost #measurement #platform-engineering

## Related

- [[Agent Coding Workflow]] — The maturity spectrum this talk operates at
- [[Writing Code vs. Shipping Code]] — Why diffs landed ≠ business impact
- [[The End of Code Review]] — The review bottleneck Uber is trying to manage
- [[Running an AI-Native Engineering Org]] — Fiona Fung's parallel report from Anthropic
- [[Automating Myself Out of Development]] — Async agent workflow, same bottleneck migration
- [[All Your Agents Are Going Async]] — HTTP is wrong for agents that outlive connections
- [[The Advisor Strategy]] — Expensive model plans, cheap model executes (Uber's cost strategy)
- [[Thrifty (Tiered Delegation for Claude Code)]] — The same cost pattern in a plugin
- [[Smart Models Dumb Pipes]] — Uber's platform architecture in principle form
- [[The Case Against Building Your Own Agent Platform]] — The counterargument to Uber's approach
- [[Who Does What — Team Topologies for the Agentic Platform]] — Platform absorbs cognitive load
- [[Software Engineering at the Tipping Point]] — AI as 10× amplifier, not directed solution
- [[Experience Design for Agents]] — Adoption is UX, not model capability
- [[Loop Engineering]] — The meta-skill Uber's platform enables
- [[Why Agents Matter More Than Other AI]] — The CFO's case, and the cost problem that complicates it
- [[StrongDM Factory Techniques]] — DTU and the toil-first pattern

---
*Sources: [[summary/ytx-uber-agentic-shift]]*
*Source URL: https://www.youtube.com/watch?v=i1tZN41VKcE*
*Gist: https://gist.github.com/njt/2e5b37628d8aa8c6153d7fea2eb6ed63*
*Last updated: 2026-07-04*
