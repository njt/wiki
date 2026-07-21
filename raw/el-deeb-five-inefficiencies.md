---
url: https://dl.acm.org/doi/epdf/10.1145/3800646.3800650
title: "Stop Blaming QA and Start Diagnosing the Hidden Inefficiencies Behind Delivery Delays: Five Drivers that Slow Releases And Erode Developer Productivity"
author: Ahmed El-Deeb
date_fetched: 2026-07-21
date_published: 2026-04
---

ACM SIGSOFT Software Engineering Newsletter

Page 12

April 2026 Volume 51 Number 2

Stop Blaming QA and Start Diagnosing the Hidden
Inefficiencies Behind Delivery Delays: Five Drivers that Slow
Releases And Erode Developer Productivity
Ahmed El-Deeb
Amazon
Berlin, Germany

ahmed.eldeeb@qawithahmed.com
DOI: 10.1145/3800646.3800650
https://doi.org/10.1145/3800646.3800650

ABSTRACT
"The low hanging fruit" is not always the one worth cutting. Most of the
time, it's the only thing we aim for just because we are too lazy or
incapable of reaching out to what's deep and high. "What is the low
hanging fruit?" is sometimes then a direction of ease rather than impact.
When it comes to efficiency and speed of software delivery, I've always
witnessed initiatives to cut QA. At the end of the day, QA is a visible
stage in the software production process that has its volume and shape –
the low hanging fruit so to speak. It's not software creation activity, it
involves manual activity, and it has its own cycle of time other than
development time. That's why when people think of what to optimize to
achieve speed, they think of removing or shrinking QA. However, at the
higher trenches of the software development tree more drastic
inefficiencies that delay software delivery and kill its productivity. This
article examines those inefficiencies for the sake of directing our
attention to what we really need to solve to improve delivery speed. It
turns out that it's not QA that we should attack the most. If we really want
to be efficient, let's leave the "low hanging fruit" mentality and aim for
what's high and deep – the real drivers behind software delivery
inefficiency.
.Categories and Subject Descriptors
A.0.3 [GENERAL]: Tech Industry and Development Teams.

General Terms
Developer Testing,
management.

Software

Testing,

Organizational,

Team

Keywords

TOP 5 KEY INDUSTRY
INEFFICIENCIES

SIGNALS

OF

Instability-driven rework (unplanned fixes &
rollbacks)
Instability-driven work is the burden the team takes to respond to an
urgent bug fix that is blocking a release or a program, production incident
blocking customers, hotfixes deployment or rollbacks rather than doing
planned delivery. Rework is not just defects fixing. Instability produces
cycles of triage, time spent in investigation, incident reporting, managing
unplanned rollback or hotfix release. A recent high-signal research thread
reinforces this fact. DORA has expanded its core measurement model to
treat rework as first-class stability concern, explicitly connecting delivery
instability to wasted effort [1]. It defines Change Fail Rate as the ration
of deployments requiring immediate intervention and Deployment
Rework Rate as the ratio of unplanned deployments as a result of
production incident [1]. DORA's well-being breakdown shows
unplanned work and rework sits around 20% of time.
Cloudflare's June 2022 outage due to resilience-improvement release
demonstrates that such limited scope release can trigger large scale
rework. The outage impacted 19 data centers and 50% of total requests
[2]. Recovery from such incident required coordinated reverts with
incident report noting engineers "walked over each other's changes"
intermittently reintroducing the issue [2].

Quality ownership, Shift-left testing, Shift-left testing, Software Quality.

INTRODUCTION
The most common and recurring narrative in software delivery is that
"QA slows us down." Nevertheless, in reality, most organizations more
often lose time and efficiency due to invisible queues: builds waiting
time, waiting for decisions, unplanned work, instability, inefficient
meetings, rework; just to name a few. This is because inefficiency debates
tend to converge on what's visible: stages with timestamps and named
owners; such as, testing stage owned by QA taking this number of days.
This misdiagnosis the true drivers behind software delivery inefficiency
problems. In many other cases, revolving the discussion around QA in
order to shrink or eliminate it is aimed at solving organizational problem
by eliminating a few headcounts, but it isn't really solving software
engineers' problem. Let's examine what are the real drivers that bog
down software delivery and bother software engineers; not in defense of
QA, but for the sake of expanding our horizons so that we see what's
harming our delivery without yielding any value.

