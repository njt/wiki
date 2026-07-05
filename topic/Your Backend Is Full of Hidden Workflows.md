# Your Backend Is Full of Hidden Workflows

How backend codebases quietly accrete coordination logic — retries, queues, callbacks, webhooks — until teams are managing workflows they can't see. Why every decision made sense in isolation, what hidden workflows actually cost, and what changes when you make the implicit explicit.

---

## Key Quotes

> "Most software systems don't become complex overnight. They grow that way little by little."

Gulam Mohiuddeen opens with the thesis that this isn't about bad engineering — it's about the natural accretion of small, individually-reasonable decisions. Each retry, queue, and notification was the right call at the time. The problem is emergent.

> "The workflow exists whether you acknowledge it or not. The difference is visibility."

This is the sharpest line in the piece. Teams treat workflows as something formal — something that requires a dedicated orchestration platform — when in reality a workflow begins the moment one action depends on another. Refusing to name it doesn't make it go away; it just makes it invisible.

> "Making workflows visible doesn't create complexity. It reveals the complexity that was already there."

A direct rebuttal to the "we don't need orchestration, we're not that complex" reflex. The complexity is already in production. The question is whether you can see it.

> "What appears to be a single feature is actually a sequence of connected steps. There are decisions being made, external services being called, dependencies that can fail, and a clear outcome the business cares about. In other words, it's a workflow."

The reframing move: signup flows, BFF layers, support ticket routing, order processing — these aren't "backend features." They're orchestration work that happens to be implemented as glue code spread across handlers, jobs, and scripts.

> "Workflows aren't the problem. Hidden workflows are."

The closer. The article isn't arguing for heavyweight orchestration infrastructure — it's arguing against invisibility.

## Key Themes

- #pattern — **Workflow accretion**: coordination logic accumulates incrementally, one reasonable decision at a time, until the system's behavior is distributed across too many places to hold in one person's head
- #concept — **Implicit vs. explicit orchestration**: the distinction isn't between "has workflows" and "doesn't have workflows" — it's between visible and invisible ones
- #pattern — **The three costs of hidden workflows**: expensive changes (no single owner of the end-to-end process), painful debugging (piecing together logs across services), and eroded trust (teams become cautious because they can't see the full picture)
- #concept — **Features as disguised workflows**: signup, order processing, support ticketing, BFF layers — these present as features but are actually multi-step coordination processes
- #tool — Unmeshed, the author's platform, gives coordination logic a "proper home" as an explicit workflow definition

## Critical Analysis

**What's genuinely good:** The core observation — that workflow complexity accretes from individually-reasonable decisions — is real and important. Every experienced backend engineer has lived this. You add a retry wrapper, then a queue for the slow path, then a webhook handler for the third-party integration, and suddenly "create order" touches six services and nobody can draw the full flow from memory. Mohiuddeen is describing something universal, and he does it clearly.

**The diagnostic is stronger than the prescription.** The first 80% of the article is a sharp articulation of the problem. The last 20% is "use Unmeshed." The lead enrichment example is the right level of concreteness, but it's also the only example, and it's built on the author's own platform. I'd have liked to see what making-the-workflow-explicit looks like without adopting a specific orchestration tool — e.g., what does a team do if they document the workflow as a state machine in their wiki, or extract it into a dedicated service using existing infrastructure?

**The "you probably already have these" section is the strongest** — signup, BFF, support, order processing. Four concrete patterns that nearly every SaaS team will recognize. This is the part that makes the article more than vendor content: it names the thing you're already doing but haven't named.

**The vendor framing is honest enough.** Mohiuddeen works at Unmeshed and the article ends with a CTA. But the problem description stands on its own. Compare favorably to most vendor content that starts from "you have a problem you didn't know about" — at least here, the problem is real and well-articulated before the solution appears.

**What's missing:** No discussion of when explicit workflows are the *wrong* choice. A team with three services and two background jobs doesn't need an orchestration platform. The article acknowledges that workflows grow gradually but doesn't give a heuristic for when the tipping point arrives — when does the cost of hidden complexity exceed the cost of adopting an orchestration layer? That's the question teams actually need answered.

**Relationship to the wiki's themes:** This is the same pattern as [[Event-Driven vs Polling Architectures]] applied to business logic rather than infrastructure triggers: the diversity IS the architecture, and pretending it's simpler than it is causes production failures. It's also adjacent to [[Process Flow]]'s thesis that workflows should emerge from service chains rather than be declared in central DAGs — though Unmeshed appears to favor central definition where Process Flow favors choreography. The [[Software Engineering Craft]] page would be the natural home for the "accretion of complexity" observation if it were generalized beyond workflows.

## See Also

- [[Process Flow]] — Choreography-as-a-service: each stage designates its successor, no central DAG. The anti-Unmeshed in architectural philosophy (emergent vs. declared workflows), but shares the premise that hidden coordination is the enemy
- [[Event-Driven vs Polling Architectures]] — Tricot's parallel thesis applied to infrastructure triggers: webhooks alone are a production trap, and the diversity of delivery contracts IS the architecture. Same pattern as this article: the thing you're not acknowledging is the thing that breaks
- [[n8n]] — Visual workflow automation with 400+ integrations. If Unmeshed is "make the workflow explicit in code/configuration," n8n is "make the workflow explicit on a canvas." Same problem, different abstraction level
- [[Swamp Club]] — Agent-first workflow framework with DAG execution and Zod-typed models. The agentic parallel: agents also need explicit coordination rather than implicit glue
- [[Software Engineering Craft]] — The natural home for the accretion-of-complexity observation; this article is a worked example of one specific accretion pattern
- [[Agent Orchestration]] — Multi-agent coordination patterns hub; the same "hidden workflows" problem appears when agents chain together

---
*Sources: [[summary/your-backend-is-full-of-hidden-workflows]]*
*Last updated: 2026-06-09*
