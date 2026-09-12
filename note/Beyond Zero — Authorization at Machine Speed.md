# Beyond Zero — Authorization at Machine Speed

Alphabet Security's ACM Queue article proposing Beyond Zero as the successor to BeyondCorp: the trust boundary shrinks from the application to the individual action on a specific resource, and authorization runs at machine speed for humans and agents alike — a static policy floor coupled with a dynamic AI reasoning ceiling, graduated challenge/containment friction, and a precomputed enterprise world model. This is the canonical queue.acm.org fetch of the article already analyzed at [[Beyond Zero — Enterprise Security for the AI Era]] (ingested six weeks earlier from a mirror URL); the text is unchanged, but this page covers what the canonical fetch adds — post-publication evidence that the paradigm is spreading — and pushes harder on the paper's most aggressive claim.

---

## Key Quotes

> "Beyond Zero shifts the trust boundary from the application to the action being performed on a piece of data in realtime—and from after-the-fact investigation to in-the-moment evaluation and containment."

Two moves in one sentence, and the second is the underrated one. The spatial shift (application → action) gets the attention, but the temporal shift — from after-the-fact SecOps review to in-the-moment evaluation — is what fuses access management and security operations into a single system. Investigation stops being a separate function; it becomes a stage in the authorization pipeline.

> "The underlying principle here is to minimize the overhead of individual realtime access decisions by precomputing as much of the reasoning outputs as possible."

This is context engineering transplanted into the authorization layer. The "enterprise world model" — who you are, what the data is, what you're assigned to work on — only works because inference is front-loaded ahead of the access request and cached as attributes. It's the same hot-path/precomputed split any agent harness uses when the latency budget can't fit an LLM call, and it quietly concedes the central constraint: you cannot reason at machine speed, so you must cache reasoning done at human speed.

> "the rise of agentic infrastructure introduces new attack vectors, such as the exploitation of ambient authority, where an agent is granted the full, often overprovisioned permissions of its human user."

The article's sharpest threat-model contribution: naming ambient authority as *the* agentic vulnerability class. The dangerous agent isn't the compromised one; it's the ordinary one faithfully exercising permissions its human never needed. This is the claim [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]] argues from principle and [[How We Contain Claude]] demonstrates empirically — and it is why Beyond Zero insists the unit of authorization is the action, never the accessor.

> "User Intent + Agent Intent can be interpreted and checked to ensure alignment and limit prompt injection risks." (Table 1)

Prompt injection has been framed almost entirely as a content problem — sanitization, delimiting, instruction hierarchy. This line reframes it as an authorization problem: when an agent's prompt inputs, execution plans, and tool invocations are logged as policy inputs, a hijacked agent looks like what it is — an accessor whose observed intent diverges from its controlling user's — and can be challenged or contained at the moment of access rather than after the exfiltration.

> "These decisions then factor into out-of-band risk evaluations, which then become attributes that can be referenced in subsequent evaluations (even across attempts to access other applications or resources)."

Verdicts become inputs. This is the feedback loop that makes the system adaptive — and its softest attack surface, since an attribute poisoned once propagates into every later decision across every application (see analysis below).

> "In the vast majority of cases, the decision to contain will be autonomous. Only a tiny percentage would be escalated to human review when there is a high confidence in malicious intent."

The paper's most aggressive sentence, and it deserves more scrutiny than it gets. Enforcement is automated; the appeal path is not — elsewhere the article notes some containments "might be lifted only when the security team interviews the contained user and their manager." Machine-speed lockout with human-speed unlock is an availability bet the article never prices.

## Key Themes

#concept **Machine-speed authorization.** The argument is symmetrical: attackers already operate at machine speed (LLMs rewriting malicious code on demand, agentic scanning of previously low-value surfaces), and agents already request at machine speed (10× human rates, tens of millions of concurrent actions per second). A human-speed decision layer fails in both directions at once. Table 1's "Decision Speed: Human → Machine" row is the whole case in two cells.

#pattern **Precompute-then-enforce.** Latency budgets, not modeling power, shape the architecture. The world model is built continuously and slowly, then consulted instantly; a hot cache serves access-time checks while a long-term store feeds slow anomaly inference. It is a memory hierarchy for authorization.

#concept **Ambient authority as the attack surface.** The agent inheriting its human's overprovisioned permissions is the canonical agentic risk; per-action evaluation is the proposed antidote, and the "access bubble" that flexes with the task is its mental model.

#pattern **Graduated friction.** Allow, challenge, contain — three severities driven by one policy surface, so escalation is automatic rather than dependent on a human noticing. Challenges gather context (justification, security key, approval, biometric); containments revoke it, durably.

