---
url: https://www.lucavall.in/blog/platform-engineering-end-to-end
date_fetched: 2026-07-05
backfilled: true
---

Most engineers I talk to either think platform engineering is "DevOps with a portal" or "the team that owns the Kubernetes cluster". Neither is wrong, but neither is right either. After reading *Platform Engineering: A Guide for Technical, Product, and People Leaders* by Camille Fournier and Ian Nowland, and after a few years building and supporting these things on GCP, I think the discipline is best understood as one boring sentence: **a platform team builds and operates an internal product whose users are other engineers.** That sounds simple. It is not.

This post walks the full arc of the book in my own words, so you can decide whether platform engineering is something you actually need, and what it looks like when it works. Assume you already know how to ship production systems. I am not going to over-explain the basics, but I will make sure each idea answers "why does this exist".

For context, the 2025 DORA Report found that 90% of organizations have adopted at least one internal platform, and platform quality is now a direct predictor of whether AI tooling produces value or chaos. This is no longer a fringe topic.

## Why Platform Engineering Exists

### The Over-General Swamp

Cloud and OSS gave us infinite primitives. Need a queue? You have twelve. Need an object store, a database, a CI runner, a service mesh? Pick a flavor. Each application team picks differently, and a year later your "infrastructure" is a swamp of glue code where every service has its own deploy pipeline, its own retry logic, its own monitoring conventions, its own subtly wrong IAM bindings.

The book calls this the **over-general swamp**. Two changes pushed us into it: the explosion of choice (every primitive on every cloud), and higher operational expectations (24/7 uptime, security, compliance, cost control). Each app team handles all of it themselves, badly, in parallel.

I have seen this firsthand on landing zone projects where every product team was reinventing the same Terraform modules for VPCs, IAM, and budget alerts. Twenty teams, twenty almost-identical implementations, each with its own bugs. Sad noises.

### What Platform Engineering Actually Does

Platform engineering clears the swamp by doing four things:

- **Limits the primitives**developers see. You don't get raw GCS plus raw Pub/Sub plus raw Cloud Run; you get a curated, opinionated way to use them.
- **Reduces per-application glue**by absorbing the repetitive plumbing into shared services.
- **Centralizes the cost of migrations.**When the underlying primitive changes, the platform team handles it once, not every app team handling it 50 times.
- **Lets developers operate what they build**without forcing them to become Linux kernel hobbyists.

This is also why platform engineering is not just DevOps renamed. DevOps said "developers, take ownership of operations". Platform engineering says "fine, but we will give you good tools to do that, and treat those tools as a real product". The DORA 2025 capability page puts it well: it is a sociotechnical discipline, not a tooling category.

## The Pillars

Five things make a platform team a platform team and not just an infra team with a Jira board.

### Curated Product Approach

You decide, with intent, what your platform supports and what it does not. If a team wants Kafka instead of Pub/Sub, the answer is not "sure, here is the link to the docs". The answer is "here is what we support, here is why, here is the off-ramp if your case really does not fit". Saying no is part of the job.

### Software-Based Abstractions

The platform is software, not a wiki. The interface to it is APIs, CLIs, and SDKs. Your developers should be able to provision a production-grade service by writing a small declarative file, not by clicking through a console or pinging Slack.

This is where the Score project, now under CNCF, gets interesting. A workload spec like:

is enough for a platform to provision the right database, the right topic, the right service account, and the right deployment. The developer does not care that under the hood it is Cloud SQL, Pub/Sub, and Cloud Run. That is the point.

### OSS Customizations and Metadata Registries

Two things make platforms feel real instead of brittle.

The first is OSS customization: you do not run vanilla Argo CD or Backstage. You run them with the plugins, default policies, and integrations that match your org. The second is a metadata registry, usually a service catalog. Without it, you have no idea who owns what, what depends on what, or what is actually running.

Backstage is the de facto OSS framework for this layer. Over 270 organizations run it in production, and CNCF launched both the Certified Backstage Associate and the Certified Cloud Native Platform Engineering Associate certifications. Whether you use Backstage, Port, Cortex, or roll your own, you need a single source of truth for "what services exist, who owns them, what they depend on".

### Serving a Broad Base, Not Just the Loud Customers

Internal platforms have a small number of very loud customers. The senior team that runs the highest-traffic service will demand exotic features. Resist. The platform exists to serve the median developer doing the median task, well. If you build only for the elite users, the long tail of teams will work around you, and that is how shadow platforms are born.

### Operating as Foundations

