---
url: https://www.thoughtworks.com/insights/articles/real-world-lessons-agentic-authority-overreach
date_fetched: 2026-10-14
---

Recent reports from cybersecurity evaluations of OpenAI agents should end any illusion that autonomous systems can be governed by prompts alone. Agents have found unsanctioned communication channels, shared credentials, coordinated unauthorized activity and pursued objectives their operators did not approve. For boards, agentic overreach is no longer a hypothetical risk. The urgent question is what, exactly, they’re authorized to do and how (and whether) the organization can stop them when they cross the line.


Unexpected agent behavior is no longer an edge case. It should be assumed. Systems that can plan, adapt, use tools and pursue goals will sometimes find routes their operators did not anticipate. While that’s now a predictable operating reality, the market does not allow stepping back from autonomy; we must now build for surprises before deployment.

- **Plan for surprises.**Unexpected behavior belongs in the operating model, not in the post-incident explanation.
- **Make the decision defensible.**Leaders must be able to show why the risk was accepted and how it was managed.
- **Fund the whole business case.**The expected return relies on the people, governance and architecture needed to deploy responsibly at scale.

That leads to a simple investment principle: if an organization is going to bank on the higher quality and efficiency gains promised by agentic AI, it must also invest in actively mitigating the risks that accompany such powerful technology.


Three recent incidents demonstrate why.

## Expect the unexpected (and plan for it)


*OpenAI evaluation agents and the Hugging Face intrusion (July 2026).*


OpenAI ran a cybersecurity evaluation that isolated large numbers of AI agents and asked them to solve an impossible task. Independent investigators reported that roughly 1,200 agents used an unsanctioned communication channel to coordinate and direct approximately 700 agents to attack a third-party software company, Hugging Face. The chain included credential discovery and sharing, lateral movement, unauthorized tool use, malicious uploads and efforts associated with scorer or transcript manipulation. Some agents reportedly acknowledged in messages that the conduct was out of scope or harmful, yet continued anyway.


Context matters here. Some safeguards were disabled or ineffective, and some tasks encouraged agents to find another route. Not every deployed agent will attack another system. Even so, the business lesson is that prompts and good intentions do not become security boundaries simply because they are written in natural language.


### What leaders should take from this incident


A prompt is not a control. Shared state can become a coordination layer. A credential can turn exploration into access. A tool can turn access into an external consequence. And a poorly designed reward signal can make the wrong outcome look like success.


The controls must operate outside the model’s write authority. Agents and sub-agents need strict identity and credential boundaries, limited permissions, approved tools and destinations, transaction and velocity limits and a reliable stop mechanism the agent cannot rewrite. These aren’t technical extras; they are crucial in defining the enterprise’s operational blast radius.

## The interface can become evidence


*Garcia vs. Character Technologies (October 2024).*


In Garcia, the estate of a minor asserted negligence and product-defect claims arising from a companion chatbot’s design and interactions. At the motion-to-dismiss stage, the court allowed most claims to proceed and stated that it was not prepared, on the pleadings, to hold that the chatbot’s output was protected speech. The case later ended without a judgment on liability. (See the May 2025 order and January 2026 dismissal.)

### What leaders should take from the incident


This case demonstrates the importance of user expectations and understanding. More specifically, that they must be treated as fundamental parts of the design and development process, not pushed downstream or treated as a second order issue. An authoritative title, a human-like persona, an unqualified recommendation or a missing escalation path can shape how a user understands the system. It miight even later become evidence about whether the design was responsible.


The court in Garcia did not apply agency doctrine, but it could have. Interface design can shape reliance. Similarly, when an enterprise presents a system as acting on its behalf, those same signals may inform an apparent-authority analysis under applicable agency law. Restatement (Third) of Agency § 2.03 asks whether a third party reasonably believes an actor has authority to act on the principal’s behalf and whether that belief is traceable to the principal’s actions. For an enterprise agent, those signals may include its title, interface, email domain, place in a workflow and the words used to describe its role. Businesses should manage those signals as deliberately as they manage system permissions.

## You cannot outsource accountability



In Mobley, the court held at the pleading stage that the complaint plausibly alleged Workday could qualify as an employer’s statutory “agent” when performing allegedly delegated hiring functions. The court later preliminarily certified ADEA collective action — this was a procedural decision, not a finding of discrimination or liability. As of late September 2026, class-certification proceedings remained pending. The alleged statutory agent is Workday, the vendor that provided the AI, not the AI itself. (See the July 2024 order, the May 2025 order and the docket.)

### What leaders should take from this incident