Instability-driven work can have the below measurable indicators:
-

Deployment Rework Rate.
Failed Deployment Recovery Time.
Incident Frequency.

Typical root causes behind stability-driven work are large blast radius
changes, insufficient pre-deploy validation, and tightly coupled
dependencies [3]. When team is incentivized or pushed to release big
batches and challenged for delivery speed, we are increasing the risk of
instability work as many things will be overlooked, including the ability
to properly validate the change. In companies where teams are forced to
only rely on automated coverage alone, such large batches with
dependencies get insufficiently analyzed for blast radius and properly
validated.

ACM SIGSOFT Software Engineering Newsletter

Page 13

Priority churn and requirement volatility
Repeated changes in priority and/or requirements or randomization
during delivery drives rework, context switching, coordination overhead,
and reduced throughput. The Google Cloud blog summarizes the 2024
DORA report argues that “constant pivot” mentality harms developer
well-being and progress; it hinders overall work progress even if other
good practices exist [4]. When the team is randomized or change course
frequently during delivery, they start to lose focus and motivation. This
has potential ripple effect on timelines which leads the team to cut corners
on the final delivery.
In an industry case study at ASELSAN, a requirements dataset
22,771requirements was analyzed and it turned out that more than half of
this dataset was modified at least once – that is, 9848 requirements in
total got changed at least once during delivery progress [5]. The same
study reports a model that identifies 63.2% of “highly volatile”
requirements covering 80% of total requirement changes [5].
A few measurable indicators for such churn can be represented by:
-

Change request per requirement
Backlog churn rate.
Scope Change Event.
Timeline Change Events.
Re-estimation Frequency.

Typical root causes include weak product strategy and stakeholder
alignment, late feedback, ambiguous requirements, and escalation-based
culture that reward responding to escalations and random requests over
finishing planned work.

Cross-team dependency overhead and service
coupling
High coupling exists when one team cannot deploy or test on-demand
without coordination with another one or more teams, through integrated
environment, upstream/downstream tech dependency, or external
approvals. This includes not only technical dependency but process
dependency a well. When coupling getting higher, considerable amount
of time is wasted in heavy coordination, alignment, and shared strategy.
The other side of the same coin, considerable amount of time is lost when
one component breaks for everyone. Coupling multiplies coordination,
blocks independent releases, and turns local changes into cross-team
programs.
That scale of dependency is explained well by Uber shift left blog post.
Uber describes gating “every code and configuration change” to core
backend systems spanning 1,000+ services, running “several thousand”
E2E tests with average pass rate 90%+ per attempt on every diff [6]. This
is massive. If one system spans 1,000+ services, independent delivery is
structurally hard. In that same manger, Uber is also mentioning that
maintaining shared staging environment became unreliable because one
team merge breaks other teams’ components.
Measurable indicators to watch for such inefficiency includes:
-

% Deployment requiring coordination.
Cross-team coordination hours.
Upstream-change-driven unplanned work (e.g. fixing
dependency breakage or feature requests for both components
to work together).

April 2026 Volume 51 Number 2

A top root cause of highly coupled systems and heavy coordination
overhead is a Conway mismatch: the architecture is mirroring the
organization’s communication patterns rather than the intended design.
When teams must coordinate frequently to deliver even small changes,
the software tends to evolve into tightly coupled components with brittle
integration points. The mismatch becomes most visible when the
architecture requires close collaboration (because components are highly
coupled), but the teams that own those components do not communicate
effectively. Another root cause is the absence of dependency
management tooling. If one component has dependency with others,
teams need tools that can help them design with such dependency in mind
and to be able to test that. Additionally, the ability to identify blast radius
of change whether it impacts or breaks dependency or not.

