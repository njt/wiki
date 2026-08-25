---
url: https://infrequently.org/2026/08/notes-on-performance-remediation-strategies/
date_fetched: 2026-08-25
---

## The Wicked Reason Removing Code Beats Better Scheduling

### A response to Marko Ilić

My colleague Marko Ilić has published an insightful piece on how to schedule work on the critical path, and I recommend reading it before proceeding here.

...and welcome back.

As you likely anticipated, Marko's beautifully presented post kicked off healthy discussions around the office. Since we almost always see eye-to-eye, it seemed interesting to surface some of that discussion here. With his permission, what follows are expanded versions of some points I posed in reply.

## Elimination vs. Scheduling: An Organisational Perspective

There's no daylight between our positions on code reduction: sending and running less is always ideal.

Where we differ, perhaps, is the priority teams should assign to scheduling vs. code removal in their performance remediation efforts.

While it may be hard to remove code, my view is that it is generally not much harder re-ordering. Why? The primary cost of both code removal and scheduling interventions is the investment to deeply understand page behaviours. This presents a narrative challenge to the proposition that scheduling is an easier fix, as both approaches share the largest cost. Ergo, changes to resource ordering cannot be assumed cheaper or easier.

At the team level, the consequences of restructuring are *harder to reason about* than direct removal. The bytes that are only deferred still land on the main thread, potentially delaying above-the-fold resources in ways that only become visible in the tail of the connection quality curve. Worse, JS resources that are fetched late generate heavy "thuds" when residual allocations from background compilation arrive on the main thread. These stalls show up in INP data, but can be maddening to track down due to their stochastic relationship with other resources and the unpredictable scheduling of main-thread tasks in the real world.

Explaining these impacts is an ongoing challenge. Working across teams to preserve carefully constructed code ordering requires exquisite regression-prevention discipline. This presents an elevated degree of operational difficulty vs. code removal, where simple bundle size clamps can help to maintain gains.

From the geosynchronous perspective, it becomes easier to see that reordering exacerbates coordination challenges, and per Fred Brooks, those communication effects dominate the shape and quality of the pages a team delivers.

In large enough teams and codebases, creating and maintaining an optimal ordering is a question of global priorities that can only be elucidated by product owners. As they tend not to be the line engineers, their goals will be translated to code in ways they will not fully understand, and the deep implications of that translation process may only become clear in retrospect. It was ever thus in the pin factory.

Goals may also be communicated in different intensities and formats, and change over time. Worse, teams often misidentify their targets. Many groups I've worked with have confidently asserted that *"everyone has 5G"* or *"everyone's on fast devices,"* despite neither their own dashboards nor the global baseline supporting that presumption. Teams in this situation can easily make incorrect assumptions about the network, CPU, and memory limits of users at the margins. Getting in touch with those limits is straightforward for interested managers, and it's a big part of climbing the Performance Management Maturity ladder. Leaders working through this landscape for the first time are frequently shocked to find just how far out-of-the-money their products have become.

Regardless, many teams build to (or past) it blithely, then spend tremendous amounts of effort re-ordering code without achieving durable gains or increasing TAM. From this market-centric view, removing code has the benefit of making the system more resilient to changing conditions, including shifts in user population due to product growth. This makes code removal *particularly* potent under uncertainty about user populations, which is where nearly every web project begins life.

Systems that rely on scheduling to assure reasonable performance are brittle. Reordering work to appear responsive for one user journey can help, but such systems have a comparatively harder time adapting to changing needs — something Marko's examples work through, but with only one alternative explored. In reality, products that need this level of attention face an evolving field of possibilities across an expanding range of users and devices, while an increasing stable of features contend for priority.

Teams hoping for performance headroom through scheduling can also reduce their agility through use-case overfitting. Latency concerns become a straitjacket when multiple critical user journeys funnel through a single chokepoint that must balance the needs of all. This should bias remediation toward removing code and breaking up experiences MPA-style until a team is *far* into the tail of diminishing returns.

Not only will those improvements prove more durable and easier to defend, but smaller systems are also easier to reason about. Code size reductions set the stage for more effective scheduling fixes when a team is ready to switch approaches from a position of confidence.

### What's Good for the Goose...

An underappreciated headache of systems drawn taut around available bandwidth and CPU constraints is that they aren't just difficult for engineers to wrangle; they also appear brittle to managers.

Busy EMs and PMs experience systems in this state as teetering on the edge of usability while soaking up huge amounts for "optimisation." Managers can even end up in learned helplessness as regressions crop up in curious parts of the experience with each seemingly unrelated change. The interplay of browser scheduling, network conditions, and device constraints is extremely fluid, making scheduling fixes the less bankable alternative, especially at page load time. Teams that spend the same quantum of effort eliminating code, by contrast, have more wiggle room and can be more agile over time. Managers that buy into the narrative that reordering is easier than removal deal their future selves a losing hand.

