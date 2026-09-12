# Who Owns the Code Claude Wrote

Sena Evren's concise legal field guide to the three unresolved IP questions hanging over agentic coding: whether AI-generated code is copyrightable, whether your employer already owns it regardless, and whether your codebase is contaminated by invisible GPL fragments from training data. The most useful thing written on AI code ownership to date — not because it resolves the questions, but because it maps exactly where the uncertainty lives and what you can do about it today.

---

## Key Quotes

> "Copyright only protects work created by a human."

The bedrock. Everything flows from this. The US Copyright Office hasn't budged, the DC Circuit upheld it in *Thaler*, and SCOTUS declined to hear the appeal in March 2026. But *Thaler* was the easy case — zero human involvement, AI listed as sole author. Agentic coding is the hard case: substantial human direction mixed with substantial AI generation, and no court has ruled on where the line is.

> "Probably yes for modules you substantially redirected, probably no for code you accepted verbatim, and unclear for everything in between."

Evren's honest summary of where "meaningful human authorship" stands for agentic workflows. This is the legal reality developers are operating in: a spectrum of risk, not a bright line. The practical implication is that *how you work with the agent determines whether you own the output*. Direction, rejection, restructuring — these are the evidence of authorship. Accepting output untouched is surrender.

> "If you are building something on the side, use a personal account, a personal machine, and tools you pay for yourself."

The work-for-hire section is the scariest part of the article. The San Francisco developer whose company claimed his personal fitness app because Claude had "access to open work files in the IDE" is a cautionary tale that should terrify anyone building side projects on company hardware. The legal argument is thin but the clause is broad, and the company has more lawyers than you do.

> "You have no way to know which side of the line your codebase is on without running a scan."

