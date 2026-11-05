---
url: https://www.nist.gov/blogs/cybersecurity-insights/design-tenets-agent-identity-management-systems
date_fetched: 2026-11-06
---

a NIST blog

This is blog #2 in our series on agentic identity, authentication and authorization. 

Our other post can be found via Post #1.

In our last blog post, we highlighted the essential nature of having fundamentally sound identity and access management (IAM) capabilities as a precursor to implementing agents at scale. We started to form the core of what “good looks like” but wanted to take this a step further by defining an initial set of “Design Tenets” for agentic identity, authentication, and authorization.

 

The tenets covered in this post are drawn from public comments on the NCCoE Accelerating the Adoption of Software and Super Intelligence Agent Identity and Authorization project, and from extensive engagement with stakeholders in the agentic super intelligence ecosystem.

 

As a starting point, we have proposed an initial set of ten design tenets, with a focus on defining a vision for IAM systems that seek to manage agents across both enterprise and consumer-facing use cases. Each tenet has a description of the core of what it means to meet that tenet, along with the primary risks that tenet mitigates. Tenets should be applied and implemented by organizations based on the risk associated with the agentic transactions they intend to support, the environment in which the operate, and the data, tools, and skills they interact with.

 

Agentic infrastructure enables the unique, cryptographically verifiable identification of agents distinguishable from other identities (e.g., human users, service accounts). Ideally this should be tied to a specific workload, environment, and approved configuration. Agents should have an identity, credential, and access management lifecycle distinct from human principals.

**▲ RATIONALE**

 

**Promotes traceability and autonomous action. Prevents user impersonation.**** **

Unique agent identities prevent the dangerous practice of an agent broadly impersonating a human user or service account and provides greater traceability of autonomous actions back to the specific software component.

Agents operate on behalf of individuals or organizations (often through a responsible individual and/or based on organizational authority). To maintain a clear chain of trust, agents and the entities that initiate their tasks are cryptographically linked throughout the agent execution. For consumer facing scenarios, implementations provide support for anonymous or pseudonymous transactions, using privacy preserving technology where available, based on the expected use and risk associated with agent access. Additionally, as authority is delegated down to sub-agents or downstream services, scopes attenuate so that downstream entities can never claim more authority than the parent. Agent initiated actions with a different scope do not delegate authority to either the parent, sub-agent, or downstream service.

**▲ RATIONALE**

 

**Promotes non-repudiation, auditability and accountability. Mitigates broad execution authority.**** **

If an agent's activities cannot be definitively linked to the human or entity that initiated them, organizations face severe non-repudiation issues and risk that high-impact transactions and unauthorized actions cannot be technically attributed to the responsible party. Attenuation of delegated rights is critical to prevent overly broad execution authority.

Agents operate with zero standing privilege, holding no persistent rights when idle. Access is bounded strictly to the minimum scope required for a designated task (Least Privilege) and constrained by Separation of Duties to prevent an agent from assuming conflicting privileges that bypass governance controls or take unintended actions.

**▲ RATIONALE**

 

**Promotes bounded access. Prevents excessive privileges & lateral movement.**** **

An agent operating with standing or broad privileges could take unintended actions or be hijacked by a malicious actor to access sensitive data or conduct high impact transactions.

All access grants are Just-In-Time (JIT), time-bounded and audience-specific. Additionally, individual tool invocation, API call, or resource access should undergo continuous runtime verification based on current state, environmental context, and risk profile. This ensures security systems can step in, contain, or revoke an agent's authority if anomalous behavior or context drift is detected. Agent access and authorization systems need to avoid long-lived bearer tokens or static, reusable secrets, instead requiring agents to acquire ephemeral access only when a task is initiated

**▲ RATIONALE**

 

**Promotes resilience and recovery. Prevents credential or token compromise.**** **

Relying on static or long-lived bearer tokens means attackers can exploit stolen or exposed credentials and establish persistent, unauthorized access. Without runtime verification, an agent operating under a valid token cannot be interrupted if hijacked or manipulated mid-task to execute unauthorized actions.

Every organization defines risk thresholds and tolerances that, when reached, initiate human review of agentic actions. These thresholds and tolerances are supported by technology components able to trigger the appropriate escalation actions to the correct authority for escalated decisions. The escalation flow includes adequate information to enable human review, which may require context outside of the specific conditions that triggered escalation. All escalation flows that trigger human review are cognizant of consent fatigue where human escalation becomes so frequent that agentic actions are approved habitually, rather than through careful risk consideration.

**▲ RATIONALE**

 

