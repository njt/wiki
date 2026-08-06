# Beyond Zero — Enterprise Security for the AI Era

Google/Alphabet Security's proposal for the next evolution beyond zero trust: a security architecture that makes per-resource, per-action authorization decisions at machine speed, coupling static policy floors with dynamic AI-driven reasoning ceilings. Published as the spiritual successor to the 2014 BeyondCorp whitepaper that reshaped enterprise security.

---

## Key Quotes

> "The assumptions underpinning BeyondCorp—that accessors are human, that actions occur at human speed, and that applications are the correct boundary for trust—are no longer sufficient."

The article's thesis in a sentence. Every assumption that made BeyondCorp revolutionary a decade ago has been invalidated by AI agents. The authors are in the unusual position of arguing against their own prior revolution — and they don't flinch.

> "Beyond Zero shifts the trust boundary from the application to the action being performed on a piece of data in realtime—and from after-the-fact investigation to in-the-moment evaluation and containment."

The boundary shrinkage is the architectural move. Applications were the right trust boundary when humans clicked through them at human speed. When agents reason across datasets at 10× the rate and can chain through APIs, MCP servers, and unstructured data in a single workflow, the application boundary becomes a sieve.

> "Static policies (the floor) to ensure baseline security and compliance. Layered on top is a dynamic reasoning engine (the ceiling) that observes behavior and applies friction when an action deviates from established norms."

The floor/ceiling metaphor is the article's cleanest conceptual contribution. Pure static policies can't handle the combinatorial explosion of agentic access patterns. Pure dynamic policies can't be statically verified and create compliance nightmares. The blend is the insight — and it's the same architecture pattern that makes [[The Agentic Product Standard v2.0]]'s harness work: deterministic guardrails plus adaptive reasoning.

> "The access bubble that controls their permissions dynamically flexes to be bigger or smaller depending on what they need to do in a given moment, reducing overprovisioning without impacting productivity."

This is the practical UX promise buried in the architecture. Beyond Zero isn't just about stopping attacks — it's about eliminating the ambient authority problem where every agent runs with its human's full permissions. The access bubble is a genuinely useful mental model for what [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]] calls for but doesn't operationalize.

> "Containments are more durable 'stop signs' that are designed to introduce substantial friction to an accessor's access in order to stop attacks."

The challenge/containment distinction is important and underdiscussed. Challenges are interactive friction (explain yourself, touch your key, get approval). Containments are programmatic revocation that escalates. The architecture treats them as a single graduated-response surface — same policy language, different severity. This is a cleaner design than the ad-hoc "alert then manually investigate" workflow that dominates current SecOps.

> "Users approved roughly 93% of permission prompts."

Not from this article — that's from [[How We Contain Claude]] — but it's the unstated premise behind Beyond Zero's entire design. If humans rubber-stamp permission prompts, the only viable security boundary is one that doesn't depend on human judgment at access time. Beyond Zero's reasoning engine is the attempt to automate that judgment without the 93% approval rate.

## Key Themes

#concept **Resource-level trust boundary.** The core architectural shift: authorization decisions move from "can you access this application?" to "can you perform this specific action on this specific piece of data right now?" This is attribute-based access control (ABAC) taken to its logical extreme, but with the crucial addition of AI-preprocessed context that makes per-resource decisions computationally tractable.

#pattern **Floor-and-ceiling security.** Static policies define the inviolable baseline. Dynamic reasoning adds adaptive friction above it. This is the same pattern that works in agent harnesses (deterministic tool allowlists + LLM judgment calls), applied to the authorization layer. The floor prevents catastrophe; the ceiling prevents the thousand smaller failures that static policies can't encode.

#concept **Precomputed enterprise world model.** The autonomous governance component front-loads inference: who people are, what data is sensitive, what work they're assigned to. All of this is preprocessed by AI into queryable attributes before access time. The insight is that real-time authorization has a latency budget measured in milliseconds, and you can't run LLM inference in that window — so you run it ahead of time and cache the results as structured attributes.

#pattern **Graduated response (challenges → containments).** Instead of binary allow/deny, Beyond Zero introduces a spectrum: allow, challenge (justification, verification, approval, biometric), or contain (partial or full access revocation). The key design property is that the same policy infrastructure drives all three, so escalation is automatic rather than requiring a human to notice and intervene.

#concept **Agent intent as a security input.** One of the article's sharpest ideas: an agent's chain-of-thought, tool invocations, and execution plan become signals fed into the reasoning engine. The system doesn't just ask "is this agent authorized?" — it asks "does this agent's actual behavior align with its stated task and its user's intent?" This is prompt injection defense at the authorization layer rather than the prompt layer.