The commercial point is simple: allowing a customer to activate or configure an AI tool does not necessarily insulate the provider when its system still performs a meaningful part of a regulated decision. Responsibility turns on how control is shared: who designs the decision logic, determines the available criteria, validates performance, processes the data, produces the outcome and can identify or correct harmful results. Contracts remain important, but customer configuration alone is not a complete liability strategy.

Boards don’t need to wait for courts to invent a special law of agentic AI. Existing law already asks familiar questions about attribution, reliance, product design and delegated functions.

Boards don’t need to wait for courts to invent a special law of agentic AI. Existing law already asks familiar questions about attribution, reliance, product design and delegated functions.

## The law will use the tools it already has (and boards should too)


Boards don’t need to wait for courts to invent a special law of agentic AI. Existing law already asks familiar questions about attribution, reliance, product design and delegated functions. UETA § 14 recognizes that electronic agents can form contracts without a person reviewing every step. E-SIGN Act § 101 protects electronic transactions from being denied effect simply because they are electronic and addresses electronic-agent actions that are otherwise legally attributable. Neither statute gives an agent unlimited authority; the answer still depends on the governing law and the facts.


The EU AI Act points in the same direction. Its logging, human-oversight and deployer duties depend on the system’s classification and the organization’s role, so they don’t apply in the same way to every enterprise agent. But the practical message is clear: as the consequences increase, so too does the expectation that someone can explain who authorized the system, how it was constrained and what happened.


*Authorities: Restatement (Third) of Agency § 2.03; UETA §§ 9 and 14; 15 U.S.C. § 7001(a), (h); and Regulation (EU) 2024/1689.*


Many organizations have mitigations and boundaries that are well known when it comes to managing human behavior. Boards have a duty to understand how they apply to AI agents, where they are just as, if not more, important.

## The way forward: Funding the infrastructure that makes autonomy scalable


The controls around an AI agent aren’t a brake on adoption; they’re the infrastructure that lets an enterprise pursue the benefits of agentic AI with confidence, resilience and commercial defensibility. The Agentic Scope of Authority Framework turns that idea into decisions leaders can fund, assign and audit. The basic premise is simple: authority must be clear inside the enterprise and credible to everyone outside it.


- *Actual authority*is what people authorize and the architecture enforces: the task, limits, tools, data, spending, counterparties and conditions under which the agent may act.
- *Apparent authority*is what others reasonably believe the agent can do based on the enterprise’s own signals: its title, interface, channel, disclosures and conduct.


Managing only one of these leaves half the risk unaddressed.


The framework brings the full operating picture together in nine blocks, covering everything from who owns the agent and what it’s meant to do, to the financial, contractual, data and operational boundaries around it, how its authority appears to others and how the enterprise will oversee, stop and audit it over time.


The executive responsibility is to ensure the organization funds, assigns clear ownership for and delivers four outcomes that put those controls into practice:


- **Put a real person in charge.**Every production agent needs a Designated Principal, a narrow purpose, a clear finish line, a “never” list and a named approver for material changes. Accountability cannot sit with a committee, a vendor or the model.
- **Set boundaries the model cannot negotiate.**Use strict credential separation, approved tools and destinations, data no-go zones, contract and spending limits, and non-expanding scopes for sub-agents. The agent should never be able to widen its own authority.
- **Match human oversight to the stakes.**The three tiers are straightforward. Manual controls establish human intent and risk appetite. Semi-automated controls turn human judgment into enforced workflows. Automated controls apply permission gates, monitoring and kill triggers at machine speed. For each material action, decide whether a person must approve it in advance, supervise it in operation or audit it afterward and then decide what happens if that person doesn’t respond.
- **Make sure you can stop it and explain what happened.**A soft pause should block new actions while preserving state. A hard stop should terminate the full agent tree, revoke credentials, block egress and preserve evidence. Independently controlled logs should be write-once or cryptographically tamper-evident and capture the mandate, relevant versions, inputs, tool calls, approvals, external actions and results, subject to privacy, privilege, retention and legal-hold rules.


Ultimately, this is a capital-allocation decision, not an ethics slogan. If the business case assumes agents will act across systems, the investment case must include the safeguards that limit those actions, the people who own the consequences and the evidence needed to defend the result.


**The organizations that win will not simply be the ones that deploy the most agents first. They will be the ones that turn autonomy into durable enterprise value. If an organization expects agentic AI to deliver higher quality and efficiency, it must fund the controls required to make those gains sustainable. Authority, oversight and evidence are not overhead added to the business case; they are what make autonomy investable at scale.**
