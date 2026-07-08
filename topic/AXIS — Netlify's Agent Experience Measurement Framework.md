# AXIS — Netlify's Agent Experience Measurement Framework

Netlify's open-source scoring framework (think Lighthouse for agent-facing services) that quantifies how well a platform serves AI agents across four dimensions: Goal achievement, Service, Environment, and Agent. Skills lifted scores by 26 points on average and reduced both time and token cost on every run. The long play is a context pipeline that makes agent context a first-class delivery artifact, gated by AXIS scores on every PR.

---

## Key Quotes

> "Agent Experience (AX) is the holistic experience AI agents have as users of a product or platform."

Netlify is naming a category, not just a metric. This is the same move Google made with "web performance" before Lighthouse existed — first you name the thing, then you measure it, then you optimize it. The definition is intentionally broad because the surface area is broad: discovery, reliable API calls, error recovery, context quality.

> "Think Lighthouse, but for Agent Experience."

The analogy is tighter than it first appears. Lighthouse didn't just measure — it *educated*. Developers learned what Core Web Vitals were by running Lighthouse. AXIS has the same pedagogical ambition: the four-dimension breakdown teaches teams what "good" means for agent-facing surfaces.

> "across all three, skills lifted scores by an average of 26 points and reduced both time and cost on every run"

This is the money quote, and it's not subtle. Providing agents with structured context (skills, CLAUDE.md, MCP tool descriptions) isn't a nice-to-have — it's a 26-point score swing. The fact that time *and* cost dropped alongside score improvement demolishes the "context is expensive" objection. Context pays for itself.

> "A finding is a data point. A practice is what makes progress."

The difference between running AXIS once and embedding it in CI. A single score tells you where you are; a pipeline that blocks regressions tells you where you're going. This is the same insight behind [[Guardrails and Feedback Loops]]: measurement without enforcement is theater.

> "The belief has aged well. But belief isn't a standard."

A quiet shot at the "we know our platform works well for agents because we use it ourselves" school of thought. Dogfooding is useful but it's anecdotal. AXIS makes it systematic.

## Key Themes

#agent-experience #evaluation #context-engineering #scoring-framework #open-source #CI/CD #MCP

## Critical Analysis

**The category is real and underserved.** "Agent Experience" isn't a marketing coinage — it's an actual gap in the toolchain. Every platform team I talk to knows their MCP server or CLI could be better for agents, but they have no systematic way to measure *how* much better or whether changes help. AXIS fills that gap. The Lighthouse analogy is earned.

**The skills finding is both obvious and important.** Of course giving an agent structured context improves outcomes. But the *magnitude* — 26 points, with time and cost reductions on every single run — is striking enough to change behavior. This isn't a marginal improvement; it's the difference between an agent that works and one that doesn't. The delegate-local-wip scenario with Codex going from 63 (barely functional) to 98 (near-perfect) while halving time and tokens is the chart you put in the slide deck to get budget for context engineering.

**The context pipeline is the real ambition.** The AXIS scoring framework is open-source and useful today, but Netlify's internal context pipeline — auto-deriving agent context from docs repos, testing it with AXIS, and gating deploys on score regressions — is where this gets interesting. "New docs can't ship without a deliberate decision about their agent context" is a genuinely new constraint on the documentation workflow. It elevates agent context from an afterthought to a first-class deliverable, alongside the docs themselves. If this works, it creates a moat: platforms that auto-generate and verify agent context will be dramatically easier for agents to use than platforms where context is hand-maintained and perpetually stale.

**The 97% MCP quality stat is a callout worth unpacking.** The Queen's University finding that 97% of MCP tool descriptions had quality issues isn't an indictment of MCP — it's an indictment of the assumption that tool descriptions are self-documenting. Most teams write tool descriptions for humans (if they write them at all), and agents need something different: structured, scoped, tested against real scenarios. AXIS is the verification layer for that shift.

**What's missing:** The article doesn't address how AXIS handles *drift*. A platform changes, its agent context ages, and the AXIS score drops — but how quickly? Is there a decay curve? Also, the 22-agent support is impressive but the article only reports results for Claude Code and Codex. What does the score distribution look like across all 22? If some agents consistently score 30 points lower on the same platform, that tells you something about those agents' harness design — and that's data Netlify has but isn't sharing.

**The Auth0 involvement is strategically interesting.** Auth0 as a "founding contributor" to AXIS suggests this isn't just a Netlify vanity project. Identity infrastructure is a natural early adopter of Agent Experience measurement because authentication is the first friction point every agent hits. If Auth0 treats AX scores as a product requirement, expect Okta, WorkOS, and Clerk to follow.

This pairs naturally with [[How AI Coding Agents Actually Use Your Technology]] — Mastykarz diagnosed the seven-step AX cascade where failures are invisible; AXIS is the measurement framework that makes them visible. Together they form a diagnosis-and-treatment pair for the agent experience problem. Also connects to [[10 Principles for Agent-Native CLIs]] (Chow's design principles are the *what*; AXIS is the *did-it-work*), [[Experience Design for Agents]] (Kemple's thesis that UX determines adoption; AXIS measures that UX quantitatively), and [[The Agentic Product Standard v2.0]] (AXIS is the evaluation layer the Standard implies but doesn't specify).

---
*Sources: [[raw/netlify-axis-agent-experience]]*
*Last updated: 2026-07-08*