If your platform is down, the company is down. That changes a lot: 24/7 on-call, real SLOs, real change management, support burden. You are not "a tool"; you are the floor. Anything built on top assumes the floor holds.

## When and How to Get Started

### Don't Form a Platform Team Too Early

At 10 engineers, you do not need a platform team. You need cooperation. One person owns the deploy scripts, another owns the Terraform, you all agree on conventions, and that is enough. Forming a "platform team" of one or two people too early just turns those people into a ticket queue and makes the rest of the org passive.

The book is explicit: at small scale, foster cooperation. Form the team only when the cooperation model is visibly breaking, usually somewhere past 50 engineers, when you start having multiple deployment targets and no one knows the canonical answer for "how do I ship a new service".

### Transforming a Traditional Infra Org

If you already have an infrastructure or SRE team and you want to turn it into a platform org, the hardest part is not technology. It is culture. Infra people are used to being the gatekeepers of "no". Platform people have to become the providers of "here is the easy yes". That means:

- Talking to customers, a lot. More than they are used to.
- Hiring or growing software engineers who like building tools, not just operators.
- Updating recognition and reward so that "I made 200 teams 5% faster" beats "I deployed a new cluster".

Don't just sprinkle product managers on top and call it done. That is the most common failure mode and it produces theater, not platforms.

## Building Platform Teams

### The Four Roles

The book splits platform engineers into four buckets, and the split is useful:

- **Software engineers**build the platform's product surface (APIs, SDKs, portals).
- **Systems engineers**know the underlying primitives deeply (Kubernetes, Linux, networking, the cloud control plane).
- **Reliability engineers**focus on operational quality, on-call, SLOs, observability.
- **Systems specialists**are the deep-domain experts (databases, security, networking).

You need a mix. Too much software focus and you ship a beautiful portal that falls over under real load. Too much systems focus and you have a rock-solid cluster nobody can use without filing tickets.

### Hiring for All of It

Hire for customer empathy. I cannot stress this enough. A platform engineer who cannot sit on a call with a frustrated app dev and walk away with a clear understanding of their problem is in the wrong job. Technical brilliance without empathy produces platforms that are correct and unused.

Allow role-specific titles. Use the same level matrix for software roles, but be flexible for systems specialists, where market value and skills don't always map cleanly to a software engineer ladder.

### Managers and Other Roles

Great platform engineering managers tend to share three traits: they have actually operated platforms (not just built them), they have shipped long-running multi-quarter projects, and they are obsessive about details. Platforms reward attention to detail. The 1% of cases you skipped because they seemed rare will eat 80% of your support time.

Product managers, technical writers, developer advocates, and support engineers all matter. But hire them only when the engineering team is mature enough to use them. A premature PM on a 4-person platform team becomes a roadmap-shaped chair.

## Platform as a Product

This is the chapter that most non-believers should read. Treating the platform as a product is not branding. It is a working stance!

### Customers Are Internal And That Makes It Harder

Internal customers are weird. They are captive (they cannot churn easily). They have strong opinions and weak product instincts. They will tell you what they want, which is often not what they need. They will ask for the platform to do their job, not give them tools to do their job.

Empathy still wins. Sit with them, watch them work, count how many times they have to context-switch to ship a single change. That is your real backlog.

### Discovery, Roadmaps, Failure Modes

Platform discovery is messier than consumer product discovery. You don't run A/B tests, you run pilots. Validate new investments by actually deploying with a friendly team and measuring whether their lead time drops, not whether they smile in interviews.

A working roadmap has four time horizons:

- **Vision**(multi-year): where this platform is going.
- **Strategy**(annual): the bets you are making this year.
- **Goals and metrics**(quarterly to annual): what success looks like.
- **Milestones**(quarterly): what you will actually ship.

Common failure modes from the book that I have seen wreck teams:

- Underestimating migration cost (always 2-3x what you think).
- Overestimating the change budget your users have for new features.
- Adding features when stability is the actual problem.
- Too many product managers for the size of the engineering team.

If your engineering team has 5 engineers and 2 PMs, you are in trouble.

## Operating Platforms

### On-Call Is Not Optional

Platforms operate as foundations, so 24/7 coverage is not negotiable. The DevOps mantra "you build it, you run it" lives here. The team that builds the deploy system is also the team that gets paged when it breaks at 2 a.m. This is not a punishment, it is the feedback loop.

Practical advice that always applies: keep on-call sustainable. If a single engineer gets paged more than a few times a week, fix the system, not the schedule. Burned-out platform engineers ship bad platforms.

