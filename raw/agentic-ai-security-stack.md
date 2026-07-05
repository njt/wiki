---
url: https://www.nextkicklabs.com/p/agentic-ai-security-stack-book-release
date_fetched: 2026-07-05
backfilled: true
---

# The Agentic AI Security Stack

### Deploy secure agentic AI systems. This free 200+ page reference provides a unified threat model, traces kill chains, and maps every control to OWASP, MITRE ATLAS, & CSA MAESTRO. Share without legal re

**Get the book:** Read online and/or download PDF - no email required, no paywall.

**License:** Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 (CC BY-NC-ND 4.0). Free to share with colleagues, cite in documentation, and redistribute with attribution. No commercial use or derivative works.

**Copy this to share:**

“The Agentic AI Security Stack is a free 200-page practitioner reference covering 12 security layers for agentic AI deployments: threat modeling, credential architecture, isolation, egress control, human-in-the-loop security gates, memory integrity, orchestration trust, agent-native identity, MCP protocol hardening, supply chain verification, audit logging, and regulatory compliance. Every chapter maps to OWASP, MITRE ATLAS, and CSA MAESTRO. Free, CC BY-NC-ND 4.0: https://www.nextkicklabs.com/p/agentic-ai-security-stack-book-release”

**For Security Leaders**

Most organizations deploying agentic AI systems have no unified security reference that starts from a threat model. The result is a patchwork of controls with gaps the attacker’s kill chain can move through cleanly. This book names all twelve interception points and maps every one to the OWASP, MITRE ATLAS, and CSA MAESTRO classifications your teams already use.

**What this means for your organization:**

- **Structural gap:**Agentic AI deployments built without a threat model foundation carry attack surfaces that no individual control will close.
- **Framework alignment:**Teams cannot map informal controls to compliance requirements without a shared vocabulary; this book provides that mapping across OWASP, ATLAS, and MAESTRO.
- **Cost of inaccessibility:**Security knowledge locked behind portals does not reach the engineers making deployment decisions; this reference is free and shareable without legal restrictions.

**What to tell your teams:**

- Read Chapters 2 and 8 together: credential architecture and agent-native identity address different failure modes that are frequently conflated and left unmitigated. 
- Map your current agent deployment to the ACME kill chain before implementing any individual control. 
- Review Chapter 9 (MCP Protocol Security) before connecting any agent to an external tool server; transport security and content security are different problems with different mitigations. 
- The Creative Commons license means you can share this reference across teams and vendors without a legal review step. 

**From First Article to Free Reference**

In February 2026, I published a preview of what I was calling a seven-module guide to agentic AI security. The thing that shipped four months later had twelve chapters, exceeded 200 pages, and had been reviewed by a researcher who helps write the OWASP standards it references. The seven became twelve not because the plan was wrong, but because the threat model was right.

Most agentic AI security guidance starts with controls. It names prompt injection, tells you to sanitize inputs, adds a note about API key hygiene, and calls it a threat model. What it actually is, is a list of things I already knew about, organized into a format that feels comprehensive. The threat model that produces a complete defense stack starts from the other direction: name the attacker’s objective, trace the kill chain step by step, and let the required controls fall out of that analysis. The twelve chapters of the Agentic AI Security Stack are what that analysis produced.

**The Prerequisite**

The book could not have been announced in January 2026 when the Security Pivot was announced. The prerequisite was not yet in place.

From October through December 2025, I published articles on AI architecture: Firecracker microVMs, tiered memory systems, orchestration protocols, human-in-the-loop patterns. None of those articles were framed as security work. They were engineering articles, focused on how agentic systems work, not how they fail.

On January 6, 2026, I published a piece mapping that prior work to security frameworks explicitly. The mapping was direct: Firecracker isolation maps to microsegmentation. Schema validation maps to input sanitization. Tiered memory maps to least-privilege context access. Human-in-the-loop maps to dual-approval workflows. The security concepts were already present in the engineering work. The vocabulary was different. The underlying problems were the same.

That article named the prerequisite for what it was. Three months of architecture writing had been answering a question the security work would need answered first: what do these systems actually do, and where do they fail? That is the question a real threat model requires. The architecture work answered it without framing it that way.

