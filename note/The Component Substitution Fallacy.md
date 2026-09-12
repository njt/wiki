# The Component Substitution Fallacy

Lorin Hochstein uses the autoscaling misconfiguration in GitHub's recent outage to explain David Woods's "component substitution fallacy": the belief that reliability improves by finding and fixing defective components. The deeper claim is that component defects are never what actually takes a system down — every live system is full of latent defects and yet stays up, so the real failure lives in the *interactions* between components, which post-incident reviews keep ignoring.

---

## Key Quotes

> "the component substitution fallacy – the idea that the way to improve reliability is to focus efforts on identifying and fixing the defective components."

This is the named concept, borrowed from resilience-engineering researcher David Woods. Hochstein positions it as the frame that matters more than the specific autoscaling bug: identifying the defect is necessary but is not the same as understanding the incident.

> "your system is, at this very moment, filled with latent component defects [and] despite the presence of all of these defects, your system is not constantly failing over. This means that *component defects aren't enough to take down your system*, or your system would be down right now."

The sharpest argument in the essay, and a genuine reframe. It turns the postmortem's "what broke?" question into a slightly embarrassing one: if defects were sufficient for failure, you'd never be up long enough to have an outage to review. The interesting question is therefore not which part is defective, but which *combination* of parts, defects, and load conspired this time.

> "Don't just look at the individual components: treat the interactions as first-class."

The positive prescription. In GitHub's case the interactions were: changing traffic patterns (including scrapers), the autoscaling policy, Istio sidecar saturation, retry logic, HAProxy node saturation, and authentication traffic. No one of these is "the root cause"; their collision is.

> "every autoscaling policy is effectively bespoke. This means that a team that owns a service is not only responsible for the business logic, but also for an operational control system with custom parameters, that can really only be checked via load testing."

The under-discussed operational cost. Autoscaling isn't a setting, it's a small control system each team builds and owns, usually without expertise, and almost never load-tested. This is the same structural insight as [[Engineering for Bounded Cognition]]: we hand constrained operators a bespoke instrument and then act surprised when it's mis-tuned.

> "a service can become saturated even if CPU is low."

The concrete autoscaling trap, illustrated by Slack's 2021 incident: thread-per-request with a thread pool, downstream latency rises, all threads block, CPU stays low while the service is effectively down. CPU is a proxy that lies for I/O-bound services.

## Key Themes

- #concept **Component substitution fallacy** — David Woods's term for the error of treating reliability as "find and fix the broken part"; the interaction, not the component, is what fails
- #concept **Latent defects** — systems are always full of defects and still stay up, so defects can't be the sufficient cause of failure
- #pattern **Treat interactions as first-class** — post-incident analysis that maps the *combination* of factors (traffic, policy, saturation, retries) rather than hunting a root cause
- #tool **Autoscaling as bespoke control system** — a per-service operational controller with custom parameters, owned by non-experts, verifiable only through load testing
- #person **David Woods** — resilience-engineering researcher; the fallacy is his framing
- #person **Lorin Hochstein** — author of surfingcomplexity.blog, incident-analysis and systems-thinker

## Critical Analysis

**The strongest move is turning the outage into evidence for a general claim.** A lesser postmortem blogger would stop at "the policy didn't watch the sidecar, here's how to fix the policy." Hochstein does give that fix, then spends the rest of the essay arguing it's the *less* important takeaway. The latent-defects argument — your system is full of bugs and not falling over, so a bug can't be what killed it — is a genuinely disorienting reframe that most teams have never been forced to sit with. It's the resilience-engineering version of "correlation is not causation," and it lands.

**The tension between the two halves is real and unresolved.** The first half is a concrete, actionable engineering lesson (scale on the right metrics; watch your sidecars; test your policies). The second half is almost a warning *against* the first half's instinct — fix the component, sure, but don't think that closes the incident. Hochstein doesn't quite reconcile these: how much should you invest in the autoscaling fix versus in understanding the interaction? His honest answer is "I'd love to know more about the history and the traffic," which is satisfyingly humble but leaves the reader with a philosophy rather than a procedure.

**The strongest unstated target is root-cause culture.** The essay is a quiet attack on the "five whys" reflex, the postmortem that produces one defective component and one remediation. If interactions are first-class, then the deliverable of an incident review is not a broken part and its fix — it's a *model* of how the system's parts, their defects, and the traffic interacted this time. That model is what lets you ask better questions next time, and it's exactly what public writeups can't give you (no history, no traffic detail) and internal ones can.

**The "bespoke control system" point deserves more weight than Hochstein gives it.** Autoscaling-as-every-team's-private-control-system is a strong, specific, and underappreciated claim — it names a whole class of "operational logic that isn't the product" that accumulates in every service. It connects directly to [[Queues Don't Fix Overload]]'s warning that infrastructure is reached for *before* the system's constraints are understood, and to the [[Engineering for Bounded Cognition]] thesis that we keep handing small minds systems they can't hold. The essay gestures at this ("are you doing load testing on all your services?") but doesn't develop the remedy beyond "ask questions."

**The public-vs-internal distinction is the practical payload.** Hochstein's closing — public writeups answer *what*, internal ones can answer *why the traffic grew, whether the policy predated the sidecars, who owns it* — is the one thing a reader can act on Monday. It's the same impulse as [[Bad Data in Production — Response Playbook]]'s blameless review: the goal is information extraction, and the questions you ask determine the signal you get back.

## See Also

- [[Queues Don't Fix Overload]] — the overload sibling: Hebert says identify the red arrow, then back-pressure or load-shed; Hochstein says the red arrow can be a saturated sidecar your autoscaling policy can't even see
- [[Engineering for Bounded Cognition]] — the bespoke-control-system problem restated as human-factors design: a service owner who isn't an autoscaling expert is the "most constrained user" the system should have been designed for
- [[Bad Data in Production — Response Playbook]] — the incident-review complement: Dave's blameless, information-extracting review is what you'd do once you stop hunting the single defective component
- [[Software Engineering Craft]] — the reliability/SRE hub this analysis feeds into
- [[Why Are Databases So Hard]] — a database reliability engineer's walk through the same correctness/availability terrain, from the inside of one component's limits

---
*Sources: [[raw/github-autoscaling-and-the-component-substitution-fallacy]], [[summary/github-autoscaling-and-the-component-substitution-fallacy]]*
*Last updated: 2026-09-04*