### Support: The Hidden Half of the Job

Support work is the ugly half of platform engineering nobody talks about in conference talks. The book lays out four stages:

- Formalize support levels (P0 vs P3, response times, etc.)
- Separate non-critical support from on-call so that "how do I add a CronJob?" doesn't wake someone up.
- Hire a dedicated support specialist when volume justifies it.
- At scale, build a real engineering support org.

If you skip stage 1, your engineers spend half their time answering Slack DMs and half their time being grumpy about it.

### Operational Feedback

SLOs and SLAs are necessary; error budgets are nice but optional. Synthetic monitoring catches the failure modes your users hit before they file a ticket. Operational reviews force the team to actually look at the data and not just glance at a green dashboard. The DORA 2025 data found the platform capability most correlated with positive user experience is **clear feedback on task outcomes**, which is just a fancy way of saying: when something fails, the user should know exactly what failed and what to do about it.

## Planning and Delivery

### Long-Running Projects Need a Proposal Document

Platform projects are big. Migrations, rearchitectures, new control planes. They take quarters. Skip the proposal step at your peril.

A good proposal answers: what problem are we solving, who benefits, what is in scope, what is explicitly out of scope, what does success look like. Write it before you write any code. Get it reviewed. Then turn it into an action plan with concrete milestones. The "long slog" failure mode (project drags on for years, nobody remembers why) is almost always traceable to skipping this step.

### Bottom-Up Roadmap Planning

Platform roadmaps are weird because they include four kinds of work: keep-the-lights-on, mandates from leadership, system improvements you decide on, and direct customer asks. You cannot just rank by customer demand. KTLO comes first, mandates come second, and the rest is a fight you have to have honestly with stakeholders.

### Biweekly Wins and Challenges

This is one of the most underrated practices in the book. Every two weeks, the team writes a short document: here is what we shipped (wins), here is what we are stuck on (challenges). Short, public, no fluff. It does three things at once: it forces the team to articulate progress, it tells stakeholders what is actually happening, and it surfaces challenges early so leadership can help. Don't skip the challenges. A document with only wins is a document nobody trusts.

## Rearchitecting and Migrations

### Rearchitect, Don't Build a v2

The instinct when a platform gets crufty is "let's build v2". Almost always wrong. v2 projects fail because they freeze investment in v1, they take longer than estimated, and migrating to v2 costs more than the rearchitecture you avoided.

Rearchitect inside the existing platform. Keep compatibility as long as you can. Use lower environments, slow rollouts, tranche-based migration. Stay a version behind in production while you stabilize the new code path in staging.

The four planning steps from the book are good:

- Think big about the final architecture goals.
- Factor in migration costs (always 2-3x, did I mention?).
- Identify the major 12-month wins that justify continued investment.
- Get leadership buy-in, and be prepared to wait.

### Security Is Architectural

You cannot bolt security onto a platform after it is built. The architecture has to enforce least privilege, isolation, and traceability by design. If your platform requires every team to remember to set the right IAM bindings, the platform has the bug, not the team.

### Migrations: The Underrated Hard Problem

Migrations are where platforms either prove their worth or expose their lies. The most common antipatterns:

- Asking every team to do the migration themselves with a clipboard and a deadline.
- Mandating without providing clear on-ramps and off-ramps.
- Underestimating the long tail of weird use cases.

Engineer easier migrations by:

- Tracking usage metadata so you actually know who is on the old version.
- Building automation, not clipboards. If 200 teams need to migrate, the platform team writes the script, not the app teams.
- Architecting for transparent migrations (the new system speaks the old API while you switch the backend).
- Documenting on-ramps clearly enough that a new team can self-serve.

Use mandates sparingly. Mandates work once or twice, then they become noise. Most of the time, make the new path so much better that the old path withers.

### Sunsetting

Don't be afraid to kill products. A platform with seven half-supported deploy systems is worse than one with one solid deploy system. Sunsetting is hard but it is what mature teams do.

## Stakeholder Relationships

Platform teams have a uniquely brutal stakeholder map. Every engineering team is a customer. Most VPs have an opinion. Finance cares about cloud spend. Security cares about everything.

### The Power-Interest Grid

Map your stakeholders on two axes: how much power do they have, and how interested are they in what you do. The high-power, high-interest people get regular updates and consultation. Low-power, low-interest people get a status page. Don't waste your team's time keeping a low-interest VP informed about Kubernetes upgrades.

### Communicating Without Oversharing

