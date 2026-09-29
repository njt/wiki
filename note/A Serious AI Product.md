# A Serious AI Product

Glyph Lefkowitz's specification-by-negation for AI tools: every chatbot's own disclaimer admits it makes mistakes, so a product that takes problem-solving seriously would be built around verification — claim checkboxes, citations as primary artifacts, provenance labels, context visibility, a real sandbox — and the fact that none of these exist after years and hundreds of billions in funding suggests the labs are hiding what honest measurement would reveal.

---

Glyph's framing is deliberately product-design, not model-capability: "this is not what a problem-solving tool would look like." Most of his demands assume the model stays exactly as flawed as it is; the harness must compensate. Only the anti-verbosity and no-apologies sections assume labs can shape model behaviour — and he notes their tight benchmark control suggests they can.

## Key Quotes

> If your product tells me that it makes mistakes and I must be the one to check for the mistakes, but then gives me *zero tools* to check for mistakes, I cannot take it seriously.

The thesis in one line. The fine-print disclaimers ("ChatGPT can make mistakes. Check important info.") are legalese that shifts liability, not UI — and "how am I supposed to know what 'info' is supposed to be 'important'?" is the killer question.

> Every result should be presented as a *list of citations*... the literal, unmodified quotation... should be front-and-center, larger than any AI-generated text.

An inversion of the current presentation hierarchy: machine output demoted to de-emphasised annotation under the human source, until the user verifies the summary. Note his sharp distinction that quotations must be "extracted with a regular program and not an LLM."

> Every frontier lab has tied a spring-loaded shotgun to a dog; the fact that dog owners can publish thoughtful blog posts explaining how you can teach your dog the basics of gun safety... does not mitigate the fact that the product should not have been allowed in the first place.

The essay's most quotable indictment of agentic coding's unsafe-by-default posture. The "best practices" literature — approval gateways, containers, prompt-based pleas not to edit tests — is read as users building "incomplete and error-prone security perimeters of our own design" so the vendor can blame operator error.

> With nothing between your personal vigilance and disaster, there are no workflows left beyond decrementing your own vigilance until there's nothing left.

A precise description of approval fatigue as a designed outcome: Y, Y, Y, "yes to all", full-auto, void.

> I think the null hypothesis is that AI tools provide, in aggregate, zero value.

The closing move from product critique to falsifiable claim. He reports that nobody has ever come back with a measured positive cost/benefit ratio from his earlier methodology — absence of evidence acknowledged, but framed as a test the labs could pass and conspicuously avoid by refusing to ship measurement tools.

## Key Themes

- #concept **Verification as interface** — checking is a mandatory workflow step, so it deserves first-class UI, not a disclaimer.
- #pattern **Harness over model** — most fixes (provenance, sandboxing, context visibility, plan review) live in the harness and would work even with unimproved models.
- #concept **Vigilance decrement** — AI output is usually right, which is exactly why human checking degrades; hence aviation-style rest rotations and periodic independent spot-checks.
- #tool **Organisational dosimetry** — usage meters, skill-practice budgets (the dockworker/crane analogy), and mental-health provisions as preconditions for deploying a hazardous tool.

## Opinion

This is the strongest articulation I've seen of the "the disclaimer is the tell" argument, because it converts a vibe into a feature list a vendor could actually ship — which makes the silence around it damning in a way pure scepticism isn't. The weakest point is the null-hypothesis leap: "I've heard from nobody who measured well" is genuinely weak evidence, and Glyph half-admits it. But the surrounding argument that labs avoid *building* measurement surfaces is structurally sound and unfalsifiable only because the labs keep it that way. The organisational section is the sleeper hit: rest rotations and deliberate practice are concrete, precedented (aviation), and almost never discussed in AI-adoption literature.

## Related Pages

- [[The Agentic Product Standard v2.0]] — offers a constructive, field-tested counterpart to Glyph's demand list; where Glyph specifies what a serious product owes the user, the Standard specifies what a serious deployment owes itself. They agree on batch plan approval and harness-level safety.
- [[Ways of Checking]] — the verification-failure-mode catalogue this essay implies but doesn't enumerate; Glyph's claim-checkbox UI is one concrete remedy for exactly those failure modes.
- [[The End of Code Review]] — Glyph's point that coding tools push verification onto code review as a dark pattern directly extends this essay's argument that review is becoming the dumping ground for unverified agent output.
- [[Zero Trust for AI Agents]] — complements the sandbox section: both argue security must live in the harness independent of the prompt, not in user vigilance or prompt-based pleas.

---
*Sources: [[raw/serious-ai-product-html]], [[summary/serious-ai-product-html]]*
*Last updated: 2026-09-29*
