---
url: https://www.infoq.com/articles/rightsizing-platform-engineering/
date_fetched: 2026-09-20
---

### Key Takeaways

- When building an internal developer platform (IDP), start with the bottlenecks that slow software delivery rather than trying to build a comprehensive platform.
- Prefer opinionated golden paths with well-defined escape hatches over supporting every possible use case.
- Invest in platform capabilities that reduce duplicated effort and operational toil across teams.
- Treat platform engineering as an evolving product (i.e., “platform as a product”), continuously shaped by user feedback and changing organizational needs.
- Choose the simplest platform that solves your organization's problems; successful platform engineering is measured by improved delivery and reduced cognitive load, not by the number of platform features.

Shift-left and DevOps have had a great impact on the way we can flow changes from inception to production, but at the cost of a higher cognitive load and duplication of efforts like testing, security, and maintenance. If your organization has been around for a few decades, this effort easily becomes an order of magnitude more difficult when you consider the technical debt and cultural barriers that would already have caused friction to begin with.

This article expands on How Much Platform Is Enough Platform presented at KubeCon EU 2026 in Amsterdam and explores real-world challenges of rightsizing developer platforms and finding a cultural match for engineering teams that use the platform to reduce cognitive load and deliver change faster.

## The Beginnings of a Platform

At Wehkamp, we write and run a lot of software across logistics, data science, and e-commerce. As one of the largest and oldest online department stores in the Netherlands, our mixed legacy of business processes and technologies naturally created siloed departments and duplicated engineering efforts. Due to the nature of work handoffs, no team was able to improve or optimize beyond their own scope, with most efforts ending up creating competing standards.

About ten years ago, we made a breakthrough when leadership set a clear goal: Release weekly rather than quarterly to deliver features faster. We created a new department, which ended up absorbing the existing tech organization and drove our first big change with public cloud, containers, and zero-handoff engineering. As long as you could emit a working container and make git commits, nobody would be in your way to deliver changes.

With such great power (or ownership) also came great responsibility; teams managed their resource consumption and runtime scaling and could roll back to previous versions (or fix forward) after a bad release. This capability had some downsides when compared to just handing off your work. You now have to know more about the systems that actually run your software, work with the observability systems, and sometimes debug in live production environments.

The initial platform lacked product thinking and abstractions. We assumed that if you worked with the "modern stack", you would know your way around. In practice, not everyone knows everything and not everything is documented equally well. As a result, engineers were wasting energy on operational toil. It sometimes felt like some of the historical handoff process had translated into everyone having to repeat the same R&D to get something working.

## Victims of Our Own Success

In a sense, solving for one kind of toil shifts focus to the next kind of toil, just like solving for a bottleneck in a complex system realistically just makes something else the new bottleneck. We generally notice this problem when the questions and tickets we receive shift from basic usability to edge cases and new features, but it is also reflected in fewer incidents and better cost control. The adoption of stronger ownership and better delegation of responsibilities had become part of the culture, but with the improved rate of change we quickly outgrew what we considered to be "the platform".

We were spending an increasing percentage of time on classic operations like creating databases, changing disk sizes, and other general resource management, not because it was taking longer, but because what used to be a small part of a big release was now becoming a point of friction in a large number of small releases. At some point we started dividing up our sprints between "real" work and "ops" work to make sure we had dedicated time to build new functionality and improve existing systems to keep up with demand.

As we were scaling our container workloads and getting close to one hundred releases per week, it would take many days to get around to resource management. This time delay wasn’t exactly slow, but did not match our release rate. Some teams began writing their own automation, which often resulted in automated pull requests and, sometimes, slightly duplicating Infrastructure as Code systems like Terraform and Ansible. This approach also involved some unwanted risk. Not everyone used the same standards for things like tagging, access control or deprovisioning; not every team had the capacity to use this approach.

## Evolving Towards Platform Engineering

We knew something had to be done to both free up some load on the platform team and to allow the change rate for infrastructure to grow in tandem with increased usage. The first steps were simple yet effective: We created a chatbot to generate pull requests from preconfigured Terraform modules and text templates with values provided by the user.

**Figure 1: Self-service workflow using configurable templates. (Source: created by the author.)**

Atlantis reacts to the opened pull request and plans the Terraform changes, which, if successful, can be automatically applied to the infrastructure and merged into the main branch. End-to-end, the user would have their resources in about a minute. This approach enables both better self-service and reduces the number of implementation variations with common defects such as missing lifecycle policies or mismatched tags.

Less variation also means simpler upkeep and a smaller set of things to learn and remember, which allows us to build deeper expertise for a smaller horizon. This approach has a secondary effect for us: By making most operations easier and faster for everyone involved, we could start enforcing some policies that weren’t always adhered to that well for productivity reasons First, local state is not allowed in containers; assume they can disappear at any time. Also, access to shells on compute nodes is not allowed.

Those weren’t goals by themselves, but used as milestones and quality guides. If we provide a system that is functionally reliable to the point that these goals are no longer a problem, it would indicate that we have reached major improvements in developer enablement, robust runtime environments, and predictable outcomes.