Be transparent, but not exhaustively transparent. A senior leader does not need to know that you debated three different gRPC retry strategies. They need to know whether you will hit your milestones and what risks you see. Use 1:1s judiciously. Track expectations and commitments somewhere visible.

### Saying No Without Wrecking the Relationship

The hardest skill. Be clear about the business impact of saying yes ("if we add this feature, our migration slips by a quarter, which costs the company $X"). Sometimes you say "yes, with compromises". Sometimes you say no but offer a path. Sometimes you tolerate a shadow platform because the cost of fighting it is higher than the cost of letting it run.

Money management matters too. When budget season comes, don't go person-by-person. Group by team and by capability. Come with strong opinions about what to cut and what to keep. If you don't, finance will pick for you and they will pick wrong.

## What Success Looks Like

The last four chapters of the book describe successful platforms with four properties: aligned, trusted, manage complexity, and loved. This is the part most posts on platform engineering skip, and it is the most important.

### Aligned Platforms

A successful platform org has multiple teams that all pull in the same direction. Alignment of purpose (everyone knows why the platform exists), alignment of strategy (the bets are coherent across teams), alignment of plans (the milestones don't conflict).

This sounds obvious until you have a runtime team that wants everyone on Kubernetes and an observability team that wants to support every framework under the sun. Misalignment shows up as conflicting customer guidance, duplicated work, and angry developers caught in the middle. Resolving it takes principled leadership, not consensus.

### Trusted Platforms

Trust is built slowly and lost in a single bad migration. Trust comes from how you operate (do you communicate when things go wrong, do you keep your commitments), from how you invest (do you ship the big bets you sold), and from how you prioritize delivery.

The book has a great case study of "the overcoupled platform," where a team built so much custom logic into their platform that any change took months. The fix was not more engineering capacity, it was challenging the assumptions about scope. Sometimes the trust problem is that you are doing too much, not too little.

### Platforms That Manage Complexity

Software complexity is unavoidable. Accidental complexity is not. The book draws a line between the irreducible complexity of the problem and the accidental complexity teams add through sloppy human coordination, shadow platforms that solve the same problem twice, and unbounded growth.

Three practical levers:

- **Control growth.**A platform that supports everything supports nothing well. Be explicit about scope.
- **Use product discovery**to figure out what to stop doing, not just what to add.
- **Manage the shadow platforms.**When a team builds a parallel solution, that is a signal: either your platform is missing something real, or someone is empire-building. Both need a response.

### Loved Platforms

The last chapter is, somewhat against type, about whether developers actually love your platform. Love can look like three things:

- **Love that just works.**Most users won't notice the platform at all. Things ship, the deploy works, the CI passes. Boring is the highest compliment.
- **Love that looks like a hack.**A small thing that delights, like a CLI command that does the obvious right thing without ceremony.
- **Love that's obvious.**Survey scores, retention, organic adoption, people recommending the platform to other teams.

If your platform is loved, you can ask for a budget and people will fight for you. If it is tolerated, you are one bad incident away from being replaced.

## What This Means in Practice

If you are starting from zero or rebuilding, the priorities are roughly in this order:

- **Decide what you support and what you don't.**Write it down. Defend it.
- **Invest in software abstractions, not wiki pages.**Score, Crossplane, your own SDK, whatever fits, but real software with real APIs.
- **Stand up a metadata registry.**Backstage or otherwise, you need to know what runs where and who owns it.
- **Build for the median team, not the loudest one.**
- **Treat operations as a first-class feature.**SLOs, on-call, support tiers, all of it.
- **Hire for empathy as much as for systems chops.**
- **Communicate ruthlessly.**Biweekly wins and challenges, transparent roadmaps, honest stakeholder management.
- **Cut what you don't need.**Sunset, consolidate, say no.

The DORA data is consistent: platform quality is now a multiplier on everything else, including AI adoption. A bad platform makes AI tools amplify chaos. A good platform makes them amplify throughput. Build the floor before you build the rocket.

## Closing

Platform engineering is not glamorous. It is one of the few engineering disciplines where the highest praise is "I forgot you existed". If you want to build something that real engineers depend on every day, that compounds in value over years, and that you can defend to a CFO with actual numbers, this is one of the most leveraged places to spend a career. Just don't expect anyone to send you a thank-you note when the deploy works.

If you want to go deeper, read the Fournier and Nowland book. It is the clearest treatment of the discipline I have come across, and most of what is good about this post is downstream of it. The mistakes are mine.
