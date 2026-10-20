---
url: https://benjamincongdon.me/blog/2026/10/19/Agentic-Monitoring/
date_fetched: 2026-10-20
---

A recent Anthropic blog post contained a short paragraph, not at all the point of the post, which has been buzzing around my brain since I read it:

A new level of operational excellence is possible. Our team uses Claude Tag heavily, which has helped us monitor each component of the system. For example, we created Slack channels where we tasked Claude with being obsessively focused on the health of a single component. We call these Claudes “superfans.” We have a scion-daemon superfan, a scion-cache superfan, an Antique merge queue superfan, and so on. They have access to our telemetry and logs, and act like an experienced engineer who has nothing better to do than to refresh charts and look for anything strange. We also have traditional alerts for all of our systems, but our superfans can detect subtle and novel issues that make these alerts seem one-dimensional by comparison.


This is *so obviously* the path forward for software systems operations. I’ve been thinking about monitoring with agents for the past few weeks, and this paragraph crystallized some of those thoughts.

Modern software systems, like the LLMs that write them, are becoming more “grown” than “designed”. Yes, human engineers often still give the high-level design and shape to the system, and yes, software systems (for now) are much more interpretable than LLMs – since you can just sit down and read the code, if you are so inclined. However, with the volume of code being written and shipped increasing exponentially, the amount of actual eyes-on-code knowledge that any human operator of a system has is decreasing.

Systems monitoring was never *easy*. There is a whole industry of observability SaaS companies – Sentry, Datadog, Splunk, Honeycomb, to name a few. Observability is expensive.1 Doing it well requires a specialized human skill set and relies on building experience operating real systems at scale. One can really only develop these skills by working in industry. This is a domain that takes years to train junior engineers on.

As with seemingly everything in the software industry in 2026, system monitoring is poised for a step change due to agents. I… am surprised I have not seen more people write about this, especially in the current milieu of “agent frenzy”. It’s right there in front of us all, and it *works*. There have been a bunch of startups trying to do the “AI SRE”, but very little writing from practitioners.

But you don’t really need a full “AI SRE” to improve operational excellence, nor do you need a built-out SaaS product to do so. As of the Opus 5.5 / Astra class of models, with a bit of care, it’s entirely possible to make a good agent-driven monitoring system that “just works”.

### Case study: change-point monitoring

I’ll ground this discussion in a problem that is near to my heart: production monitoring of change points, like binary releases and config pushes. The idea is that you want to do specialized, intensive monitoring around critical points of change to a system; these change points are more likely to trigger a regression, and thus if you monitor these events closely, you can limit its impact. To quote the clichéd line from the Google SRE book, “~70% of outages are due to changes in a live system”.

This is a wicked problem. I’ve been working on it in some form since 2023, and it’s been hard to get traction on. From talking with folks across the industry, I’ve learned there are ways to do change-point monitoring “well”, but these methods largely still rely on humans to do the right thing (e.g. tagging metrics/alerts diligently, defining precise signals, etc.) and require significant organizational investment.

More specifically, achieving “takeoff” for change-point monitoring as a piece of platform engineering is hard because:

- Your first instinct may be to target highly motivated users who already have a good ops posture. However, these teams’ rollouts are already mostly safe, and you don’t actually raise the ops waterline much. The thing you’re selling here is *toil reduction*, which is “nice to have” but is not itself a first-order safety improvement. Precision is easier to achieve for highly motivated teams, but this means the change-point system is “competing” on recall and latency with traditional alerts.
- You can alternatively target the broad population of services. In other words, you go super wide and opt in “all services”, on a chosen subset of universal health metrics. In practice, most of these general health metrics are irreducibly noisy. This noise results in poor precision and alert fatigue.
- Prior to LLMs, there was a clear precision/latency tradeoff as well. Along this spectrum are: structured logs (precise, slow), metrics (coarse, fast), and alerts (even coarser, recall is pretty bad and precision is pretty bad, but “at least it’s something”).

Shockingly, agents largely “just” solve this problem. Agents are happy to run copious queries, can write per-change queries on the fly, and will scan logs ad nauseam. As a result, you can get both good precision and recall for cheap. In the basic version of this setup, you just need the ability to connect your codebase (e.g. to reason about config consumption) and your observability stack (e.g. to view and monitor the live production state) to the agent.

I’ve spiked a prototype of exactly this: an agent that proposes a monitoring plan based on the service code, a scaffold that executes that plan, and an agent that investigates triggered monitors, with an escalation path to humans. Even in this early prototype, I’ve been able to detect regressions in systems I have *zero* understanding of with precision and latency that would have been impossible a year ago.

In my iteration on this, I’ve found that a pure agent loop is not the optimal shape for agentic monitors. Rather, a harness that allows the agent to write a spec of how to monitor, along with a deterministic system for running those specs, produces significantly cleaner signals with dramatically lower token usage. To the extent there is “secret sauce” in this pattern, it’s in the noise-reduction heuristics, the prompts that get the agent to write good specs, and a labeled eval set large enough to validate further tweaks.2

