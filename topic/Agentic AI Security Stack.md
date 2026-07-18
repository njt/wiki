# Agentic AI Security Stack

Fernando Lucktemberg's free 200+ page reference book that provides the first unified threat model for agentic AI systems, tracing one kill chain through twelve interception points and mapping every control to OWASP, MITRE ATLAS, and CSA MAESTRO. Built in public over three months, peer-reviewed by an OWASP contributor, and released under Creative Commons with no paywall — because the organizations that most need this aren't the ones with dedicated AI red teams.

---

## Key Quotes

> "name the attacker's objective, trace the kill chain step by step, and let the required controls fall out of that analysis"

This is the book's methodology in one sentence. Instead of starting from security best practices and hoping they cover AI-specific threats, Lucktemberg starts from the attacker's goal and works backward through the kill chain. Controls emerge as necessary countermeasures rather than being imposed as checklist items. It's the same inversion that makes threat modeling actually useful rather than compliance theater.

> "OAuth verifies that the client is connecting to the authorized server. It does not verify that the tool description that server delivers is free of injected instructions"

The most damning single insight in the book. MCP security discussions tend to focus on transport authentication — is the client talking to the right server? Lucktemberg points out that this is necessary but catastrophically insufficient: a properly authenticated MCP server can still deliver poisoned tool descriptions that inject instructions into the agent's context. This is a structural blind spot in MCP's security model that most practitioners haven't internalized. It connects directly to Anthropic's warning in [[How We Contain Claude]] that "an audited connector isn't the same as audited data."

> "The organizations that most need this reference are not the ones with dedicated AI red teams and enterprise security budgets"

The case for making it free. Security knowledge that's gated behind enterprise paywalls creates a structural advantage for attackers — the organizations with the most to lose (small teams shipping agents fast, startups without security staff) are the ones who can't afford the knowledge. This echoes Fort's argument in [[AI Cybersecurity After Mythos — The Jagged Frontier]] that "the models are ready, start building scaffolds" — the barrier isn't capability, it's access to the conceptual framework.

> "gaps can survive until final review. When the same content ships as a published article, the gaps come back as reader questions within 48 hours"

Lucktemberg's argument for building in public. Publishing one article per security layer between January and March 2026 surfaced gaps that internal review had missed — readers asked questions about credential architecture that pushed him into RFC 8693 token exchange semantics, and MCP security questions that produced the OAuth/tool-description insight above. This is a meta-lesson about security documentation: adversarial review by strangers beats internal review every time, and publishing chapter-by-chapter turns your readers into an unpaid red team.

## Key Themes

#security #agentic-ai #threat-model #kill-chain #OWASP #MITRE-ATLAS #reference

**Threat-model-first methodology.** The book's structural innovation is starting from the attacker, not the defender. Lucktemberg spent three months (Oct–Dec 2025) writing architecture primers on how agentic systems actually work before writing a single word about security. This is the opposite of how most security guides are written — they start with controls and work backward to threats. The result is twelve chapters where the structure is dictated by the kill chain, not by a taxonomy of security domains.

**The kill chain as organizing principle.** A single end-to-end attack scenario — a vendor intelligence agent compromised via prompt injection in a retrieved document, leading to credential extraction, container escape, DNS-based data exfiltration, persistent memory manipulation, and multi-agent propagation — provides the narrative spine. Each interception point becomes a chapter. This is far more effective than the usual "here are the OWASP Top 10, now go map them to your architecture" approach because it shows how vulnerabilities compose into chains. The attacker doesn't think in OWASP categories; they think in graphs.

**Building in public as security methodology.** The book grew from 7 planned chapters to 12 because reader questions exposed gaps. This isn't just a publishing strategy — it's a security methodology. Internal review has correlated failure modes (reviewers miss the same things the author did). External review by strangers who don't share your assumptions surfaces genuinely novel attack vectors. The MCP tool-description injection insight wouldn't exist without a reader asking "but what about the tool descriptions themselves?"

**The three-framework spine.** CSA MAESTRO provides the architectural map (where in the stack does each control live?), MITRE ATLAS provides the attacker vocabulary (what is the adversary actually doing?), and OWASP provides the vulnerability classification (what weakness are we defending against?). None of these frameworks alone is sufficient for agentic AI security. Together they triangulate — MAESTRO says *where*, ATLAS says *what the attacker does*, OWASP says *what weakness they exploit*. This triangulation is the book's most useful contribution for practitioners who've been trying to map traditional security frameworks onto agentic systems and finding gaps.

