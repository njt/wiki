---
url: https://thegeneralpartnership.substack.com/p/a-practical-guide-to-brownfield-ai
date_fetched: 2026-07-05
backfilled: true
---

# A Practical Guide to Brownfield AI Development

### Context engineering for legacy codebases

Conventional wisdom has it that AI excels at greenfield work. It’s great at writing new code without the drag of legacy baggage. Brownfield development—when AI touches established systems—is where things get dangerous. That belief is understandable. And not entirely wrong.

A couple of months ago, I spoke with an engineering lead who’d nearly given up on AI tooling. Her team had experimented with Claude to help refactor an 8-year-old Django monolith. Nothing dramatic—but the agent produced clean, confident patches that quietly broke integrations with a couple of external services. After a few rollbacks, chasing subtle regressions, and unwinding overly optimistic changes, the team shelved the experiment and went back to a manual approach.

“I’m not anti-AI,” she told me. “But real systems carry a lot of invisible context.”

She’s not wrong, and you’ll find many similar stories on r/vibecoding.

Some people see this as evidence of limitations in the models. And while I do expect the next generation of models to perform significantly better, I suspect the real problem is that many legacy systems lack the structure agents need to operate safely.

Greenfield codebases start with clear boundaries, minimal hidden dependencies and no accumulated quirks. Legacy systems often have the opposite. Without guardrails, agents can only see the code in front of them—not the invisible contracts holding the system together.

This creates a bind. Agent autonomy requires structure, but legacy systems don’t have it. And adding structure to a messy codebase is exactly the kind of slow, careful work you want agents to help with.

The way out is to build structure incrementally. Start with heavy oversight and let agent autonomy increase as the system becomes more legible. After several production migrations—including a recent medical visualization tool that moved from vanilla JavaScript to React—a pattern is emerging for how to do this without destabilizing what already works.

Here are three principles that I’ve found useful:

### Tests as System Boundaries

Before touching production code, linting, type-checking, and E2E tests serve as the system boundary that makes everything else possible. This is about creating feedback loops that allow the agent to verify its work.

Historically, I’ve tried to avoid writing integration tests whenever I can—they’re slow to write and annoying to maintain. But AI changes the maths and in a recent migration, I was able to write 120+ Playwright tests against a vanilla JavaScript application quickly. Claude inspected the HTML and JavaScript, identified important structural markers and actions, then used Playwright to determine what the side effects of clicking links and buttons should be. This essentially became a functional spec the agent could test against.

AI agents excel at local transformations but lack global context. Tests provide the negative feedback that prevents drift. Without them, an AI can confidently break authentication while fixing a linting issue.

The tests are a primary safety mechanism for the migration. They won’t catch every invariant or performance regression, but they dramatically reduce the risk of silent breakage while the system is in flux.

### Documentation as Context

Tests tell the agent when something breaks. Documentation tells it why things are built the way they are.

Most legacy systems carry invisible contracts—assumptions about data flow, integration quirks, business rules encoded in code nobody remembers writing. Agents often can’t infer these. And when they make guesses, things go off the rails.

We’ve started maintaining agent-focused documentation alongside the codebase: architecture overviews, integration maps, the kind of context a new team member would need but rarely gets written down. The standard CLAUDE.md or AGENTS.md files help, but for brownfield work, you often need more—explanations of why the auth flow is weird, which external services are sensitive to timing, where the bodies are buried.

The documentation doesn’t have to be perfect upfront. We use a /learn command so that when the agent struggles with something, it analyzes the failure and proposes updates to the docs. The system gets smarter as we go. It becomes institutional memory that compounds.

### Incrementalism as Risk Management

Complex systems resist wholesale change. Effective approaches break migrations into discrete, reversible phases. For this recent project, I moved through tooling introduction (build systems, linting, testing), then structural extraction (deduplication, modularization), then framework migration (component by component), and finally design system integration for visual consistency.

Each phase leaves the system in a working state. You might not ship every step to users—feature flags can hide visual inconsistencies while the migration progresses—but you maintain the ability to stop, assess, or roll back at any point. No phase depends on completing the next one to function.

Agents perform best with bounded problems and clear success criteria. “Migrate the upload modal to React” is tractable. “Modernize the application” is not.