### Agents are structurally well suited to monitoring

Two mindset shifts need to happen before this pattern can catch on:

- Agents actually can do a much better job at monitoring than humans.
- Monitoring agents need scaffolding, the same way the rest of the software development lifecycle had to adapt for coding agents. There hasn’t been a “Claude Code moment” for monitoring yet.

Once you make these shifts, monitoring agents have several “unfair advantages” over human operators:

- Ability to track everything, 24/7. If tokens are cheap, then any regression in a metric you care about and any suspicious log line can be investigated – not just the events that would previously have risen above the “human attention cutline”. Agents don’t need to “follow the sun”.
- Ability to fan out and investigate several paths toward resolving a production problem. Agents can parallelize investigations when multiple things go wrong at once, and coordinate quickly. This is already my go-to for incident triage: I just launch a bunch of agents to investigate several potential threads or theorized incident causes. Right now, this is human-driven, but there’s really no reason it couldn’t be agent-initiated in the future. You can elastically scale up a fleet of incident responders.
- We’re already seeing that this is becoming critical for security incidents. Teams/fleets/swarms of agents are being used for offensive cyberattacks, and defenders are beginning to build up similar defensive capabilities (e.g. OpenAI’s Daybreak Blue, Anthropic’s Project Glasswing, Google’s Fairwind).
 
- Ability to respond faster to a problem than can a human. Agents can read code faster than humans. Agents can also query and digest metrics and log signals significantly faster. This speed doesn’t always translate into faster incident detection time; a human with a deep understanding of a system may still be faster at finding the right signal and intuiting the root causes. But to the extent that tacit knowledge can be recorded and made available to an agent, the agent will eventually beat the human on speed.
- By extension, for systems where the human operator isn’t deeply experienced, the “time to understanding” is going to be shorter for an agent.
 

### “A new level of operational excellence is possible.”

**Be more diligent:** You wouldn’t pay a full-time software engineer to look through all the Git commits for a service, pick out things to monitor, and make special short-term fast-trigger alerts just on those changes. However, for a couple cents per million tokens, it’s possible now, and will become an industry standard.

**Continuously improve performance:** Agents can monitor performance, look at heap dumps and stack traces and CPU profiles, and propose optimizations from the patterns they find. Put the proposed improvements behind A/B tests if necessary, and monitor as you would any other change to the service.

In one instance, I observed that a few sessions of Astra/Opus investigation discovered 50% heap savings and 30% latency reduction in a common hot-path library that humans had already spent nontrivial effort to optimize. Once you give a sufficiently powerful LLM a VM and benchmarks to run, the “hill-climbing” ability of current-generation LLMs is remarkable.

**Take action**: Thus far, I’ve mostly been discussing “read-only” monitoring systems. Agents, of course, needn’t be “read-only”. With the right harness and guardrails in place, there’s definitely a future in which agents will be able to take front-line actions to remediate incidents. This could span anything from well-scoped operational tasks like resizing instances or rotating compute pools, to deterministic actions like rolling back a config change or binary, to open-ended tasks like root-causing bugs, proposing fixes, and redeploying on-the-fly. Obviously there is risk the more control you hand-over to the agent, but considering how much agent capabilities expanded over the course of the past 12 months, the amount of write-path work that can be responsibly handed over to agents is bound to increase. The promise of truly self-healing systems is exciting.

**Cost is no longer a limitation.** Luna/Haiku-class models3 are now competent graph-starers. A moderately large tech company could likely monitor its entire fleet for on the order of $100/day. Startup-scale fleets would be a small fraction of that. Compared to human labor, or even the cost of the infra that runs observability, this is trivial.

**Create a virtuous cycle:** If done well, agentic monitoring creates a positive feedback loop: coding agents allow you to ship more, monitoring agents allow you to support more systems, which in turn frees up human time to direct more things to ship. Humans increasingly spend time “building the factory” or “tending the garden”.

Agentic monitoring is worth investing in. It will soon be reckless not to have agents monitoring your systems. Until the rest of the industry catches up, getting started now affords an unfair advantage.

- 
A set of optimizations my team recently completed saved us several million dollars in annualized observability cost, and… that was a mere fraction of what *just my team*spends on observability. ↩︎
- 
Much, much more could be written about the design of agentic monitoring harnesses. This is one of the clearest “we’re so early” feelings I’ve had in a while. General harnesses, like Anthropic’s “Claude Tag”, evidently already do a good job at agentic monitoring, per the original post. However, based on what I’ve seen, even a little monitoring-specific harness engineering picks up a lot of low-hanging fruit. ↩︎ 
- 
Here I mean GPT-6-Luna and Claude Haiku 5.5. ↩︎