The open source contamination risk is the sleeper issue. AI models trained on GPL/LGPL code can reproduce substantial verbatim portions, and "I didn't know" is not a defense to copyleft violation. The same dynamic is playing out in music: [[Suno Training Data Breach|Suno's breach]] revealed training on copyrighted recordings from YouTube and Deezer, and the fair use argument that worked (so far) for code is being tested against a much more litigious rights-holder industry. The chardet dispute — Claude rewriting an LGPL library, developer rereleasing under MIT — is unresolved. M&A lawyers are already making license scans a standard due diligence condition. This isn't theoretical; it's showing up in acquisition contracts *now*.

> "The developer who documents creative contributions from the start is in a meaningfully different legal position than the one who accepted three thousand lines of Claude output and merged without review."

The article's core practical thesis. Documentation isn't bureaucracy — it's evidence of authorship. Commit messages describing *what you changed and why* (not just what was generated), design documents that predate the code, prompt logs showing deliberate redirection — these "may be protectable as human-authored expression even if the code they produced is not."

---

## Key Themes

- **#concept** — *Copyright and AI authorship*: The human authorship requirement is settled law; what counts as "meaningful" in an agentic workflow is not. The *Zarya of the Dawn* partial-protection model (human text protected, AI images not) is the best working precedent for code.
- **#concept** — *Work-for-hire in the AI era*: The doctrine doesn't care how code was generated — if you're an employee, your employer likely owns it. Broad IP clauses that cover "company-licensed tools" can reach your personal projects.
- **#concept** — *Open source license contamination*: Training data includes copyleft code. AI can reproduce it verbatim. You won't know unless you scan. Ignorance is not a defense. This is the risk most developers aren't thinking about.
- **#pattern** — *Documentation as authorship evidence*: Write commit messages that describe creative decisions (architecture, rejection, restructuring), not just output. Preserve prompt logs. Write design documents before generating code. These are your copyright insurance policy.
- **#pattern** — *License scanning as due diligence*: FOSSA, Snyk Open Source, Black Duck. If you're shipping commercial AI-assisted code without running one, you're operating on assumption.
- **#tool** — *Anthropic plan tiers matter for IP*: Consumer (free/Pro) plans have narrower indemnification; API/enterprise plans include IP indemnification. Neither covers downstream GPL violations from training data contamination.

---

## Critical Analysis

Evren has written the missing manual. Most writing about AI and code ownership is either legal academic hand-wringing (all edge cases, no practical advice) or tech-industry denial (pretending the questions don't exist). This piece does neither. It maps the legal landscape honestly — here's what's settled, here's what's emerging, here's what's pure speculation — and then gives developers four concrete things to do right now. That's rare.

The article's biggest contribution is reframing the question. "Who owns AI-generated code?" is the wrong framing because it assumes a binary answer. The right framing is: *what evidence of human authorship can you produce, and how much contamination risk are you carrying?* These are questions you can answer with documentation and scanning, not legal rulings.

The work-for-hire section is the most actionable and the most alarming. Evren doesn't resolve the legal question (she can't — no court has), but she correctly identifies that broad IP clauses in employment contracts are the immediate practical risk, not copyright doctrine. The advice to use personal machines and personal accounts for side projects isn't novel, but the specific mechanism — Claude having "access to open work files in the IDE" creating a derivative-work claim — is a genuinely new and underappreciated vector.

One limitation: the article focuses on US law. The EU's text and data mining exceptions, the UK's specific provision for computer-generated works, and other jurisdictions' approaches are not addressed. A developer shipping globally needs more than this.

Another: the open source contamination section is the most speculative part of the article (even Evren puts it in the "emerging consensus, no definitive rulings" category), but it's presented with the most urgency. The chardet dispute didn't resolve cleanly, *Doe v. GitHub* is still in progress, and no court has ruled on whether AI reproducing training-data patterns counts as verbatim copying. The risk is real, but the probability is unknown. That said, Evren's point that M&A lawyers are already treating this as a standard due diligence condition is a market signal worth taking seriously — the lawyers don't wait for courts to resolve uncertainty; they price it in.

The article's most provocative claim — that Anthropic's copyright over Claude Code itself may be invalid if the code was "predominantly written by Claude" — is more thought experiment than legal analysis. Anthropic almost certainly has enough human authorship in architecture, design, and curation to satisfy even a strict standard. But the fact that the question is askable, and that 8,000 DMCA takedowns were issued over code that may be AI-authored, exposes the tension at the heart of this space.

---

## Related Pages

- [[Agent Coding Workflow]] — The practitioner's daily loop; this article is about the legal infrastructure underneath that loop
- [[Claude Code Mastery]] — The tool whose legal status this article questions
- [[The End of Code Review]] — Monperrus on accepting AI output without review; this article is about the legal consequences of doing exactly that
- [[Writing Code vs. Shipping Code]] — The production gap between AI commits and actual releases; license contamination adds another filter to that gap
- [[The Founder's Playbook]] — Anthropic's startup playbook; founders building on AI-generated code need to understand these IP risks
- [[Specifications as the Product]] — If code is disposable, what's the copyrightable artifact? Evren's answer: the spec, the design doc, the commit history
- [[Guardrails and Feedback Loops]] — Deterministic enforcement over AI output; license scanning is the guardrail most teams haven't built yet
- [[The Cost YAGNI Was Never About]] — Beck on why cheap code generation doesn't retire YAGNI; this article adds "legal contamination risk" to the list of reasons
- [[Don't Trust the Label — License Laundering in AI Supply Chains]] — the empirical confirmation: Jewitt et al. measure what Evren warns about, finding Copyleft→Permissive collapse at 93.1% and Sharealike survival at 4.7% across 232,270 real AI supply chains
- [[Zero-Cost Fallacy of Open Source]] — the economic dimension beneath the legal question: even if IP ownership were settled, open source maintenance would still be structurally unfunded. Ford and Gall diagnose the licensing paradox (permissive enables exploitation, restrictive burdens maintainers with enforcement) that no legal clarity resolves

---
*Sources: [[summary/who-owns-the-code-claude-wrote]]*
*Last updated: 2026-07-05*