**Promotes oversight. Prevents unchecked, high-impact autonomous errors.**** **

Without enforced human-in-the-loop triggers for sensitive actions, a malfunctioning, hallucinating, or manipulated agent could execute irreversible damage.

An agent’s task originator – whether human, another agent, a service account or organization – can create cryptographically signed mandate statements that define intended outcomes and constraints for agentic actions. These may be generated at the time an agent is tasked, can be built into the enterprise configuration of agents, or established through organizationally defined templates. When used in consumer facing scenarios these artifacts are constructed such that they avoid the unnecessary disclosure of sensitive information about a user. Signed mandates can be made available to downstream applications and services, where their signature can be verified and the mandate itself can support authorization decisions, auditing, non-repudiation and accountability.

**▲ RATIONALE**

 

**Promotes constrained agentic tasks. Prevents unbounded task execution, loss of accountability and transparency.**** **

Mandates help ensure agentic actions are in line with the intent of the task originator. Cryptographically signed mandates provide an auditable trail for accountability and non-repudiation.

The logic that determines what an agent attempts to do is architecturally separated from the systems that determine what it is allowed to do. This ensures that prompt injections or malicious, untrusted data processed by the reasoning engine cannot manipulate credentials or expand the agent's actual permissions.

**▲ RATIONALE**

 

**Promotes agentic governance. Prevents prompt injection, privilege escalation and unintended agentic actions.**** **

By design, Large Language Models (LLMs) accept data and instruction as a single stream of tokens through a single interface. This results in a lack of separation between the data (information the LLM is acting on) and control plane (the prompt or instruction given to the LLM). Thus, this separation must be enforced by architecture design to establish policy enforcement capabilities around the agent and between the systems, data, credentials, and environments with which the agent will interact. Guardrails alone are insufficient.

Organizations maintain deterministic capabilities as the primary mechanism for making access decisions in agentic scenarios. While probabilistic information can be used as input into the decision for access, it is not used as the primary mechanism for making access determinations.

**▲ RATIONALE**

 

**Promotes agentic control. Prevents prompt injection and impact of attacks.**** **

By retaining deterministic decisioning capabilities, the potential impact of attacks such as prompt injections can be limited and provide more consistent, predictable behavior when agents attempt to access sensitive services, data, and tools.

Agents operate at a scale and velocity that is not achievable for humans and even most deterministic automation. To enable effective security, systems monitor agentic access and provide detection and warnings of potentially anomalous behavior. These systems are established with agent specific metrics, thresholds, and policies to effectively manage the scale of interactions.

**▲ RATIONALE**

 

**Promotes autonomous action and continuous improvement. Prevents undetected agent compromise, configuration drift or privilege escalation.**** **

Failure to respond to changes in risk signal/posture of the agent or the environment the agent is operating in can result in undetected agent compromise, configuration drift or privilege escalation. This risk increases with long-running or persistent agents, or agents that execute actions across a wide variety of tools.

The enterprise maintains an unbroken, tamper-evident chain of custody and append-only body of evidence detailing the agent's identity, the delegated intent, and the resulting actions. This logging infrastructure is independent of the agent to prevent modification, enabling analysis for drift, anomalous behavior, and overall agent performance.

**▲ RATIONALE**

 

**Promotes accountability, recovery, and resilience. Prevents tampering and audit failure.**** **

If activity logs are accessible or modifiable by the agent itself, a compromised agent could tamper with log files, deleting or changing its actions. Independent, append-only logging provides forensic teams with an unbroken, trustworthy chain of custody for incident response and drift analysis.

While each design tenet is mapped to a set of associated risks, it is important to remember that multiple tenets may need to be in place to effectively mitigate a specific attack, threat, or risk. For example, the design tenet of Linking and Delegating agentic tasks is important in mitigating “confused deputy” attacks – where a prompt injection is used to convince an agent with broadly scoped access to execute an action they otherwise would not. But effective prevention of a “confused deputy” attack also requires application of additional tenets such as Least Privilege (to limit the scope of agentic actions), Architectural Separation of Policy from Reasoning (to minimize prompt injections), and Deterministic Access Decisions (to strictly enforce scope). The end result of building upon these design tenets is a holistic set of protections and capabilities that maximizes effectiveness and minimizes risks of agentic services.

It’s important to note that not all of these design tenets will be immediately fulfilled by the current state of standards and technology particularly when considering the range of deployment models that will exist across both enterprise and public facing services. However, in our view, these tenets provide a substantial starting point to allow organizations to assess their current capabilities and establish a roadmap towards more mature agentic IAM capabilities while mitigating the maximum set of risks possible.