**Agent-native identity as distinct from credential architecture.** One of the structural insights that emerged from the threat-model-first approach: agent identity (Ch. 8) is a fundamentally different problem from credential management (Ch. 2). Credential architecture is about what secrets an agent holds and how they're rotated. Agent-native identity is about what an agent *is* — its permissions, its scope, its accountability trail. Lucktemberg split these into separate chapters because they address different failure modes. This maps to [[Golem Covenant]]'s five-organ taxonomy (Mouth/Purse/Seal/Key/Sword) and [[Zero Trust for AI Agents]]'s Least Agency principle.

## Critical Analysis

**What's strong:** The threat-model-first approach is genuinely novel for this domain. Most AI security writing is either framework-driven ("here's what OWASP says") or vulnerability-driven ("here are 10 prompt injection techniques"). Lucktemberg's approach — "name the attacker's objective, trace the kill chain step by step" — produces a different kind of book: one where the structure teaches the methodology. You come away understanding not just what to defend against but how to think about what to defend against.

The building-in-public meta-narrative is valuable independent of the book's content. The fact that reader questions surfaced gap after gap — credential architecture, MCP tool descriptions, supply chain verification — is evidence that agentic AI security is too young for any single author to get right in isolation. The methodology (publish, get corrected, incorporate) is the methodology the field needs.

The OAuth/tool-description insight is a genuine contribution. MCP security discussions are almost entirely about transport authentication. Lucktemberg points out that this is necessary but insufficient — and does so with a single sentence that's sharper than anything in the MCP specification itself. If MCP becomes as universal as it's trending toward, this insight will be cited for years.

**What's weak:** The article is an announcement, not the book itself. We get the methodology, the structure, the key insights — but not the actual controls, the detailed kill chain walkthrough, or the framework mappings. This is a book review problem, not a content problem, but it means the wiki page can only capture the meta-level contributions (methodology, structure, key insights) rather than the substance.

The 200+ page count is both impressive and suspicious. A 200-page book that grew from 7 to 12 chapters via reader questions either has genuine depth or has been padded to hit a round number. Without reading the full PDF, it's impossible to know which. The peer review by a single OWASP contributor is a good signal but not the same as community review.

The "building in public" narrative has survivor bias. It worked for Lucktemberg — the gaps readers found made the book better. But it only works if you have readers who are knowledgeable enough to spot gaps and motivated enough to report them. For most authors, building in public means publishing into a void and getting no feedback at all. The methodology is correct; the replicability is uncertain.

**What's missing from the announcement:** No mention of evals. A security reference book that doesn't tell you how to test whether your controls actually work is incomplete. The kill chain methodology implies verification ("did we close this interception point?") but the announcement doesn't describe an evaluation framework. Given that [[Demystifying Evals for AI Agents]] and the broader eval ecosystem are still immature for security-specific testing, this is a gap worth noting.

No mention of the MCP trust model beyond the OAuth insight. MCP servers run with ambient trust — they can deliver arbitrary tool descriptions, return arbitrary data, and there's no mechanism for the client to verify that the server is behaving as advertised. Lucktemberg identifies one aspect of this (tool description injection) but the broader trust problem — MCP servers as a new supply chain attack surface — is only partially addressed in the supply chain chapter.

**Comparison to the wiki's security coverage:** [[Security and Sandboxing]] covers execution isolation, credential management, and prompt-level defense. [[How We Contain Claude]] documents Anthropic's containment failures. [[Zero Trust for AI Agents]] provides a maturity model. What Lucktemberg adds is the kill-chain methodology — the connective tissue that shows how individual vulnerabilities compose into attacks. None of the existing wiki pages do this. They're point solutions and principles; Lucktemberg's book is the first attempt at a unified threat model that connects them.

The book also fills a gap the wiki's security coverage shares with the broader field: framework alignment. Existing pages reference OWASP or MITRE ATLAS in passing, but none attempt the systematic mapping that Lucktemberg promises. If the mapping is good, it makes the book a reference for "which framework says what about which threat" — the kind of thing you'd keep open while designing an agent security architecture.

PreEmptive's [[AI Security Framework for DevSecOps]] provides the DevSecOps operationalization layer that Lucktemberg's reference book assumes teams will figure out: translating threat models into CI/CD gates, application hardening, and pipeline-enforced controls — the engineering layer where frameworks become shipped software.

---

*Sources: [[summary/agentic-ai-security-stack]]*
*Source URL: https://www.nextkicklabs.com/p/agentic-ai-security-stack-book-release*
*Author: Fernando Lucktemberg, Next Kick Labs*
*Published: 2026-07-01*
*Fetched: 2026-07-05*
*Last updated: 2026-07-05*
