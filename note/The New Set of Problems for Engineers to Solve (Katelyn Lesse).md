# The New Set of Problems for Engineers to Solve (Katelyn Lesse)

Katelyn Lesse's essay argues that the identity crisis among software engineers has a resolution: serious multiagent systems — thousands of persistent agents working on research, discovery, and hardware design — create a fresh, wide-open engineering problem space (infrastructure, agent identity, security, monitoring), and AI's promise is contingent on engineers solving it.

---

## What it argues

Agents evolved from human-in-the-loop chats to cloud-running long-task workers to early multiagent systems (orchestrators spawning subagents, agents as graders, advisors, dreamers). The next step — persistent fleets of thousands of agents — is where the engineering goes existential, in the good way. The essay maps four problem areas:

- **Infrastructure.** Kubernetes assumed stateless replicated apps or stable stateful pods. Agents are "isolated, stateful, single-instance workloads" that idle for arbitrary periods and spend their lives waiting — on tools, inference, other agents, humans. New filesystems, sandboxing products, and runtimes like Google's Agent Substrate are early answers.
- **Identity and access.** Authentication inherits hardened microservices patterns (workload identity, short-lived certs). Authorization is wide open: most agents act on-behalf-of (OBO) using their human's identity, which already obscures who acted, and raises the consent question — how does a human meaningfully approve what their agent does over weeks? Agents with their own identity, and agent-to-agent trust, are less solved still.
- **Security and isolation.** Vulnerabilities are already surfacing faster than they get patched in the critical core software everything depends on. Thousands of agents running untrusted code on top of that is an amplifier. Worse, a compromised agent can pass injected instructions to peers, where they persist in memory for future agents to trust — prompt injection as a lateral-movement problem, with blast radius as the governing question.
- **Monitoring and halting.** Cross-agent OTel tracing that agents cannot modify, kill switches that stop the whole system and cut its credentials — and the genuinely hard part: well-intentioned persistence looks like maliciousness. Repeated attempts to reach a non-allowlisted domain might be a bug, an attack, or an agent being diligent.

The closing is a recruiting pitch: go deep in these domains, form opinions, because "you and your skills can be the difference in whether... multiagent systems can solve the world's hardest problems."

## Key quotes

> "AI just isn't going to be able to create the incredible outcomes it's capable of creating unless software engineers solve problems to make it possible. This is existential, which is honestly a lot of fun."

The thesis in one line, and the emotional frame — she deliberately reframes "existential" from threat to stake, aimed directly at the depressed half of the identity-crisis split.

> "They're essentially isolated, stateful, single-instance workloads. They can go idle for arbitrary amounts of time between bits of work, and they spend a lot of time waiting on tools, model inference, other agents, or human input."

The sharpest technical claim in the piece: a crisp statement of why the Kubernetes-era compute model fits agents badly. Single-instance stateful workloads are precisely the thing the last fifteen years of orchestration tooling was built to eliminate.

> "A compromised agent could pass injected instructions to other agents it's working with, and those instructions could end up in memory for future agents to accidentally trust."

This reframes prompt injection from a per-session nuisance into a persistent, lateral-spreading threat inside multiagent systems — injected content becoming trusted memory is a qualitatively worse failure mode than a single poisoned context window.

> "Well-intentioned persistence to solve a problem can look a lot like maliciousness."

The most honest sentence in the piece. Everything else is a tractable engineering problem; this one is an epistemology problem, and she doesn't pretend otherwise.

## Themes

#concept #pattern #tool

- **The engineer's job migrates around the model** — the durable work is the surrounding systems: runtimes, identity, isolation, observability.
- **Inherited tooling mismatches** — Kubernetes, static credentials, and human-consent models all assumed workloads that agents aren't.
- **Injection as lateral movement** — compromise propagates through shared memory and inter-agent messaging, not just through one context.
- **Halting as the open problem** — distinguishing diligence from malice is unsolved and possibly unsolvable with heuristics alone.

## Opinionated take

This is the most useful genre of AI essay: not another prediction, but a work order. The four-area framing (infrastructure, identity, security, monitoring) is a genuine research-and-product roadmap, and the K8s-mismatch observation deserves to be the standard way people describe why agent infrastructure is hard. That said, the optimism does some work the essay doesn't pay for: " cured diseases rather than bad outcomes" is doing rhetorical heavy lifting, and the consent problem gets one paragraph when it may be the least tractable of the four — "meaningful approval" of weeks-long autonomous work is not an engineering problem with a known shape, it's a socio-technical one. The monitoring section also quietly admits its hardest subproblem (malice vs. persistence) has no proposed solution beyond naming it. Still, as a map of where the hiring and the tooling dollars go next, it's accurate and unusually concrete for a tweet-length essay.

## Relations

- [[Agent Substrate]] — Lesse name-checks Google's Agent Substrate as an example of the new-agent-runtime work; this essay supplies the "why" (K8s mismatch) that the Substrate write-up supplies the "what" for.
- [[Agent Identity]] — this source strengthens the identity page's claims from the OBO direction: borrowed human credentials obscure attribution and make consent unsolvable at scale, exactly the pressure that page documents.
- [[A Deep Dive on Agent Sandboxes]] — Lesse's "strongly isolated sandboxed execution environments" as the answer to multiagent blast radius is the same conclusion that page reaches from the sandbox-design side; this essay adds the injection-as-lateral-movement argument for why it's non-negotiable.
- [[Nobody Is Ready for the AI-Agent Flood]] — complicates this essay's optimism: where Lesse sees a wide-open, exciting problem space, that source catalogs the same gaps as active, unpatched exposure.

---
*Sources: [[raw/2114542280535318557]], [[summary/2114542280535318557]]*
*Last updated: 2026-10-26*