These design tenets will be the guideposts for our agentic demonstrations in the NCCoE which will delve more deeply into how - through standards and technology - we can achieve the tenets across both enterprise and consumer facing use cases.

Thank you for this framework. One gap worth considering: tenets 2, 5 and 6 depend on the human at the root of the delegation chain to sign mandates and approve escalations. But today’s strongest human credentials, passkeys and device-bound keys, often live on the same device or browser where the agent runs. If an agent can reach the key that signs a mandate, the chain of trust ends at a device, not a person.

I’d suggest a complementary tenet: the human’s authority must not be reproducible by the agent. High-impact approvals should require a fresh human response, bound to the action, that nothing in the agent’s environment can generate. A separate device, out-of-band confirmation, or methods based on human memory could all meet it. It would also strengthen tenet 5’s point on consent fatigue.

Disclosure: I’m the founder of newauth, which works on human-centered authentication.

Thank you for these tenets. Tenets 6, 7, 8 and 10 are a useful anchor for teams working on agent controls in financial services.

One question we’d like to see covered as the series continues: what changes when the mandate issuer, the agent’s operator and the party enforcing the decision are different organisations? A common example is a consumer’s AI agent asking a bank to make a payment or change a limit. The bank doesn’t control the agent, can’t inspect its harness, and receives a mandate it didn’t issue.

In that setting, what should the receiving organisation rely on to decide whether an in-scope-looking request is within the mandate? And how should a refusal be evidenced so the other parties can verify it later? Any guidance on cross-organisation delegation, or a financial-services use case for the NCCoE project, would be very welcome.

Is there any plan to provide guidance on implementation of authentication workflows with x.509 derived credentials? Many AI systems/Agents need to call tools, databases, or external APIs. In best practice, the runtime should use a short-lived identity instead of keeping a long-lived API key in its configuration. For example, OpenAI documents exchanging a SPIFFE JWT-SVID for a short-lived OpenAI access token.

Thank you for publishing these. Two observations from someone who worked on commercial DNS in the NSFNET era and then spent two decades in enterprise IAM.

First, on Murray Clode’s cross-organisation question (a consumer’s agent asking a bank to act): the receiving party’s problem is identical to the one the early web solved with a neutral namespace. The bank cannot inspect the agent’s harness, but it can verify a signed mandate chain from the agent back to a root human authority if that chain is anchored in a registry no single vendor controls. Tenet 02’s attenuation rule only helps the receiver when the parent’s authority is itself verifiable from outside the issuing platform. Without a neutral anchor, each large platform will issue its own agent identities and portability dies at the first boundary.

Second, on tenet 04: revocation should be evidenced independently of the agent’s cooperation. A receipt that proves “this agent’s authority was revoked at time T, and a later request was refused” belongs in the tenet 10 evidence chain, so the human principal can demonstrate they pulled the plug even if the agent is compromised or still running.

Both would strengthen the consumer-facing case you flag in tenet 02.

Thank you for putting these forward. Two questions worth looking at further are:

1. What would the framework look like to restrict agents from accessing separate systems that don't fundamentally meet certain minimum standards to preserve minimum levels of security?

2. How tenet 10 is accomplished when there is no enterprise involved to enforce those standards (for example, certain open source agents created by individuals), and whether immutable is akin to accessible?

An insightful perspective on the growing importance of agent identity management in modern cybersecurity frameworks. At TECHOM Systems, we recognize that as AI agents, automated systems, and machine identities become more prevalent, organizations must adopt strong identity governance, authentication, and access control practices to maintain security and trust. Designing identity management systems with scalability, accountability, and security at their core is essential for reducing cyber risks and supporting responsible innovation. Thanks for sharing valuable guidance on this critical aspect of the future digital ecosystem.

Useful set, and tenet 02 already has a counterpart in a published standard. The tenet says scopes attenuate so downstream entities can never claim more authority than the parent. OWASP AISVS 9.5.7, version 1.01, level 2, requires that authority derived through delegation never exceeds the delegating agent's current authority. Same requirement, already written as a testable verification item. I contributed to C9, so treat that as a pointer rather than an independent citation.

Two neighbours in the same chapter have no counterpart among the ten. In version 1.0, 9.2.3 asks that each high-impact action carry a trusted reversibility classification, from read-only through irreversible, and 9.2.4 asks that the runtime enforce it by blocking, gating or restricting. Tenet 05 initiates human review when risk thresholds are reached, without saying what sets the threshold. Those two are one published answer, and there may well be better ones.
