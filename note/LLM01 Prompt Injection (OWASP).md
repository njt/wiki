# LLM01 Prompt Injection (OWASP)

The OWASP Top 10 for LLM Applications' canonical entry on prompt injection: the definition (injection needs no human-readable form, only to be parsed by the model), the direct/indirect split, the injection-vs-jailbreak distinction, seven mitigation strategies, and nine worked attack scenarios. This is the shared vocabulary the rest of the field argues from — the page every practitioner survey, defense taxonomy, and incident postmortem tacitly assumes you have already read.

---

## What it argues

The entry's foundational move is to define the vulnerability by what the model parses, not what humans see: injected content "do not need to be human-visible/readable, as long as the content is parsed by the model." That single sentence licenses every later invisible-channel attack — adversarial suffixes, hidden image text, encoded payloads, terminal escape codes.

It splits the class in two. **Direct injections** are the user's own prompt derailing the model, whether malicious or accidental. **Indirect injections** arrive through external sources — websites, files, retrieved documents — and are the operationally dangerous half, because they let an attacker who never talks to the model steer it through content the model fetches. The impact list scales with agency: from disclosure of sensitive information and system-prompt leakage up through "executing arbitrary commands in connected systems" and "manipulating critical decision-making processes." A multimodal section warns that instructions hidden in images accompanying benign text create cross-modal attacks that "current techniques" struggle to detect.