Right around that time, concepts like Platform Engineering and the Internal Developer Platform (IDP) became more widespread. These practices and concepts were better defined than our ad-hoc shift-left-and-fix-later path. While we were moving from one hundred releases per week to one hundred releases per day, this difference became more apparent because the change rate wasn’t just releases of the existing services. The commissioning and decommissioning of services grew just as fast, making inventory, resource provisioning, and template management a growing point of friction.

## A New Platform

Our navigation towards a more mature platform model was primarily focused on technology and product delivery. The organization was ready and eager to see what we would come up with because everyone had already seen and experienced the benefits of faster development flow from our earlier transformation. Their anticipation gave us the buy-in we needed for this next stage.

On top of our own provisioning systems’ shortcomings, the container orchestration stack was nearing deprecation. Doing all the maintenance ourselves was beyond our capacity. At the same time, feature requests were becoming harder to implement because the underlying infrastructure required for things like workload identities and dynamic scaling simply didn’t exist. Realistically, we needed a new platform. But where were we to begin such a monumental task?

At a recent KubeCon, we learned that other teams had the same problem, but that their approaches varied wildly. For instance, in the presentation Starting and Scaling a Platform Engineering Team, a team built a new multiregional KV store as a platform foundation because, from their perspective, it would greatly simplify and unify data querying as well as storage for other teams. That is a very different approach from Spotify’s Backstage, which was more geared towards discoverability, self-service, and developer experience.

Determining ahead of time what kind and scope of platform to build is difficult. You already need an internal concept for what a platform even is to your organization and to discover your capacity to build it. During our first attempts, we focused either on resource provisioning, on runtime services, or even spent an entire sprint on user interfaces and processes, only to conclude that it is too much to do all at once. Getting started is hard!

If we view those two platform examples as extreme ends of a spectrum, Wehkamp wants to sit somewhere in the middle:

**Figure 2: A balance between technical details and collaboration. (Source: created by the author.)**

While platforms have many facets, we used this perspective to reason about the first half of what we were looking for: the technical delivery aspect. On one end, a platform could be a foundation of primitives that users build their applications with. On the other end, a platform wouldn’t provide solutions directly. Rather, it would facilitate a catalog and scaffolding system for a user community to bring their own building blocks.

The second half is the community factor. Can we get cooperation or contribution to flow in both directions, between a platform team and platform users? While some platform facilities are top-down in nature, on average we are somewhere in the middle:

**Figure 3: Additional balance axis showing autonomy and requirements. (Source: created by the author.)**

Finding our own place was not as easy as it could have been if we had a graph like this ahead of time. Our earlier goals and ideas were based on the classic divide between development and operations, which is about as far removed from the middle as it can be.

Because responsibility and accountability used to be tied to individual silos, sharing tools was risky. If a provisioned resource had an issue, whoever built the automation was blamed, leaving the implementing party with all the risk and little benefit.

To break free of those assumptions, we pushed our perspective towards facilities and ownership. We split platform resources into two governance types: consume-only and multiparty.

A consume-only resource is usually something you build with as a service owner but would never need to define yourself, such as the public cloud infrastructure or the observability systems. Multiparty resources mostly cover crosscutting concerns, like messaging and traffic, which require direct involvement from all participants, who only work well when everyone follows the same rules.

When we combine the governance models and the graph we have, we can start to work out where our platform and engineering concerns fit:

**Figure 4: Placing various examples on our chosen governing model. (Source: created by the author.)**

If a capability lands near the top, the platform should provide it directly, especially in the top left corner where governance is consume-only. In contrast, something like an internal SDK is both provided and consumed by engineering teams. This model also fits things like Apache Kafka where topics and messages are multiparty, but the brokers and storage are consume-only, which makes it fit in the top right corner.

## The Current Model

Because supporting every possible resource with every configuration permutation isn’t feasible for us, we have two base rules. First, strong foundations will get you where you need to go (golden paths). Second, apply or explain (you can do something else, but you have to have a good reason).

This approach doesn’t prevent local variants or experimentation, but it also doesn’t make it friction-free; you’ll have to arrange collaboration, budget, and expertise or find some other way to show that your needs can be fulfilled.

**Figure 5: Increasing effort and proof of need for diverging choices. (Source: created by the author.)**

It looks a bit like a crescendo. The more custom your solution gets, the better your story and resources will need to be to build it. The inverse also applies. If you stay close enough to the golden paths, you’ll get almost everything instantly with very little effort.

To get there, we started out with the core features that are application-delivery-centric to us, which would otherwise take a lot of effort without being a differentiator to the business side:

- You need a Git repository
- That software in the repo to be built
- When software is built, it needs packaging and storage
- Packaged software needs a place to execute
- While the software runs, you will need to monitor results
- To make the software useful, it might have dependencies, such as HTTP traffic, object persistence, and Kafka messaging and queues.

Each of these somewhat abstract needs has concrete implementations and backing technologies. As long as they are provided for, most application archetypes can be built from start to finish without raising tickets or waiting for a team to act.

Self-service and provisioning is similar to our previous process, but instead of a text chat interface, it’s a direct integration with a GUI for instant onboarding:

**Figure 6: Interface for using higher-level building blocks. (Source: created by the author.)**

As a platform user, you can pick from a menu of common building blocks that are provisioned in near real-time, without having to manually run Terraform, access GitHub, or use a cloud console as we might have in the past. It will still commit the changes to Git and let Atlantis handle Terraform and branch merging.

This is not the only (or best) way to manage infrastructure or specifically application dependencies, but it allows us to break glass at any point in the process and reuse knowledge and experience we already have. Git commits can be done manually, infrastructure tools can be executed manually. The configuration itself can be written by hand if needed.

As an application evolves, resources might need to be adjusted, which can be done using the same underlying GitOps workflow. Because such adjustment happens infrequently and rarely blocks change delivery, it doesn’t warrant engineering a custom GUI feature and complex backend service. Because the configuration lives in Git, making a change is a matter of opening a pull request. Resources are scoped to an application and the responsible team, so with a single review, Terraform can be automatically applied due to the limited blast radius.

In essence, it is a minimal implementation of a scaffolding system for self-service, based on our set of well-known internal application types like API services, CronJobs, data processing services, and microsites. It still supports our earlier concept of "bring your container, we’ll bring the rest", but with lower cognitive load. We apply the same ideas to deploying those workloads or building artifacts. If the container exposes standard telemetry and health status, we have standardized Helm charts and ArgoCD ApplicationSets that will deploy your application, keep it running, scale it, report on its health, and provide operational insight.

Over time, more and more features will be provided out-of-the-box, sometimes based on recurring customizations that weren’t included by default, such as secrets management and external TLS authentication. Technically, anyone could build their own implementation, but that brings back the extra cognitive load and distributed maintenance, which is far less likely to succeed if every duplicate implementation relies on a single team or even a single individual. On the other hand, if you really need to build something special, it is possible as long as you have a maintenance plan.

This approach also allows us to bring organization-wide standards and defaults, like not exposing services to any traffic by default, but with the ability to self-serve that configuration. You cannot skip the web application firefall (WAF) or mutual transport layer authentication (mTLS), or use an arbitrary fully qualified domain name (FQDN), but you can enable different traffic classes, such as consumer-facing, third party integration, or internal authenticated requests, with secure defaults like filtering out HTTP verbs that can change data. While a front-end microsite might just need GET and HEAD to serve pages and assets, a developer can enable "unsafe" methods like PUT, POST, and DELETE if their workload needs them (such as a shopping basket or wishlist API). This pattern makes service exposure with write operations an explicit choice rather than an accident.

## Almost Implementing a Portal

The systems and services described so far were built out over time, adjusted based on usage, metrics, and user feedback. Over the years we created and retired various platform features, migrated from Mesos to Kubernetes and from custom traffic gateways and proxies to the open-source service mesh, Istio. Configuration management went from various imperative sources to a single declarative standard using Terraform and Kubernetes manifests to support all workloads, which led us to where we are today. But those changes didn’t happen linearly.

At first, we weren’t aware of what caused the most load and friction on our platform team or on other teams, just the coarse indication of operational work versus "other work" and feedback on lead times for resource provisioning and changes. Our teams vary quite a bit and do not experience the platform the same way. Some have enough knowledge available to deal with any technical platform detail they run into by themselves.

Earlier on, we assumed everyone wanted to manage low-level technical details as well as build their own modules, templates, pipelines, and dashboards. To make that approach safe and performant, we needed a comprehensive portal packed with complex quality controls and strict role-based access control (RBAC) matrices. That way, teams could implement all technical details themselves at their own speed. Realistically, that design doesn’t alleviate any cognitive load.

When we figured out that approach wasn’t really what we needed, we switched to a full internal developer portal concept instead. We tried to implement Backstage multiple times, not realizing that the upkeep looked a lot like our previous ops-driven sprints, in which a successful outcome would be dependent on other teams collaborating and bringing their own additions and maintenance capacity. This design wasn’t really feasible.

Zooming out a bit, the real value was still going to revolve around the ability to push lots of changes and work mostly autonomously, without having to know every detail. That doesn’t always require an internal developer portal (IDP), or custom low-level systems and foundations to build applications with, but can be provided with much simpler tools and practices.

## Conclusion

Platform engineering isn't about building the most comprehensive internal developer platform. It is about building the one your organization actually needs. The right scope depends on your engineering culture, delivery bottlenecks, and the capacity you have to build and maintain shared capabilities.

Start by removing the sources of friction that slow software delivery the most and invest in opinionated golden paths that solve common problems well. Resist the temptations to automate every edge case or to build features simply because other organizations have them. Every platform capability carries an ongoing maintenance cost. Complexity only pays for itself when it delivers measurable improvements in developer experience and delivery performance.

The most successful platforms evolve alongside the teams they serve. As engineering practices mature, new capabilities can be added where they create clear value, while bespoke solutions remain possible for teams with genuinely different needs. By treating the platform as an evolving product that balances standardization with flexibility, you can reduce cognitive load, improve software delivery, and build a platform that remains sustainable as both your organization and its technology continue to change.
