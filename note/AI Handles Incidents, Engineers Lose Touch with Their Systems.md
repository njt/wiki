# AI Handles Incidents, Engineers Lose Touch with Their Systems

Sylvain Kalache's warning that AI-assisted incident response ("AI SREs") is automating away the very thing that makes human responders competent: the routine incidents that build intuition for how a system fails. The better the tools get, the less practice engineers get — and the hardest incidents are exactly the ones automation hands back to humans.

---

## Key Quotes

> "The better these tools become at resolving routine incidents, the less practice human responders will get. And when an ambiguous, high-severity incident comes in that automation cannot solve, responding engineers will be in trouble."

The thesis in one sentence. The routine incident is not a nuisance to be automated away — it is the training set that prepares a responder for the novel one.

> "In the years to come, I predict that the average MTTR for most incidents will go down – thanks to AI-assisted incident response – but that the resolution time will shoot up for complex incidents because incident responders lost touch with their system and are struggling to investigate."

The sharpest falsifiable claim in the piece: a bifurcation in MTTR rather than a uniform improvement. The metric that improves globally is exactly the one that hides the collapse locally.

> "You might pick up a few things from watching Serena Williams play, but you only learn tennis by getting on the court, and incident response is no different."

Kalache's counter to the "just ask the agent to explain itself" fix. Explanation and observation are not practice — a point he argues from his own decade-plus running a progressive, project-based coding school where students learned troubleshooting by being handed broken infrastructure.

> "As LLMs do more of our work, engineering teams risk accumulating comprehension debt: a growing gap between how their systems work and how well responders understand them."

The coinage that ties this essay to the wider wiki conversation. Note it is *comprehension* debt, not *technical* debt — the debt lives in the people, not the code.

> "That's the irony of automation, the more successful it becomes, the less prepared humans may be for the moment it fails."

Kalache's restatement of Bainbridge, applied to on-call. It inverts the usual success story: every automated win makes the eventual human takeover harder.

## Key Themes

- **#concept Comprehension debt in operations** — the gap between system behavior and responder understanding, accrued one auto-resolved incident at a time. The operational cousin of [[Agents and Acquiring Debt]]'s comprehension debt and [[Cognitive Debt]]'s loss of understanding.
- **#concept The Irony of Automation** — Lisanne Bainbridge's 1983 finding that automation removes routine practice while preserving responsibility for abnormal cases, so operators need *more* training, not less.
- **#pattern Incident simulation as on-call readiness** — the aviation fix (FAA recurrent training, simulator rehearsals for rare failures) ported to software, via Rootly + Uptime Labs. Practice as a scheduled part of the job rather than an accident of traffic.
- **#pattern AI as trainer** — asking an agent to explain its diagnosis steps as a form of coaching, with the explicit caveat that watching is not doing.
- **#person Sylvain Kalache** — AI Labs lead and DevRel at Rootly, former LinkedIn SRE, co-founder of Holberton School. A rare voice arguing *for* more human practice in the automation era.

## Critical Analysis

**The right worry, argued from the right authority.** Kalache is one of the few people who can make this case without it reading as Luddism: he *built* a self-healing incident system at LinkedIn in 2012, he runs DevRel at an incident-management company, and he spent a decade building a hands-on coding school. When he says automation is eating the training ground, it is a practitioner's diagnosis, not a skeptic's.

**The aviation parallel does real work — up to a point.** The TransAsia 235 crash (117 seconds from first warning to stall, because the crew misidentified which engine had failed) is a genuinely chilling demonstration that rare events punish exactly the skill that routine events build. But aviation's answer — mandated, six-month recurrent simulator training — rests on a regulator with the power to ground an airline. Software has no FAA. Kalache's prescription therefore lives or dies on *voluntary* adoption by teams that are, in the same breath, being pressured to ship faster with fewer humans. He names the practice; he does not name the incentive that would make anyone actually buy it.

**The metric claim is the most valuable and least examined part.** "Average MTTR down, tail resolution time up" is testable — and, if true, it reframes every AI-SRE success story as a deferred failure. But Kalache offers it as a prediction with no proposed measurement. A team that wanted to check would need to track resolution time *conditioned on whether automation was able to handle the incident*, which is precisely the telemetry nobody is currently collecting. [[Principal Drift]] names the observable half of this same fear from the other side — incidents surging as comprehension drains — and it at least supplies failure signals (incident reviews taking >30 min, nobody able to talk through the data flow).

**"Comprehension debt" is the bridge to the rest of the wiki — and it's under-built here.** The term is doing the same work bl00cyb's [[Agents and Acquiring Debt]] does for code: agents hold *what* happened, not *why*, and the *why* is what a human needs in the middle of a novel SEV0. Kalache gestures at the fix (simulation, AI-as-trainer) but doesn't connect it to the ADR-style paydown mechanisms the debt conversation has already worked out. The interesting open question is whether a *simulated* incident is a form of decision-record — a rehearsed answer to "what would we do if X" — and whether the artifacts it produces could pay comprehension debt down the way ADRs do.

**Where it lands.** This is the missing operational-specifications page that [[The Future of Software Engineering is SRE]] asked for and [[Software Engineering Craft]] listed under "What's Missing." The claim that operations is the scarce skill now has a concrete downstream corollary: the scarce skill is not just *running* incidents, but *rehearsing* them. That is a small shift in phrasing and a large shift in what on-call readiness means.

---

*Sources: [[raw/ai-handles-incidents-engineers-lose-touch-with-their-systems]], [[summary/ai-handles-incidents-engineers-lose-touch-with-their-systems]]*
*Last updated: 2026-09-08*