Technical debt and maintainability drag
Maintainability drag is the cumulative tax on change due to debt: harder
refactors, brittle dependencies, and cross-team friction. Teams spending
time on maintaining and improving code not only takes away time that
can be spent on writing new features, but also they are introducing risks
in areas that should have been relatively stable in the product. Refactoring
or improving legacy code takes away development time and takes away
testing time required to cover the blast radius of that kind of change.
A 2025 Meta study reports over 14% of changes are explicitly devoted to
“code improvement,” spanning grassroots refactors to blocked time and
major reengineering initiatives [7]. This is significant effort taken away
from doing feature development that delivers value to customers. DORA
frames maintainability/dependency management as a major
organizational pain point at scale.
Measurable indicators to look for include:

-

% changes labeled as improvement.
Dependency upgrade lead time.
Frequency of dependency breakages.
Regressions Due to Redesign.

As to root causes behind technical debt, the most cited reason is the
incentive for delivery over Code Quality. When organizations create
strong time/throughput pressure (often via delivery-only incentives),
teams tend to show higher short-term output but lower quality; and they
accumulate technical debt, which later drags quality and productivity.
Across empirical studies, time pressure is commonly associated with a
pattern of higher short-term output but lower quality [8]. Given that
technical debt is associated with substantial wasted effort (≈23% in one
longitudinal study), delivery-only incentives can plausibly push teams
toward choices that degrade maintainability and quality over time.

Review and approval queueing
Queueing is waiting time for reviewer/approver action (PR pickup,
review cycles, CAB approvals), producing long-tail delays and context
switching. This includes notable disputes among engineers on the validity
of some review comments that sometimes result in one or more reviewers
blocking the PR for sometime until the conflict is resolved.
Meta tracks “Time In Review” and observed early-2021 P50 review time
“a few hours” while P75 increased “by as much as a day,” correlating
with lower satisfaction. [15] Microsoft’s Bad Days telemetry validates
that when developers cite code reviews as a “bad day” factor, their PR
dwell time is 48.84% higher (22.49 hours vs 11.51) and total PR time
23.87% higher [9]. DORA warns that heavier approval processes can

ACM SIGSOFT Software Engineering Newsletter

Page 14

create vicious cycles by increasing lead times and batch size [10]. Stripe’s
Developer Coefficient report (survey across countries) quantifies the
drag:
-

17.3
hours/week
spent
on
maintenance
work
(debugging/refactoring/modifying) [12]
13.5 hours/week on technical debt [12]
3.8 hours/week on “bad code”
Average work week 41.1 hours [12]
“Bad code” estimated at ~$85B annual opportunity cost; global
GDP loss estimate shown in-report.

Measurable
-

indicators

of

such

queuing

behavior

include:

PR dwell time, time in review (P75/P90).
Acceptance-to-merge delay.
Approval lead time.
Reviewer load distribution.
Number of Disputes per PR.

Review queuing is a result of multiple factors, sometimes all acting at
once. Reviewer scarcity, especially for particular types of changes that
require specific expertise in the team. Uneven distribution of expertise
within the team can create centralized gatekeeping to senior team
members, which increases review load on them. Lack of accepted teamlevel norms, strategy, and best practices are also one of the reasons that
increase chances of disputes among engineers as to which approach is
better. In many cases, the reviewers can dispute the approach followed in
the PR because the reviewer believes it’s the wrong approach or school
of thought.

OTHER IMPACTFUL INDUSTRY SIGNALS OF
INEFFICIENCIES
The below didn’t make it to the Top 5 list because they are considered
downstream symptoms of the top 5. However, I will still briefly list them
out here because their impact is non-marginal and what they convey is
very important to watch for.
Based on consolidated industry research (Microsoft Dev Productivity
studies, Google internal engineering reports, McKinsey engineering
velocity research, Atlassian collaboration data, Stripe developer
productivity report, and multiple ICSE/CHI empirical papers), research
shows that the below are not primary inefficiencies, but the powerful
finding is that they are:
-

