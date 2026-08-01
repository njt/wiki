---
url: https://spawn-queue.acm.org/doi/10.1145/3819083
title: "Beyond Zero: Enterprise security for the AI era"
author: Joseph Valente, Michal Zalewski
date_fetched: 2026-08-01
date_published: 2026-07-20
---

Google's vision for enterprise security in the age of AI agents. Joseph Valente and
Michal Zalewski of Alphabet Security argue that the application-centric zero-trust
model (BeyondCorp) is breaking under the strain of autonomous AI agents, exponential
data velocity, and machine-speed threats. They propose **Beyond Zero**, a new paradigm
that shrinks the trust boundary from the application level down to individual actions
on specific resources.

Key claims:

- **Access velocity has outgrown human-speed authorization.** AI agents access data at
  ~10× the rate of humans, reasoning across unstructured datasets. Legacy infrastructure
  can't authorize tens of millions of concurrent machine-driven actions per second.
- **The trust boundary must move.** Instead of granting access to an application or
  tool, Beyond Zero evaluates every action on every resource in realtime, blending
  static baseline policies (the "floor") with a dynamic AI reasoning engine (the
  "ceiling").
- **Four-component architecture.** Autonomous governance precomputes an enterprise
  world model (who, what, how); event intake streams signals from servers, clients,
  and agents; a hierarchical reasoning engine makes allow/deny/challenge decisions at
  access time or via slower deeper inference; challenge infrastructure issues granular
  interruptions—justifications, security-key touches, biometric checks, approvals—or
  durable containments that revoke access.
- **Challenges replace blunt denials.** When risk is ambiguous, the system asks for
  context rather than blocking outright, reducing disruption for legitimate users while
  stopping attackers who can't satisfy tailored hard challenges.
- **Attackers have weaponized AI too.** LLMs rewrite malicious code on demand; agents
  exploit ambient authority (inheriting overprovisioned human permissions). Beyond Zero
  counters with intent-based defense and realtime investigation rather than
  after-the-fact SecOps review.

The article ends with a call for industry standards: open architectures for agent
introspection, standardized agentic-identity and request-annotation frameworks, and
pluggable policy-evaluation points in SaaS products. It frames the transition as a
strategic necessity—security as an immune system that continuously adapts, not a static
perimeter.
