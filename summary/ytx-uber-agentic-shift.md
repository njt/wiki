---
url: https://www.youtube.com/watch?v=i1tZN41VKcE
gist_url: https://gist.github.com/njt/2e5b37628d8aa8c6153d7fea2eb6ed63
title: "Uber: Leading engineering through an agentic shift - The Pragmatic Summit"
author: Anshu (Dev Platform Lead, Uber) and Ty (Principal Engineer, Uber)
date_fetched: 2026-07-04
date_published: 2026-05-18
channel: The Pragmatic Engineer
duration: 37m 39s
ytx_by: njt (Nick Tomlin)
topics:
  - agent-coding-workflow
---

## Summary

Anshu (leading Uber's developer platform org) and Ty (principal engineer driving the agentic shift) present Uber's internal transformation toward AI-augmented engineering. The talk covers infrastructure, developer workflow changes, adoption challenges, and the measurement gap between activity metrics and business impact.

### Key Points

1. **From pair programming to peer programming** — The earlier model involved synchronous tab completion and IDE chat, providing roughly a 10–15% bump in diff velocity. The emerging model treats agents as asynchronous peers that developers direct like a tech lead, handing off toil tasks and stepping in only for course correction.

2. **Toil as the sweet spot for agentic ROI** — About 70% of workloads pushed to Uber's agentic system are toil: upgrades, migrations, bug fixes, and dead code cleanup. These well-defined tasks yield higher accuracy, which drives a virtuous cycle of adoption and impact.

3. **Infrastructure as the differentiator** — Uber built a layered stack including an MCP gateway, agent registry, background agent runners (Minions), code review helpers, test generation, and large-scale change automation—all atop its existing Michelangelo ML platform. This internal infrastructure enables cost control, organizational context integration, and rapid model swapping.

4. **Reshaped developer workflow** — Engineers now run multiple background agents in parallel, spending more time on planning and code review. Uber created CodeInbox and UReview to manage the resulting PR volume and filter low-value AI review comments.

5. **A new platform for large-scale change** — To match claims from peers about the percentage of AI-generated code, Uber first built a scalable large-scale change system (AutoMigrate/Shepherd) handling problem identification, code transformation, validation, and campaign management across hundreds of PRs.

6. **Adoption slower than expected; peer wins beat mandates** — Engineers resist changing ingrained IDE-centric habits. Sharing concrete wins from fellow engineers drives adoption far more effectively than leadership directives.

7. **Costs up 6x since 2024** — The technology is impactful, but GPU and token costs now require CFO-level approval. Uber mitigates by using expensive models for planning and cheaper ones for execution, and by guiding developers toward appropriate models.

8. **Activity metrics don't satisfy the CFO** — Diffs landed, NPS, and self-reported productivity are at all-time highs, but the business needs revenue-impact proof. Uber is now instrumenting the feature pipeline to measure speed improvements.

### Quotes

- Dara Khosrowshahi, quoted by Anshu: "AI is enabling people to become superhumans in terms of their productivity"
- Anshu on the goal: "We want to enable all tasks that people do at Uber to be supported by generative AI"
- Anshu on augmentation vs. automation: "What we're not pushing for is AI automate all humans"
- Anshu on the new developer role: "We imagine developers acting as their own tech leads … directing AI agents"
- Anshu on cost: "The cost of AI is too damn high. Since 2024 our costs have gone up at least 6x"
- Anshu on business impact measurement: "I need to show him what's the impact on revenue"
- Anshu on executive onboarding speed: "I ran a demo session with some of my VPs and in 24 minutes I had four VPs land code"
- Anshu on build-vs-buy philosophy: "The tech we're building will likely be replaced with something better"
- Ty on the parallel-agent workflow: "The new flow looks like running several agents at once"
- Ty on code review noise: "We really only want to put the high confidence changes"
- Ty on the prerequisite for AI-generated code at scale: Uber lacked "the ability to kind of scale out large scale changes" that peers already had

### Tools & Practices

- **MCP Gateway** — Central gateway proxying external and internal MCP servers, with registry, sandbox, authorization, telemetry, and logging.
- **AIFX CLI** — Provisions and configures agent clients (Claude Code, Cursor, Codex, etc.), installs MCPs from registry, deploys standard configs.
- **Minions** — Background agent platform running on Uber's own infrastructure. Web, Slack, CLI, and API interfaces. Includes prompt improver.
- **CodeInbox** — Unified PR inbox with smart assignments (ownership, history, timezone, calendar), strict SLOs, batched Slack notifications, and risk analysis.
- **UReview** — Code review assistant with pre-processor, plugin system, review grader filtering low-confidence/nit comments, deduplication, and categorization.
- **AutoCover** — Custom unit test generation built on LangFX SDK. ~5,000 merged tests/month at ~3x quality of generic agents. Includes separate critic engine.
- **AutoMigrate / Shepherd** — Large-scale change platform: problem identification, code transformation (deterministic via OpenRewrite or agentic via Minions), validation, and campaign management across hundreds of PRs.
- **Prompt improver** — Built into Minions; analyzes prompt quality and suggests edits.
- **Model routing for cost** — Expensive models for planning, cheaper models for execution, infrastructure handles routing.
- **Peer win sharing** — Adoption tactic: engineer promoters share concrete successes with peers rather than top-down mandates.

### Omissions & Open Questions

1. No actual revenue impact data — only a plan to instrument the feature pipeline.
2. The 6x cost increase lacks absolute numbers; no discussion of whether productivity gains justify it.
3. No deep dive into how AI-generated code affects technical debt, architectural coherence, or maintainability.
4. No discussion of how agent-authored production code correctness is validated beyond human review, or security of broadly scoped agents.
5. No discussion of how junior engineers learn fundamentals when toil is offloaded.
6. Adoption numbers are vague — no specific percentages or segments for resistance.
7. No framework for evaluating build-vs-buy for agent infrastructure.
8. No solutions for integrating MCP endpoints into old, poorly understood systems ("archaic code").
9. No address of job displacement fears, team structure changes, or performance evaluation changes.
10. No counterarguments or failures discussed — narrative is overwhelmingly positive.

### Transcript

Full transcript available at gist URL. 37m 39s conversation covering Anshu's strategic framing and Ty's technical deep-dive with demos of Minions, AutoMigrate, and Shepherd.
