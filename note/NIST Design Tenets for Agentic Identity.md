# NIST Design Tenets for Agentic Identity

NIST's NCCoE proposes ten design tenets for IAM systems that manage AI agents: distinct cryptographically verifiable agent identities, attenuating delegation chains, zero standing privilege, JIT access with runtime verification, human escalation on risk thresholds, signed mandates, architectural separation of policy from reasoning, deterministic access decisions, agent-scale monitoring, and independent append-only audit. Each tenet names the risk it mitigates, and NIST is explicit that they compose — no single tenet stops a confused-deputy attack on its own. The reader comments are substantive enough to treat as part of the source.

---

## Key quotes

> "Large Language Models (LLMs) accept data and instruction as a single stream of tokens through a single interface. This results in a lack of separation between the data ... and control plane ... Thus, this separation must be enforced by architecture design ... **Guardrails alone are insufficient.**"

The strongest sentence in the piece: NIST, a standards body not known for rhetorical flourish, states flatly that guardrails don't work and the fix is architecture. This lands squarely with [[CaMeL — Defeating Prompt Injections by Design]], which is exactly that architectural enforcement, and with [[OpenAPPA]], which puts information-flow tracking outside the prompt loop for the same reason.

> "Organizations maintain deterministic capabilities as the primary mechanism for making access decisions in agentic scenarios. While probabilistic information can be used as input into the decision ... it is not used as the primary mechanism."

The same duck-typing principle this wiki keeps circling ([[Deterministic Spine, Agentic Leaves]], [[Bondsmith — Hard Bonds for Soft Claims]]), now stated as national-standards guidance for authorization specifically: the LLM may propose, but a deterministic authorizer disposes.

> "If an agent's activities cannot be definitively linked to the human or entity that initiated them, organizations face severe non-repudiation issues..."

Tenet 2 makes attenuating delegation the core primitive — downstream entities can never claim more authority than the parent. This is the formal-standards echo of [[Real-World Lessons from Agentic Authority Overreach]] (actual vs. apparent authority) and the trust-ratchet logic in [[The Agent Access Model]].

> (Comment, newauth founder): "If an agent can reach the key that signs a mandate, the chain of trust ends at a device, not a person."

The sharpest external critique. Tenets 2, 5, and 6 all hang off a human at the root of the chain, but passkeys live in the same browser as the agent. The proposed fix — a fresh human response bound to the action, generable by nothing in the agent's environment — exposes that NIST's chain of trust has a hole at its root. Compare [[Passkeys Are a Trap, Do Not Use Them]]'s argument that credential branding hides unlike things.

## Key themes

#concept #security #identity #standards

## Analysis

The tenets are good but conservative — they are essentially classical zero-privilege IAM (JIT, least privilege, separation of duties, append-only audit) restated for a principal that hallucinates. That restatement is the real contribution: two tenets (#7 policy/reasoning separation and #8 deterministic decisioning) concede the *structural* point that the reasoning engine is untrusted, which most enterprise vendor material still won't say. Framing it as tenets-with-rationales rather than a checklist is smart, because it lets implementations differ while the threat model stays fixed.

What the tenets skip is as telling as what they include. No reversibility classification (a commenter points to OWASP AISVS 9.2.3's read-only-through-irreversible scale as a published answer). No cross-organisation case — the financial-services comment about a consumer's agent asking a bank to pay is precisely where tenets 2, 6, and 10 have to work across trust domains, and the answer (a neutral registry anchoring mandate chains) is the kind of infrastructure question a national standards body is uniquely positioned to address. And NIST's own honesty — "not all of these design tenets will be immediately fulfilled by the current state of standards and technology" — is the right posture: this is a roadmap, not a spec.

## Relations

This source strengthens [[Identity Management for Agentic AI (OpenID Foundation)]] by supplying a government-standards counterpart to the OIDF whitepaper's open-problems list — consent fatigue and recursive delegation appear in both, now as named tenets. It nuances [[Agent Identity]], which argues agent identity is a distinct lifecycle problem; NIST gives that argument a ten-item structure with mitigated risks attached. It complicates [[The Agent Access Model]]: Cloudflare's task-scoped-credential principles now have standards-body backing, but the comment thread surfaces the cross-organisation gap both leave open. And it strengthens [[CaMeL — Defeating Prompt Injections by Design]] — tenet 7's architectural separation of policy from reasoning is the policy-level statement of CaMeL's design-level proof.

---
*Sources: [[raw/design-tenets-agent-identity-management-systems]], [[summary/design-tenets-agent-identity-management-systems]]*
*Last updated: 2026-11-06*