There’s a related point worth making here: AI’s speed doesn’t replace refactoring discipline. Just because you’re moving significantly faster doesn’t mean you should abandon what works. Martin Fowler’s refactoring patterns and Kent Beck’s small steps—these remain essential.

This is run-of-the-mill, good refactoring hygiene and is what keeps agent-driven changes small enough that failures are understandable, reversible, and cheap. (Turns out the fundamentals remain annoyingly fundamental.)

### Compromise as Strategy

Technical debt isn’t uniformly toxic. In brownfield contexts, the trick lies in distinguishing between compromises that enable progress and those that cause long-term pain.

```
// Exposing functions globally. Inelegant but functional
window.handleUpload = handleUpload;
// Wrapping legacy visualizations. Imperfect but stable
import LegacyChartRenderer from './legacy/charts';
// BLOCKER: Security issue flagged for fix before merge
// TODO: Replace with proper authentication
if (user === 'admin') { ... }
```
These were conscious tradeoffs that the agent actually identified during the planning phase for one of the migration steps. The hierarchy I instructed the agent to follow: “Security issues and data integrity are non-negotiable—fix them or flag them, no exceptions. Don’t break existing functionality while making changes. Ugly patterns and tech debt are fine temporarily, between steps.”

The challenge with AI assistance is that models default to “best practices” when you need “what actually works.” They’ll often suggest comprehensive refactors that balloon and become impossible to land. What you really need is surgical changes that unlock the next step. The skill is in directing AI toward pragmatic solutions rather than theoretical ideals.

### Structure as Enabler

Tests become the living specification. Phases provide the substrate for safe experimentation. An explicit compromise hierarchy tells the agent what invisible context to preserve. Once these boundaries are in place, the same agent that might have wrecked your system becomes capable of evolving it safely.

The process may look heavy on paper, but it actually speeds things up. A React migration that might have taken a week by hand was completed in a day. There’s also something counterintuitive about speed here: the faster you can fix things, the bolder you can be about breaking them. You spend less time in the grey zone between working states, and recovery costs drop. The structure that seems like overhead becomes the thing that enables both speed and confidence.

One caveat: AI will help you no matter what, but it’s not magic. What I’m talking about here assumes your core architecture is still conceptually valid—the data model mostly fits the business, invariants exist even if they’re implicit, and ownership boundaries haven’t completely collapsed—even if the implementation is outdated and crufty. If your core data model no longer fits what the business needs, things are going to be hard no matter what, you are going to need to take a very careful and systematic approach.

Second caveat: structure enables AI assistance, but it doesn’t necessarily automate it. We’re starting to track “agent autonomy” across projects—the ratio of agent turns to human prompts. Some hit 95%; others sit at 60%. The gap comes down to how well you communicate intent and how much context the agent can access without you.

Legacy codebases carry tacit knowledge that agents can’t reach on their own. Miss the right moment to inject it and you get a 4000-line change full of subtle bugs. The work is more like managing a fast-moving team than running a script. The real skill is systems thinking—holding the whole in your head while the agent works on the parts.

Both caveats point in the same direction: AI can absolutely help with brownfield work. But it won’t rescue a broken architecture, and it won’t supervise itself. Set up the conditions, stay engaged, and it’s remarkably effective. Skip the setup, and you’ll probably end up like that Django team.

This was a genuinely practical take on AI adoption instead of the usual “rewrite everything around agents” narrative.

The part that stood out most to me is the focus on brownfield constraints: existing workflows, operational risk, fragmented ownership, legacy data models, unclear boundaries, and teams that still need systems to remain predictable under failure. That’s the reality most engineers are actually living in, whether LinkedIn AI influencers acknowledge it or not.

I also liked that the article frames AI integration as an architectural and organizational problem, not just a model problem. In production, the hard parts are usually orchestration, reliability, evaluation, permissions, observability, and rollback strategies long before the model itself becomes the bottleneck.

I write quite a bit about distributed systems, DDD, and production AI systems myself, and this was absolutely worth subscribing for. Rare case where the engineering trade-offs are treated seriously instead of being buried under AI hype aesthetics.

Regarding brownfield AI, your observation that 'many legacy systems lack the structure agents need to operate safely' really hit home, offering a sharp perspecitve that makes me think we'll need human ingenuity for those messy systems a while longer.