#tool **BeyondCorp lineage.** The article explicitly positions itself as the successor to Google's 2014 BeyondCorp paper, which launched the zero-trust movement. The authors are senior Alphabet Security leaders (Valente is a director of product management; Zalewski — lcamtuf — is a distinguished engineer and former Snap CISO). This isn't a think piece; it's a declaration of architectural direction from the company that defined the prior paradigm.

## Critical Analysis

**The honesty about what breaks is the article's strength.** The authors don't pretend Beyond Zero solves everything. They're explicit about the gaps: preprocessing latency is a real constraint, dynamic policies are harder to statically verify, and the whole thing depends on an enterprise world model that most organizations don't have. The call for industry standards (pluggable policy evaluation, agent context annotations) is an admission that Google can't build this alone — and probably can't build it at all without the SaaS vendors making their products Beyond Zero-compatible.

**The floor/ceiling metaphor is doing a lot of work, and it might not hold.** The article presents static and dynamic policies as cleanly separable layers, but in practice they'll bleed into each other. A static policy that says "no access to crown-jewel data without a valid work assignment" depends on the autonomous governance system having correctly classified both the data and the assignment — and that classification is itself dynamic and fallible. The floor isn't as solid as the metaphor implies. This is the same class of problem that makes [[Bounding the Blast Radius — Prompt Injection Defenses]] conclude that every layer collapses under adaptive attack: the static/dynamic boundary is a design aspiration, not a formal guarantee.

**The "rogue agent" example is doing two jobs, and one of them is dishonest.** The SalesGenie scenario is pedagogically useful — it walks through a concrete decision pipeline. But it also presents Beyond Zero as catching an attack that BeyondCorp would miss, and the framing is suspiciously neat. In the example, the agent is authorized for sales reports but queries strategic planning documents. The Beyond Zero check catches it through work-assignment mismatch. But this framing dodges the harder case: what happens when the agent's *legitimate* work assignment gives it access to data that it then exfiltrates? The system catches the contractor who shouldn't be there; it's less clear it catches the insider who should.

**Cloudflare's Agent Access Model (AAM) takes the next step Beyond Zero gestured at but didn't specify.** AAM's Trust Ratchet — a mechanism that narrows the capability ceiling of a task execution graph in-flight, before protected data reaches the model — is the runtime enforcement primitive that Beyond Zero's "access bubble" metaphor implies but never operationalizes. AAM's task-scoped credentials (RFC 8693 Token Exchange + DPoP binding) and dual-boundary mediation (harness + network) give Beyond Zero's floor/ceiling architecture a concrete implementation path. See [[The Agent Access Model]] for the full proposal.

**The call for standards is the real product.** The article's final section — open architectures, agentic identity standards, pluggable policy evaluation — reads like a standards strategy document. Google is attempting to do for enterprise security what it did with BeyondCorp: publish the vision, build the internal implementation, then work to make the standards universal so that every SaaS vendor has to support them. The NIST mention is deliberate signaling. Whether the industry follows depends on whether Google can convince Microsoft, AWS, and the major SaaS platforms to make their products externally-policy-evaluable — which is a business-model question as much as a technical one.

**The biggest unstated assumption is organizational.** Beyond Zero requires an enterprise world model built from HR systems, project management tools, document classification pipelines, and behavioral baselines. Most enterprises have none of these in machine-consumable form. Google does because Google built them. The gap between "here's the architecture" and "here's how you build the prerequisites" is where most organizations will stall. This is the same adoption problem that made BeyondCorp take a decade to diffuse beyond the companies that had Google's internal infrastructure.

**Zalewski's involvement matters.** lcamtuf is one of the most respected names in security engineering — author of *The Tangled Web*, *Silence on the Wire*, and the foundational browser security research that shaped modern web security. His name on this paper signals that this isn't a product marketing document. It's an architecture proposal from someone who has spent decades thinking about what breaks and why. The attack scenarios (curious contractor, suddenly foolish administrator) have the texture of real incidents, not hypotheticals.

**The immune system metaphor is the right one, and it's underdeveloped.** The article closes by describing Beyond Zero as "security as an immune system that continuously adapts to the context and intent of every request." This is the right framing — the immune system doesn't block all threats, it detects and responds to anomalies with graduated responses — but the article only gestures at it. The real immune-system insight is that false positives (autoimmune disorders) and false negatives (infections) are managed through feedback loops, not eliminated. A fuller treatment of how Beyond Zero handles its own false positives — challenges that harass legitimate users, containments that lock out people who did nothing wrong — would make the metaphor land.

---

*Sources: [[raw/beyond-zero]]*
*Last updated: 2026-08-01*