#concept **Intent as a policy input.** The "activity window" before and after an action, plus agent-internal signals, become checkable attributes. Intent misalignment is treated as a policy violation — the authorization layer's answer to prompt injection.

#person **The BeyondCorp lineage.** Zalewski (lcamtuf — distinguished engineer, former Snap CISO) and Valente (director of product management) are Alphabet Security leadership declaring their own prior revolution's assumptions dead. A paradigm's authors pronouncing its end-of-life is rarer and weightier than an outsider doing it; the attack scenarios (curious contractor, suddenly foolish administrator) have the texture of incidents these people have actually handled.

## Critical Analysis

**The enforcement/appeal asymmetry is the unpriced cost.** Containment is autonomous; un-containment is an interview with the security team and your manager. In the article's own immune-system framing, this is engineering the response and hand-waving the autoimmunity: false-positive containments lock out legitimate work at machine speed and heal at ticket speed. The article gestures at containments that "move up or down in strength should a completed challenge clarify that the detected risk was a false positive" — but a challenge-capable containment is the mild case. No false-positive rate is offered, no appeal SLA, no discussion of what a containment storm does to an organization's trust in the system. Until those exist, a "self-defending enterprise" is also a self-bricking one.

**The feedback loop is the adaptive-attack surface.** Decisions become attributes; attributes feed subsequent decisions across applications. That composition is what makes Beyond Zero an immune system rather than a firewall — and what makes it steerable. An adversary who learns which observables move risk scores (peer-group deviation, subject-matter crossover, work-assignment mismatch) can stay just under thresholds, or can deliberately shape a target's attribute history so a legitimate user trips containment. Every adaptive defense has this property; the article presents the loop purely as a defensive asset. "Attacker moves second" applies to the defense itself.

**The prerequisite is the moat.** Autonomous governance assumes machine-consumable HR data, project assignments, document classification, and behavioral baselines. Google has these because Google spent twenty years building them; most enterprises have none in usable form. The architecture is downstream of an organizational capability the article treats as given — the same gap that made BeyondCorp take a decade to diffuse beyond the companies with Google-shaped internal infrastructure.

**The standards section is the real product, again.** The BeyondCorp playbook repeats: publish the vision, dogfood internally, then standardize so the ecosystem must follow. The asks — standardized agent-introspection APIs, request annotations attributing every action to an agent, a controlling user, and a task, and external pluggable policy evaluation as "a first-class citizen for all software-as-a-service products" — would make every SaaS product policy-evaluable from the customer's side. That last demand is a business-model demand on the entire SaaS industry, not a technical proposal; complying vendors surrender a control point. The NIST name-drop is deliberate signaling. [[Authentication Is Largely Solved]] supplies the tide this rides: with authentication effectively closed, the field's energy has moved to authorization — exactly the layer Beyond Zero rebuilds for agents.

**What the canonical fetch adds: the paradigm is landing.** The mirror-URL ingest captured the vision; this fetch captures the reception. The article's page now shows ~40K downloads and its first citation — Mok, Jo & Lee, "An Action-Centric Zero Trust Maturity Model for Agentic AI Environments" (Sensors, Aug 2026) — third parties converting the position paper into maturity models within a month of publication. Whatever else it is, Beyond Zero has become the reference text of the "agentic security" category.

## Related Pages

- The same article, ingested six weeks earlier from a mirror URL, is analyzed in [[Beyond Zero — Enterprise Security for the AI Era]]; that page's floor/ceiling reading and its Cloudflare comparison stand, and this page strengthens them with the post-publication evidence and a harder line on autonomous containment — a compile step should treat the two as one source rather than double-count them.
- This source gives [[The Agent Access Model]] its enterprise-scale counterpart: Cloudflare's AAM supplies the runtime primitives Beyond Zero leaves abstract — task-scoped credentials and the Trust Ratchet are what the "access bubble" looks like as an enforcement mechanism — while Beyond Zero supplies the cross-application reasoning layer AAM admits remains unsolved. Each strengthens the other precisely where it is weakest.
- [[How We Contain Claude]] is the empirical premise of this source's entire design: when users approve roughly 93% of permission prompts, human-in-the-loop approval cannot be the security boundary, and authorization must move into the machine path. Beyond Zero is what that move looks like as enterprise policy.
- The source both uses and complicates [[Bounding the Blast Radius — Prompt Injection Defenses]]: it relocates prompt-injection defense from the prompt layer to the authorization layer — intent checked as a policy input — which strengthens the runtime-defense case, but its attribute-feedback loop is exactly the adaptive surface on which "attacker moves second" wins, so the economic framing still governs.

---
*Sources: [[raw/3819083]], [[summary/3819083]]*
*Last updated: 2026-09-13*