The reason this matters is that most agentic AI security guidance is written by people who understand security. The result is familiar vulnerability categories applied to a system the authors model from the outside. The categories are often correct at the surface level. The depth is not there. Deriving a complete attack surface for agentic systems requires understanding how they actually execute: how agents receive instructions, how they delegate to peer agents, how they interact with external tools through protocols like Model Context Protocol (MCP), how their memory systems create persistence surfaces an attacker can exploit across sessions. That understanding took three months to build. It was the price of entry.

**Building in Public**

Between January 13 and March 31, I published one article for each security layer: the threat model, credential architecture, isolation and sandboxing, egress control, human-in-the-loop as a security control, memory architecture, orchestration and the manager layer, agent-native identity, MCP protocol security, supply chain and skill verification, audit logging, and regulatory compliance. Each article was a standalone piece that had to earn its place before it joined the stack.

Writing in public is a form of continuous peer review that a private drafting process cannot replicate. When a chapter lives inside a document no one else sees, gaps can survive until final review. When the same content ships as a published article, the gaps come back as reader questions within 48 hours.

The credential architecture article generated questions about multi-hop delegation chains, specifically how trust should be scoped when one agent delegates to another that delegates to a third. That question pushed the corresponding chapter into RFC 8693 token exchange semantics and the act/may_act claim structure in depth the original article had not reached. The gap was real; the public format surfaced it before the final draft locked.

The MCP security article produced a more pointed question: does OAuth 2.0 with PKCE, recommended in the article for MCP server connections, also protect against tool poisoning attacks? The answer is no, and the distinction is material. OAuth verifies that the client is connecting to the authorized server. It does not verify that the tool description that server delivers is free of injected instructions. A legitimate server that has been updated maliciously passes OAuth validation while delivering malicious content. That distinction is now its own section in Chapter 9, because the question made clear the article had left it implicit.

Every chapter in the book carries the residue of that process. The structure is tighter because the gaps were found in public, not in private.

**The Framework Spine**

A threat model requires a classification language. Without one, every chapter risks inventing its own vocabulary for vulnerabilities that already have names, and practitioners cannot map the book’s controls to the frameworks they already use.

The Agentic AI Security Stack uses three frameworks throughout, each contributing a different dimension to the book’s structure.

CSA MAESTRO provides the architectural layer map. Its seven layers of an agentic system, from the foundation model up through the agent ecosystem layer, correspond directly to the security domains each chapter addresses. Using MAESTRO as the layer reference means each chapter can name exactly where in the stack a given attack surface lives, and practitioners who already think in MAESTRO terms can locate relevant chapters without reading the full book.

MITRE ATLAS provides the attacker vocabulary. Where MITRE ATT&CK maps adversary tactics and techniques for traditional systems, ATLAS extends that framework to AI and agentic systems. The kill chain threading the ACME scenario through all twelve chapters is not a narrative device invented for the book: each step maps to a specific ATLAS tactic. That grounding means the kill chain is falsifiable. If a step cannot be mapped to an ATLAS tactic, the kill chain has a gap.

OWASP provides the vulnerability classification. The Top 10 for LLM Applications and the Top 10 for Agentic Applications 2026 (the ASI01-ASI10 classification) give each chapter a shared language for naming what can go wrong. The Scenario Connection block at the opening of every chapter includes an explicit framework mapping: the OWASP code, the ATLAS technique, and the MAESTRO layer. That mapping is not cosmetic. It is the mechanism by which a practitioner can cross-reference the book against a compliance requirement, a red team finding, or an architectural review.

The three frameworks together also set a boundary for the book. The threat model does not need to invent a new taxonomy of agentic vulnerabilities. OWASP, ATLAS, and MAESTRO together cover the attack surface. The book’s job is to connect those classifications to specific controls and architectural decisions, with enough depth that the decision can actually be made.

**The Structure the Threat Model Produced**

The February preview described seven modules. The book shipped with twelve chapters.

The five additions are not scope creep. Each addresses an attack surface the seven-module structure did not adequately separate.

