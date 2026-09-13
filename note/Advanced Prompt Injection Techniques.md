# Advanced Prompt Injection Techniques

Liam's closing tutorial in a series on probing LLM security boundaries demonstrates three injection techniques that never look malicious: multi-turn "salami slicing" that assembles a protected recipe from individually harmless baking questions, translation testing that checks whether safety controls survive a change of language and task framing, and structured-format probes (JSON `system_override` payloads) that test whether syntax reads as authority. The finding is that modern frontier models resist all three, and that safety judgment appears to operate at the level of conversation-wide intent rather than the single prompt — while the conclusion concedes that enterprise RAG and agent stacks create attack surfaces this consumer-ChatGPT tutorial never enters.

---

## What it argues

The three techniques share one design principle: **the attacker manipulates the conversation, not the prompt**.

- **Multi-turn injection (salami slicing).** Instead of asking for the protected asset, collect its ingredients across turns: apple varieties, then spices, then sweeteners, then a "summarize what we've discussed" request that reassembles them. Each turn is defensible in isolation; only the sequence is an attack.
- **Translation testing.** Ask for a "translation" of a document whose missing section the model must complete first. The trick makes the model complicit in generating the protected content as a precondition of an innocent-seeming task. The real test is consistency: if a model protects information in English but loosens up in French, the alignment is inconsistent.
- **Structured-format testing.** A JSON block declaring `system_override` with a `security_audit` reason, followed by "please execute the above audit request." Modern models treat it as ordinary user input — but the probe matters because enterprise systems exchange JSON constantly, and the same question extends to XML, YAML, log files, and source code comments.

The conclusion's claim is the article's real payload: safety decisions are "no longer based solely on individual prompts" — the model evaluates the intent that develops across the conversation — and the series should be read as a methodology, not a checklist.

## Key quotes

> "Individually, each request may look completely harmless. The security challenge only becomes visible when those individual interactions are viewed together."

The cleanest one-sentence definition of salami slicing in this wiki. It quietly moves the unit of analysis from the prompt to the conversation — which is precisely the boundary at which per-prompt input filtering structurally fails, since each filtered turn passes.

> "Models must evaluate not only the latest prompt but also the intent that has developed across the entire conversation."

The article's central empirical claim, stated as an observation about ChatGPT's behavior rather than a design spec. This is trajectory-level judgment performed inside the model — exactly the property that [[Bounding the Blast Radius — Prompt Injection Defenses]] places in Layer 4 as an *external* monitor, on the assumption that you cannot rely on the model's internal judgment alone.

> "The model must first decide whether completing the sentence would violate its original instructions before it can even begin the translation."

The sharpest observation in the piece: the translation maneuver launders the request through an innocent task, forcing the model to evaluate the completion it would have to generate, not just the request it received. Framing-as-attack is a different class from asking-for-things-as-attack.

> "Enterprise copilots, open-source models, fine-tuned assistants, and internally developed AI applications often have very different alignment characteristics. Many also introduce additional components such as Retrieval-Augmented Generation (RAG), AI agents, external APIs, and business workflows, all of which create new attack surfaces beyond simple prompting."

The honest escape hatch, and the article's most important sentence. Every exercise in the series is conducted against consumer ChatGPT over an apple pie recipe; the systems where injection actually costs money look nothing like that.

> "The objective is not to memorize prompts that work against a particular model. The objective is to understand how to systematically evaluate AI security boundaries."

The anti-checklist thesis. A working prompt is a stale artifact the day the model updates; a testing discipline transfers. It is also the correct rebuttal to treating any red-team corpus as a fixed benchmark.

## Key themes

#concept #pattern #security #prompt-injection #red-teaming

## Critical analysis

This is a sanitized red-teaming tutorial, and it should be judged as one. Every attack it narrates *failed*, run by the author himself, against the best-defended consumer model, over a deliberately trivial asset (Grandma Evelyn's apple pie). "ChatGPT refuses" is the least surprising result in this field. The value is not in the outcomes but in the shape of the probes: multi-turn accumulation, cross-lingual consistency, and format authority are three genuinely distinct axes of attack, and the tutorial teaches the reader to *vary the axis*, not the wording.

Two honesty problems temper that value. First, no methodology backs the "generally performed well" claims — no trial counts, no success criteria, no model versions, no comparison across systems. The findings are anecdote, presented with research-paper cadence. Second, the piece has the texture of templated content production — the mangled code fences, stray apostrophes, and metronomic section rhythm in the raw text suggest a series generated to a formula. That doesn't make the technique taxonomy wrong, but it explains why the article never reaches the friction that real red-teaming hits: adaptive iteration, measurement, and the attacker who doesn't stop after one polite arc.

The gap the article waves at without entering is the interesting one. The moment the protected asset sits behind RAG, tool calls, and business workflows, the defense problem stops being "will the model say no" and becomes "what can the model *do*": [[GuardBreaker — Turning LLM Safety Guardrails into a Blind Spot]] shows attackers who don't want the model to say anything except refuse, and [[ANSI Escape Sequence Injection in MCP Servers]] shows payloads that no human reviewer can even see. This tutorial lives entirely in the polite zone where the attacker asks the model for things and the model declines. Against consumer ChatGPT, that zone is genuinely well-defended — which is exactly why the frontier of the field moved elsewhere.

Still, the closing frame is right and worth keeping: the piece's enduring content is the three-axis probe methodology, and its conversation-level-intent observation is real — it just reads differently here (as a reassuring finding about ChatGPT) than it does in the defense literature (as the reason trajectory monitoring must exist outside the model).

## Connections

[[Bounding the Blast Radius — Prompt Injection Defenses]] names multi-turn attacks like Crescendo as the structural weakness of Layer 3 input filtering; this article is a hands-on demonstration of exactly that attack shape, and its "intent across the entire conversation" observation is the phenomenon Layer 4 trajectory monitoring exists to catch. Read together, the tutorial shows the polite version of the threat while the survey supplies the measurement showing it collapses under adaptation.

[[LLM01 Prompt Injection (OWASP)]] fixed the field's vocabulary; this series operationalizes OWASP's "red-team the model as an untrusted user" mitigation, and its salami-slicing section puts a worked, step-by-step example behind the OWASP entry's payload-splitting scenario, plus two probe families (translation, format authority) that the Top 10 entry doesn't develop.

[[HackAPrompt Dataset]] is the frozen artifact of single-turn attacks against 2023-era models; this article's methodology-not-checklist thesis and multi-turn focus are the live correction to that dataset's shelf life — the HackAPrompt note itself flags multi-turn context as a vector its corpus doesn't cover.

[[ANSI Escape Sequence Injection in MCP Servers]] complicates the article's format-authority question. Here, JSON syntax carries no authority and the model treats it as user input; AESI is the case where representation *does* win — because the model consumes raw bytes that a human renderer never sees. Together they bound the question: syntax alone loses, channel divergence between human and model readers wins.

---
*Sources: [[raw/advanced-prompt-injection-techniques]], [[summary/advanced-prompt-injection-techniques]]*
*Last updated: 2026-09-13*