Signals of misalignment.
Signals of low trust.
Signals of unclear ownership.
Signals of structural complexity.

April 2026 Volume 51 Number 2

The pattern, especially in large orgs, is that meeting inflation scales with
size and cross-team dependency. The more team members involved and
the more architecture or process gets intertwined, the more friction arises.
Meetings in that sense become friction band-aids. Also, companies with
CYA (Cover Your Ass) culture, where self-protection becomes more
important than taking responsibility and collaborating to solve the actual
problem, rely on defensive meetings where teams justify or point fingers.

Operational Toil
Operational toil is manual, repetitive, automatable work that scales
linearly with service growth; interruptions include on-call workload. This
includes time spent managing internal campaigns, triaging incoming
issues, upgrading tools and libraries, maintaining DevOps, to name a few.
This.
Google reports that quarterly surveys show average toil around ~33%,
with outliers as high as 80%. This is a very strong indication of how
operational overhead is eating engineering capacity. DevOps Institute’s
Global SRE Pulse 2022 (460+ respondents) reports that organizations
track and manage toil and identifies major sources: 27% of respondents
cite process issues as their number-one source of toil and 19% cite
application release as the top toil source (with additional categories such
as production interruptions, skill gaps, and human error) [13][14].

Developer Productivity Loss
The developer productivity gap is the measurable divergence between (a)
how developers actually spend their time and (b) how they believe they
should spend it to create value sustainably; it is an output signal that
aggregates many upstream inefficiencies.
Microsoft survey of 484 developers mapped actual vs ideal time
allocations and found that, in the actual workweek, developers dedicate
the largest share to “Communication & Meetings” (~12%), followed by
“Coding” (~11%), “Debugging” (~9%), “Architecting/designing”
(~6%), and “Pull Requests/Code Reviews” (~5%). In the ideal week,
developers prefer a markedly higher share for coding (~20%) and
architecture/design (~15%), and less time on communication/meetings
and task-management work. Stripe developers rate their org productivity
at a mean 68.4% (100% = perfectly productive), and the report quantifies
the maintenance/debt share consuming a huge fraction of capacity [11].

WHERE THEORY BREAKS: GAPS BETWEEN
RESEARCH AND PRACTICE
Same as the visible stage of QA is tempting to make orgs believe that by
removing QA software delivery will be faster and more efficient, there
are a lot theories around the inefficiencies discussed above that are not
withstanding the test of operational practicality:
-

Contemporary DevOps and CI/CD theory assumes that
increasing deployment frequency reduces risk through smaller
batch sizes and faster feedback loops. The theoretical
assumption that “smaller changes = safer changes” does not
account for:
o System-level interaction complexity in distributed
architectures.
o Infrastructure fragility and environment drift.
o Organizational
response
patterns
(incident
swarming, rollback rituals, emergency approvals).
o Once failure rate crosses a threshold, the
organization enters a reactive mode characterized by
firefighting, context switching, and release
hesitancy. The system transitions from proactive
flow
to
defensive
coordination.

-

Agile theory treats adaptability to changing requirements as a
competitive advantage. Responsiveness is equated with
efficiency. Backlog reprioritization is framed as agility rather

Ineffective Meetings
Meetings are not evil and rarely the root cause of inefficiency by
themselves. However, engineers report meetings as top disruption to flow
state. Microsoft research (survey of 484 developers) reports actual time
allocation for communication and meetings to be ~12% of time [11].
What makes meetings nightmare of inefficiency and disruption to
engineers is because they are used as compensatory mechanisms for:
-

Poor decision clarity.
Unclear ownership.
Requirements change.
Weak written documentation or specs.
Leadership Indecision.
Status syncs replacing asynchronous clarity.

ACM SIGSOFT Software Engineering Newsletter

Page 15