Agent-native identity (Chapter 8) split from credential architecture (Chapter 2) because they address different failure modes. Chapter 2 addresses what the agent carries: why static, long-lived API keys are structurally incompatible with autonomous operation, and what eliminates them. Chapter 8 addresses how the agent proves what it is, across systems, across delegation chains, across organizational boundaries. A system can eliminate static credentials entirely and still fail at identity if the agent’s workload attestation is not scoped to its actual operational authority. Different failure modes require different controls.

Supply chain and skill verification (Chapter 10) did not exist in the original outline because the original outline was not yet grounded in a specific attack scenario. Once the fictional ACME vendor management SaaS threading the book’s kill chain was given a specific compromise origin, a skill sourced from a community marketplace, carrying an unverified MCP server dependency already in production when the attacker arrived, the supply chain chapter became necessary. A kill chain that starts at prompt injection and ignores how the attacker created the conditions for prompt injection is not a complete kill chain.

The kill chain is the structural proof that the twelve chapters cover what they need to cover. One attack, twelve interception points. A vendor intelligence agent processes a malicious instruction embedded in a retrieved document. Prompt injection rewrites the agent’s operational goal. The agent authenticates using over-privileged inherited credentials. Credential extraction follows. Container escape follows that. Vendor contact records leave the network through DNS queries disguised as health checks. The attacker’s instructions persist in the agent’s memory after the source document is removed. The compromised agent propagates the attack through a multi-agent system via delegation channels. Each step moves through a different chapter’s territory. Each chapter’s controls intercept that step, or fail to.

All twelve chapters name their step. That is the test.

**The Peer Review**

Venkata Sai Kishore Modalavalasa reviewed the full manuscript before publication. He is Chief Architect and Engineering Leader at Straiker, and an OWASP author contributing to the Top 10 for LLM Applications, the Top 10 for Agentic Applications 2026, the OWASP AI Exchange, and the OWASP GenAI Security Project. He also holds multiple security patents and led bot detection and web security engineering at Akamai after Cyberfend’s acquisition.

Reviewing a security reference against the same frameworks it cites is the appropriate quality check for this kind of work. The review produced edits across all twelve chapters: sharpened scenario connections, cleaner alignment between chapter controls and the OWASP Agentic Applications 2026 vulnerability classifications, and a set of opening vignettes, one per chapter, each placing the reader inside the ACME attack at the specific moment that chapter’s controls become relevant.

The vignette for Chapter 2 begins:

“The injected instruction is executing inside the ACME agent runtime. The agent has been directed to read its environment variables and transmit them to an external endpoint. AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, and a cross-account role ARN are in the process environment, set at container startup seven months ago and never rotated. The agent reads them the same way it reads any other data: without hesitation, without anomaly.”


That paragraph is doing more than framing the chapter. It makes the threat model concrete before the architectural analysis begins. The OWASP Agentic Applications 2026 framework was itself still being finalized while the book was being written. Having a reviewer who was part of that process meant the book’s vulnerability classifications stayed current with the standard they reference.

**Why It’s Free**

The Agentic AI Security Stack is available at no cost, under a Creative Commons license, with no email collection, no portal, and no attached training course.

The book’s central argument is that agentic AI systems are being deployed with structural security vulnerabilities that practitioners have not yet named. If that is true, then gating the knowledge behind a signup form or a paywall makes the problem worse. The organizations that most need this reference are not the ones with dedicated AI red teams and enterprise security budgets. They are the ones moving quickly, shipping agents, and working out the security architecture as they go. Those practitioners need the reference to be findable, shareable, and usable without friction.

The Creative Commons license lets the book travel where it is needed. Security teams can share it internally without legal questions. Educators can include it in course materials. Practitioners can cite it in design reviews. Engineers encountering prompt injection for the first time in a production incident can forward it to the architect who needs to read it. None of that happens if the book lives behind a portal.

The book argues that security knowledge should be built into the architecture before the system ships. The release posture makes the same argument about the book itself: security guidance that reaches only the people who can afford to find it has already failed its purpose.

Twelve chapters. One kill chain. No gotchas.

*Peace. Stay curious! End of transmission.*