*Ceteris paribus*, the level of difficulty from an enforcement, regression-prevention, and span-of-control perspective is *higher* for re-ordering vs. code removal, and savvy managers should always favour investments that will cut code or move it to the server, rather than pushing peas around the plate over the wire or on the client.

### "Preloading" as Scheduling

A particularly grim failure mode of scheduling as the go-to performance remediation approach plays out in large organisations through competitive use-case devaluation. Individual feature owners advocate for their code to be loaded early, implicitly deferring or delaying code required for other user journeys. This interacts poorly with haunted SPA architectures that force code-loading priorities to play out over an unknowably slow or laggy network and onto a device whose memory and CPU properties are a mystery. In the tail of the distribution, SPA architectures assure that every choice to fill the channel with Team A's code harms Team B's experience.

Bun fights inevitably break out as a degraded commons raises the stakes on every team, backing managers into increasingly strident advocacy for their feature. Because engineering teams in this situation are generally not imposing code size budgets (thanks in part to the mistaken belief ordering will save the day), the consequences for de-prioritisation grow over time. The negative feedback loop is obvious. The notion that scheduling is easier than code removal steers the product into a coffin corner sure as day follows night.

But it's even worse than it sounds.

Because the implications of deferring other code may be hard to reason about, devaluation of a product's core experiences can spiral, unopposed, for years. In this twist on the general failure mode, "special interests" degrade an undefended commons. Thanks to compilers and code motion in "modern" frontend codebases, order-dependent optimisation approaches and "preloading" hacks can hide in plain sight, ballooning costs for all users as seemingly innocuous library code is added to `XXXXXX-common.js` and `YYYYYY-vendor.js` without opposition.

A common attribute of low Performance Management Maturity organisations is an absence of processes to push back on this sort of thing. The result is a Gordian Knot that only ever grows more tangled. The linchpin is not doubling down on code ordering, but a decision to get wire size under control.

An unreasonably effective down payment on this commitment is to evict all "preloading" and "deferred loading" systems from a product, size it up as it stands, then force trims. This isn't always the right approach (consulting engineers should always be trace-driven), but for teams that have followed the logic of preloading down the primrose path, it's a bracing corrective that creates accountability through visibility. From that point, *"we don't do that here"* must become the manager's mantra whenever a feature PM repeats discredited paeans to scheduling, otherwise the product will end up back in the drink.

### Scheduling Is For Closers

Unlike removed code, the impact of reordered code is not easy to reason about. Innocuous-looking scheduling changes like `loading="lazy"` for `<img>` and `<video>` can go horribly wrong. Without system knowledge, infrastructure, and culture to support investigations, it's much more difficult to know if work reordering improves end-user perceived latency.

All of this is to take nothing away from the thoughtful technical advice in Marko's post. We *can* dramatically improve critical-path loading and responsiveness of experiences by better ordering resources, and we *should* strive to only do work proportional to what's currently on-screen, above the fold.

But that set of opportunities should be treated by engineers and managers as special occasion food. While both approaches can, in theory, yield similar gains per hour of effort, scheduling changes only pan out in systems that already have strict size budgeting, regression prevention, and hawkish launch gates. Unless a team is *at least* demonstrating the regression prevention attributes of Level 4 Performance Management Maturity, investments in better ordering are a management error. Teams without a strong hand on the rudder and battle-tested early warning systems should, instead, focus almost all of their effort on reducing complexity and code size.

Investments in size reduction also tee up future scheduling improvements, teaching teams about the interplay of system components. Without deep insight into site behaviour, it is challenging to select good opportunities for reordering, and the learning inherent in code reduction invariably helps identify ripe scheduling wins. Likewise, reductions in payload complexity make it simpler and more effective to test the impact of reordering.

In other words, going from decent to excellent performance is a technical challenge, but moving from poor to decent performance is fundamentally a management and culture problem.

Marko's piece is excellent fodder for teams that have a technical problem. I wish that represented more than a sliver of the organisations I've consulted with.

## General-Purpose Advice

Taken together, these factors give rise to the structure of the advice I dole out to teams looking for help:

- Do everything possible to reduce code sent to the client; even the stuff that seems "dirty."
- Move work to the server where possible.
- Only think about restructuring work when #1 and #2 reach diminishing returns.

Management challenges also give rise to our request that teams enunciate "critical user journeys" and put thought into who their marginal user *really* is. Just getting leads and PMs to agree on which users shouldn't be excluded due to poor performance and what the most important flows are can create clarity and value, independent of which approach to alleviation a team pursues.

There are a lot of problems that can be ameliorated with better scheduling, but the level of affirmative control a team has to have over a system to effectively execute on those tactics is shockingly uncommon. Therefore, most teams, most of the time, are better off thinking about how to remove code from their bundles.