than waste. However, empirical data on requirements volatility
shows frequent modifications per requirement, and industry
surveys link unstable priorities to productivity decline and
burnout. The break occurs because theory:
o Assumes low switching cost between priorities.
o Ignores cognitive reset cost.
o Underestimates coordination rework across
dependent teams.
o Agile responsiveness scales poorly when
coordination
complexity
is
high.
-

-

-

Microservices theory promotes autonomy, independent
deployability, and reduced coordination. The assumption is
that decoupling architecture leads to decoupled teams. In
practice:
o Logical service boundaries do not eliminate runtime
and data dependencies.
o Organizational boundaries rarely align cleanly with
architectural ones.
o Shared environments and integration testing create
hidden coupling.
o Platform teams become coordination hubs.
Technical debt theory frames shortcuts as conscious trade-offs
that can be repaid. It assumes rational decision-making and
manageable debt accumulation. Reality diverges in several
ways:
o Debt is often emergent, not deliberate.
o Incentive structures prioritize feature throughput
over maintainability.
o Debt compounds invisibly until productivity drops
sharply.
Code review theory assumes peer review improves quality with
minimal flow disruption. Lean theory assumes queues are
reducible through small batches. However,
o Review latency (waiting time) often dominates
actual review effort.
o Large orgs experience asymmetric review load
distribution.
o Approval
processes
introduce
multi-layer
gatekeeping.

OPPORTUNITIES
FOR
ENGINEERING RESEARCH

SOFTWARE

To address the gaps between theory and practice in front of real industry
signals discussed, research in software engineering looking into the
below topics can help the industry solve its pains:
-

Instability Propagation Models: Develop formal models
quantifying how defects propagate across microservices and
boundaries.
socio-technical

-

Rework Cost Modeling: Current metrics capture failure
occurrence but not secondary costs (interruptions, coordination
overhead,
burnout,
opportunity
loss).

-

Socio-Technical Incident Modeling: Move beyond rootcause analysis to model organizational reaction cost and

April 2026 Volume 51 Number 2
recovery

friction.

-

Cognitive Reset Cost Modeling: Empirical measurement of
cost when engineers switch priorities mid-cycle.

-

Churn-to-Rework Ratio Analysis: Quantify how much
priority churn translates into technical and coordination waste.

-

Requirement Stability Index (RSI): A standardized metric
capturing
volatility
rate
per
release
cycle.

-

Quantitative Coupling Metrics Across Socio-Technical
Boundaries: Combine code-level dependency graphs with
organizational
communication
graphs.

-

Coordination Cost Estimation Models: Translate cross-team
dependencies
into
measurable
delivery
delay.

-

Deployment Independence Index: Formalize how
independently deployable services truly are in large orgs.

-

Debt Accumulation Trajectory Modeling: Longitudinal
models predicting inflection points where debt reduces
throughput.

-

Maintainability-to-Lead-Time Correlation Studies: Largescale empirical validation linking structural complexity metrics
speed.
to
delivery

-

Incentive-Sensitive Debt Models: Study how performance
evaluation
systems
affect
debt
growth.

-

Cross-Team Debt Externality Analysis: Quantify how one
team’s
debt
impacts
others.

-

Reviewer Load Balancing Algorithms: Optimize assignment
based
on
expertise
and
availability.

-

Latency vs Quality Trade-off Studies: Empirically measure
diminishing returns of prolonged review cycles.

-

CONCLUSION
It turns out that QA stage is not the mere stage consuming time and
bogging software delivery. In fact, comparing the time spent in testing
the software before shipping it to customers to the time spent in all the
above mentioned inefficiencies, we can clearly see that (a) at least the
time spent on testing has a return and value to software delivery in the
sense of delivering a high quality product while the time spent in those
inefficiencies is time spent with no remarkable ROI and (b) the time
spend in QA is not the longest timeline. Organizations needs to prioritize
measuring for these inefficiencies that lowers the quality bar, churn
software engineers, and slow down delivery process. Once measured,
organizations should add mechanism, both technical and procedural to
limit the waste.