The mitigation list is pragmatic rather than hopeful: constrain model behavior, validate output formats with deterministic code, filter input and output, enforce least privilege (the model's extensible functions handled "in code rather than providing them to the model"), require human approval for high-risk actions, segregate untrusted content, and red-team by treating the model as an untrusted user. And it concedes the ceiling upfront: "it is unclear if there are fool-proof methods of prevention for prompt injection."

## Key quotes

> "these inputs... do not need to be human-visible/readable, as long as the content is parsed by the model"

The definitional sentence that governs the whole field. The trust boundary runs wherever the model reads, not wherever a human reviewer looks. [[ANSI Escape Sequence Injection in MCP Servers]] is this sentence made concrete: ANSI bytes that render invisibly to people are consumed raw by models, so a terminal-heritage artifact becomes an injection primitive the moment the consumer changes. Every invisible-channel attack in the wiki is a footnote to this line.

> "While techniques like Retrieval Augmented Generation (RAG) and fine-tuning aim to make LLM outputs more relevant and accurate, research shows that they do not fully mitigate prompt injection vulnerabilities."

The entry's most load-bearing disclaimer, aimed at the most common false comfort in the field. Teams treat "we use RAG" or "it's fine-tuned" as a security posture; OWASP flatly says otherwise. [[Bounding the Blast Radius — Prompt Injection Defenses]] later quantifies exactly this: Zverev et al. (2025) proved no evaluated model separates instruction from data cleanly, and prompt engineering and fine-tuning don't substantially improve separation. The standards body knew in 2023 what the 2026 survey proved.

> "it is unclear if there are fool-proof methods of prevention for prompt injection"

Rare honesty from a security standard. Most vulnerability entries promise mitigations; this one opens its defense section with an admission that prevention may be impossible in principle, because the stochastic nature of generative AI has no deterministic boundary to patch. The wiki's subsequent literature agrees — the honest question shifted from "how do we prevent it" to "how do we bound the blast radius."

> "Provide the application with its own API tokens for extensible functionality, and handle these functions in code rather than providing them to the model."

The strongest single mitigation in the list, and the one that prefigures the whole environmental-defense turn. Moving function execution out of the model's hands and into deterministic code is exactly the brains-vs-hands separation that [[How We Contain Claude]] practices — where the agent reasons but execution happens in an environment that enforces properties "regardless of what the model decides to do." OWASP said "keep capabilities away from the model" before the industry learned to say "sandbox the hands."

## Key themes

#concept #security #prompt-injection #OWASP-Top-10 #reference

## Critical analysis

This is a standards-body entry, and it should be judged as one: its job is not novelty but a stable vocabulary and a defensible minimum. On that basis it succeeds. The direct/indirect distinction, the injection/jailbreak split, and the insistence that injected content need not be human-readable have become the field's shared ontology — nearly every source in this wiki's prompt-injection cluster argues *from* this entry rather than against it. Its nine scenarios are well chosen: the unintentional case (an applicant tripping a job posting's AI detector by optimizing their resume with an LLM) is a genuinely subtle inclusion, and payload splitting and adversarial suffixes point at attack classes that later research confirmed as central. The MITRE ATLAS mappings (AML.T0051, AML.T0054) give the entry a hook into established threat vocabularies.

But the entry has three characteristic weaknesses. First, the mitigation list is flat: seven items, no composition logic, no statement of which to do first or what residual risk remains after each. [[Bounding the Blast Radius — Prompt Injection Defenses]] is the necessary sequel precisely because it supplies the logic OWASP lacks — compose layers until adversarial cost-to-exploit exceeds value-at-risk. A checklist without a composition principle reads as "do everything," which teams predictably do partially. Second, the entry is silent on the *user as injection vector*: the case where the attacker is the legitimate user, typing the malicious prompt through a phished employee. [[How We Contain Claude]]'s February 2026 incident (24/25 successful exfiltrations) showed that no model-layer defense can catch this, because the attack is indistinguishable from authorized use — a threat shape this taxonomy simply doesn't name. Third, its scenario #2 (webpage summary → image insertion → conversation exfiltration) is the Greshake et al. indirect-injection result in miniature, but the entry dates from before the indirect-injection literature matured, so the scenario reads as hypothetical when it was in fact the opening move of a documented attack genre.

The deepest reading is that this entry is the field's fossil record of its own learning. Written as a vulnerability listing, it ends up documenting the moment before the field understood that prompt injection is not a bug but an architectural condition — that the missing code/data boundary is structural, and the response must be environmental (sandboxing, least privilege, deterministic validation) rather than rhetorical ("instruct the model to ignore attempts to modify core instructions," mitigation #1, is the weakest item on the list precisely because it asks the model to win the argument). Read alongside its successors, LLM01 is valuable as the baseline: everything the sharper sources say is a correction or a measurement of the terms this page fixed.

## Connections

[[Bounding the Blast Radius — Prompt Injection Defenses]] is the survey this entry's mitigation list grows up into: it keeps OWASP's concern but replaces the flat checklist with a four-layer taxonomy, the Nasr et al. (2025) adaptive-attack collapse, and an economic composition principle. Read LLM01 for the vocabulary, read Abdu for the strategy.

[[How We Contain Claude]] makes concrete the entry's least-privilege mitigation (#4) and exposes what the entry misses: Anthropic's incidents show that custom glue code, permission prompts approved 93% of the time, and the user-as-injection-vector case are where containment actually breaks — threats a taxonomy of prompt content cannot see because they live in the environment, not the text.

[[ANSI Escape Sequence Injection in MCP Servers]] is the entry's "not human-visible" clause extended to the byte level: Bright Security's AESI class demonstrates that the injection surface isn't just semantic content parsed by the model but raw bytes the model reads differently from any human renderer, including the stored variant that detonates on later reads — a persistence vector OWASP's scenarios don't contemplate.

[[Agentic AI Security Stack]] shows what this entry looks like when one kill chain is traced end to end: Lucktemberg's reference maps OWASP's classification onto MITRE ATLAS (the same framework LLM01 cites) and CSA MAESTRO, and its headline insight — an authenticated MCP server can still deliver poisoned tool descriptions — is the indirect-injection problem applied to the tool layer, one step beyond anything in the Top 10 entry.

---

*Sources: [[raw/llm01-prompt-injection]], [[summary/llm01-prompt-injection]]*
*Last updated: 2026-09-13*
