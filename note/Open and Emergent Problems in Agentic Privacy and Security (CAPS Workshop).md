# Open and Emergent Problems in Agentic Privacy and Security (CAPS Workshop)

The Google CAPS workshop manuscript is the field's most systematic statement yet that agent security cannot be reduced to either model training or static policy: 50+ authors from Google and academia frame trustworthy agents through Contextual Integrity — privacy as *contextually appropriate information flow* rather than secrecy — and enumerate roughly a hundred open problems across model, system, user, and multi-agent ecosystem levels. Its distinctive claim is that the security primitive agents need is "appropriateness," judged dynamically per task, not consent collected once.

---

## What the paper argues

Three agent properties break conventional security engineering. Unstructured interfaces mean trusted and untrusted data arrive in the same natural-language channel — the ground truth of prompt injection. Probabilistic control flow (generative planning, code synthesis) defeats testing and secure-development practice built on deterministic paths. And autonomy with delegation makes per-action confirmation collapse into consent fatigue, while higher-level task descriptions widen the gap between what the user meant and what the agent does.

The fix it proposes is not another wall but a normative layer. Borrowing Nissenbaum's Contextual Integrity, the paper defines "contextual security": boundaries, permissions, and threat models that are judged appropriate to the context — a gym booking system does not need nation-state hardening, and "how society agrees upon a reasonably accurate adversarial model" becomes itself an open problem, citing how Snowden's revelations retroactively changed what "secure enough" meant for Google's inter-datacenter links.

## Key quotes

> "The core challenge of controlling AI agents lies in precisely understanding which actions are appropriate in every context in which the agent operates and enforcing such appropriate behavior."

The framing that separates this from the rest of the agent-security literature: the problem is stated normatively (what *should* the agent do here?) before it is stated mechanically (how do we enforce it?). Most of the wiki's security sources start from the mechanical side and never ask the normative question.

> "In highly autonomous multi-agent systems, this inevitably leads to consent fatigue, and, combined with more abstract task descriptions, increases the chance of unwanted behavior."

An unusually blunt admission that notice-and-choice — the entire post-GDPR privacy apparatus — does not survive agent autonomy. The user-level chapters argue for replacing static permissions with dynamic control, "what if" scenario rehearsal, and circuit breakers, which is a redesign of the interaction model, not just of the sandbox.

> "Sharing data appropriately still allows agents to achieve high utility since they are not required to keep all user data secret, but it also protects users in that agents cannot liberally share any data that the user has given them access to."

This dissolves the false binary that plagues current deployments — broad access versus data minimization — and gives a principled target for capability policies: the question is not "may the agent see this?" but "is *this flow* appropriate to this context?"

## Key themes

#concept #pattern #security #privacy

- **Contextual Integrity as an engineering spec, not a philosophy paper.** The CI-tuple (actor, role, data type, transmission principle) becomes a schema a policy engine could consume.
- **Enforcement across three levels.** Model-level contextual reasoning (and synthetic RL environments to teach it), system-level auth/authorization/accounting for agentic apps, user-level trust calibration — the open-problem list refuses to pick one layer.
- **The agentic ecosystem problem.** Verifiable agent identity, agent-to-agent trust, cross-agent incident response, and non-repudiation: security for fleets, not single agents.
- **Governance as the last mile.** Who sets the norms, and how is their enforcement audited — the questions every other framework quietly assumes away.

## Opinionated take

This is the most important kind of document and the least fun to read: a workshop consensus that mostly confirms what practitioners have learned through incidents, but states it precisely and identifies what remains genuinely unsolved. Read against [[Security and Sandboxing]], the CAPS paper supplies the normative vocabulary that the containment literature lacks — sandboxes tell you what an agent *cannot* do; contextual security tries to say what it *should*. The unresolved tension is that the paper's central primitive, "appropriateness," is exactly the kind of fuzzy judgment that current enforcement mechanisms ([[CaMeL — Defeating Prompt Injections by Design]], [[The Agent Access Model]]) are built to avoid needing: CaMeL wins by making exfiltration impossible *by construction*, not by asking a model to judge social norms. The honest reading is that the two programs are complementary — CaMeL-style data-flow control as the floor, CI-derived policy as the layer above — but nobody has demonstrated the stack. The paper is also candid that the multi-agent portion is mostly open ground, which puts it years ahead of the fleet-orchestration sources in the wiki: while they optimize coordination mechanics, the genuinely unsolved problem is trust between agents that nobody vetted.

## Related pages

This source strengthens [[Security and Sandboxing]] by supplying the normative layer (what behavior is *appropriate*) that its containment-first sources deliberately bracket; it nuances [[The Agent Access Model]] by generalizing per-action least privilege into per-context appropriateness, complicating AAM's mechanical answer to a question AAM doesn't pose; it deepens [[CaMeL — Defeating Prompt Injections by Design]] by naming the class of attacks — contextually wrong but mechanically permitted flows — that pure data-flow control cannot catch; and it extends [[Zero Trust for AI Agents]] into the multi-agent ecosystem, where identity, accountability, and non-repudiation across agents remain, on this paper's own account, unsolved.

---
*Sources: [[raw/1047874-pdf]], [[summary/1047874-pdf]]*
*Last updated: 2026-10-18*