REFERENCES
[1] DevOps Research and Assessment (DORA). “DORA’s Software
Delivery Performance Metrics.” DORA, https://dora.dev/guides/dorametrics/. Accessed 2 Mar. 2026.
[2] Cloudflare. “Cloudflare Outage on June 21, 2022.” The Cloudflare
Blog, 21 June 2022, https://blog.cloudflare.com/cloudflare-outage-onjune-21-2022/. Accessed 2 Mar. 2026.

ACM SIGSOFT Software Engineering Newsletter

Page 16

[3] DevOps Research and Assessment (DORA). “Loosely Coupled
Teams.” DORA, https://dora.dev/capabilities/loosely-coupled-teams/.
Accessed 2 Mar. 2026.
[4] Harvey, Nathen, and Derek DeBellis. “Highlights from the 10th
DORA Report.” Google Cloud Blog, 23 Oct. 2024,
https://cloud.google.com/blog/products/devops-sre/announcing-the2024-dora-report. Accessed 2 Mar. 2026.
[5] Holat, Anıl, and Ayşe Tosun. “Predicting Requirements Volatility:
An Industry Case Study.” QuASoQ 2021: 9th International Workshop
on Quantitative Approaches to Software Quality, CEUR Workshop
Proceedings, vol. 3062, 2021, https://ceur-ws.org/Vol3062/Paper08_QuASoQ.pdf. Accessed 2 Mar. 2026.
[6] Liu, Quess, and Daniel Tsui. “Shifting E2E Testing Left at Uber.”
Uber Blog, 2024, https://www.uber.com/blog/shifting-e2e-testing-left/.
Accessed 2 Mar. 2026.
[7] “Code Improvement Practices at Meta.” arXiv, 2025,
https://arxiv.org/abs/2504.12517. Accessed 2 Mar. 2026.
[8] Kuutila, Miikka, Mika Mäntylä, Umar Farooq, and Maëlick Claes.
“Time Pressure in Software Engineering: A Systematic Review.”
Information and Software Technology, vol. 121, 2020, 106257,
https://doi.org/10.1016/j.infsof.2020.106257. Accessed 2 Mar. 2026.
[9] Obi, Ike, et al. “Identifying Factors Contributing to Bad Days for
Software Developers: A Mixed Methods Study.” arXiv, 2024,
https://arxiv.org/abs/2410.18379. Accessed 2 Mar. 2026.

April 2026 Volume 51 Number 2

[10] DevOps Research and Assessment (DORA). “Streamlining Change
Approval.” DORA, https://dora.dev/capabilities/streamlining-changeapproval/. Accessed 2 Mar. 2026.
[11] Noda, Abi, et al. “Time Warp: The Gap Between Developers’ Ideal
vs. Actual Workweeks in an AI-Driven Era.” Microsoft Research, 2024,
https://www.microsoft.com/en-us/research/wpcontent/uploads/2024/11/Time-Warp-Developer-ProductivityStudy.pdf. Accessed 2 Mar. 2026.
[12] Stripe. The Developer Coefficient. Stripe, 2018,
https://stripe.com/files/reports/the-developer-coefficient.pdf. Accessed
2 Mar. 2026.
[13] Beyer, Betsy, Chris Jones, Jennifer Petoff, and Niall Richard
Murphy, editors. “Eliminating Toil.” Site Reliability Engineering: How
Google Runs Production Systems, O’Reilly Media, 2016,
https://sre.google/sre-book/eliminating-toil/. Accessed 2 Mar. 2026.
[14] Oehrlich, Eveline. Global SRE Pulse 2022: The State of SRE
Adoption, Deployment and Automation. DevOps Institute, 2022,
https://insights.devopsinstitute.com/hubfs/Global%20SRE%20Pulse%2
02022.pdf. Accessed 2 Mar. 2026.
[15] Riggs, Patrick. “Move Faster, Wait Less: Improving Code Review
Time at Meta.” Engineering at Meta, 16 Nov. 2022,
https://engineering.fb.com/2022/11/16/culture/meta-code-review-timeimproving/. Accessed 2 Mar. 2026.

