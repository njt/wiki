# HN: Don't Fall Into the Anti-AI Hype

A 1,631-comment HN thread that functions as an accidental debate about what LLMs actually are and what they're actually good for. The trigger was antirez's essay pushing back against AI skepticism, but the comment section is the real product: practitioners across domains reporting grounded successes and failures, with the most original contribution being friendzis's entropy/convergence framework — LLM prompting converges slower than traditional fix cycles because LLM output is fundamentally less analyzable, making the technology work best in low-entropy codebases.

---

## Key Quotes

> "Works on a 15-year-old Java Spring + React + Thymeleaf app... I have consistently rewritten about 70% of AI generated code... either I am terrible at prompting, or everyone else is far less thorough in code review."

— **totallykvothe**, who has tested every SOTA model from GPT-3.5 through "GPT-5.1 Codex Max and Opus 4.5." The thread's most important humility check: when someone with 15 years of experience and every model available reports 70% rewrite rates, the "you're just bad at prompting" dismissal collapses.

> "LLMs are fundamentally... super search engines with the capability to aggregate and filter results into concise answers. The problem is, when problems become complex, LLMs... give you *all* the answers, not the *best* one."

— **unyttigfjelltol**. The thread's cleanest articulation of the search-engine position. LLMs surface relevant information but can't discriminate quality — a librarian who retrieves every book on the shelf but can't tell you which one is right.

> "Every single technical question I know the actual answer to, the LLM will be substantially wrong in some fashion. What's worse is that I've noticed it's actually gotten worse with the newer 'thinking' models... they are more confidently incorrect now."

— **20k**, an astrophysicist working in numerical relativity. Provides a detailed, technically-specific breakdown of ChatGPT Thinking mode's failures on ADM and BSSN formalism questions — including "timecube levels of nonsense" and the invented phrase "manifestly Hamiltonian." The most devastating domain-specific critique in the thread.

> "'LLM prompting' is on average much less analyzable (if at all) than 'code', so the prompt-fix cycle falls somewhere between 'does not converge' and 'convergence tail is much longer.'"

— **friendzis**, introducing the thread's most original framework. Traditional development converges because code is analyzable and fixes are cheaper than writing. LLM output lacks that analyzability, so the fix cycle drags. Works better in greenfield and well-structured codebases — low existing entropy.

> "The hard problem has shifted from prompting to context engineering."

— **0xf8**, naming the skill that separates productive AI users from frustrated ones. Since reasoning-class models emerged, the bottleneck isn't what you ask but what context you provide. Speculates that optimal context engineering may converge to what it already means to be a competent software engineer — which undermines the productivity case.

> "A search engine with a lossy-compressed dataset of most public human knowledge that returns results in natural language. That is a very useful thing! But certainly NO! it is not intelligent."

— **IAmGraydon**, the thread's most quotable skeptic. The lossy-compression framing captures both the value and the limitation: useful retrieval of compressed knowledge, not reasoning.

> "LLMs are not just search engines, they're so much more... they have compressed the knowledge... learned the relations between all kinds of different levels of abstractions and meta patterns."

— **XenophileJKO**, the strongest pushback against the search-engine framing. Compression that preserves relational structure across abstraction levels is doing something qualitatively different from search.

> "I don't blurt out different answers to the same question using different phrasing, I doubt any human does."

— **croon**, identifying what may be the irreducible difference between LLMs and human cognition. LLMs randomly fail on fundamental questions if token prediction veers off. Despite daily use, "there is absolutely 0 illusion of intelligence for me."

> "If the business can get rid of their engineers, then why can't the user get rid of the business providing the software?"

— **sublinear**, the most elegant rebuttal to daxfohl's "engineering team replacement" prediction. If AI makes software creation costless for companies, it makes it costless for customers too — the business dissolves from both ends.

> "Traditional code development is 'write code → QA → fix' which converges because code is analyzable and fixes are cheaper than writing. LLM prompting is on average much less analyzable (if at all)."

— **friendzis**, restated because it deserves emphasis. This is the thread's most actionable theoretical contribution. It explains why [[Compound Engineering]] works: each added system (tests, linters, types) increases analyzability, tightening the convergence loop.

---

## Key Themes

### Entropy and Convergence #concept

friendzis's framework is the thread's headline contribution. LLM-assisted development has different convergence properties than traditional development because LLM output is less analyzable than hand-written code. The fix cycle is longer and less reliable. This means:

- LLMs work best in **low-entropy** environments: greenfield projects, well-structured codebases, strongly-typed languages with strict compilers
- LLMs struggle in **high-entropy** environments: legacy codebases, complex business logic, niche domains with sparse training data
- The solution is to **reduce entropy first**: architecture documentation, type systems, test suites — everything [[A Practical Guide to Brownfield AI Development]] prescribes

This framework explains the entire success/failure variance in the thread without needing to invoke model capability differences.

### Lossy Compression vs. Learned Relations #concept

The thread's central philosophical divide: IAmGraydon's "search engine with lossy-compressed dataset" vs. XenophileJKO's "learned relations across abstraction levels." Both are partially right — LLMs do retrieve and recombine training data patterns, but the recombination across abstraction boundaries produces genuinely novel outputs. The "swimming spaceship" example (ffwd): a concept not in the training data, assembled from probability-based chunking.

omnimus settles the practical question: the intelligence debate is marketing for AI CEOs. The real question is usefulness, and "advanced search engine" is plenty useful — it's just not what the companies want to sell.

### Domain Variance #pattern

The same models produce radically different results depending on domain:
- **React/TypeScript**: near-universal praise
- **Numerical relativity**: "timecube levels of nonsense" (20k)
- **Terraform**: invents non-existent internal functions (dividedbyzero, JohnMakin)
- **Architecture review with Claude skills**: "there's only one way to do things" (richardw)

This variance is not random. It follows training-data density: domains with abundant public code and documentation produce good results; niche domains with sparse training data produce confident hallucinations. This is the same pattern documented in [[HN Opus 4.5 Is Not the Normal AI Agent Experience]], applied to an even broader set of domains.

### Context Engineering #pattern

0xf8 names the emerging meta-skill: context engineering. Since reasoning-class models emerged, the bottleneck has shifted from "how do I prompt this?" to "what context does this need?" richardw operationalizes this with Claude skills — repeatable context templates for architecture review, SOLID enforcement, performance analysis, etc.

The implication is uncomfortable: if optimal context engineering converges to what it already means to be a competent software engineer — understanding architecture, constraints, and tradeoffs well enough to specify them — then the productivity multiplier may be smaller than enthusiasts claim.

### Verification Burden #concept

The thread's most practical tension: LLMs can generate code faster than humans can verify it. solid_fuel: "If you have to verify every test as an expert, I might as well code the solution myself." simonw's counter-workflow: spec.md → approval → red/green TDD — make verification systematic and the LLM does the implementation. This is [[Spec-Driven Development]] operationalized.

eru makes the key observation: software is actually the best domain for LLMs because "you can easily check whether it builds and passes tests." The verification problem is solvable in software in a way it isn't in other domains. This is why coding agents have advanced faster than general-purpose agents.

### The Testing Paradox #pattern

thesz drops the thread's most sobering data point: SQLite has a 590:1 test-to-code ratio and *still* finds bugs in point releases. If the most-tested software in existence has bugs, what confidence can you have in LLM-generated code that passes a test suite? mohaine's corollary: "Most of the problem in programming is writing the tests. Once you know what you need the rest is just typing."

DrammBA names the deeper risk: LLMs "hallucinate reasonable but unintended specs." If the implementation matches a wrong spec, tests pass and "everything 'passes' and is still wrong."

### Monolith Revival and Team Restructuring #concept

daxfohl predicts AI will drive a return to monolithic architectures — microservices exist for human team coordination, and AI benefits from a single codebase. catlifeonmars counters: small, scoped repos keep agents focused and reduce blast radius. Both could be right at different scales.

matwood: the engineering team of the future may be "mostly product, spec writers, and testers" — the composition changes more than the headcount. concats: there will always be a "transitioning layer from humans to AIs" — someone with high technical competence to convert "CEO fever dreams" into strict specs.

### Historical Perspective #pattern

eloisant: when 3GL languages and Visual Basic emerged, bosses claimed anyone could code. 40 years of productivity gains led to *more* developer demand, not less — needs grew faster than efficiency. This is the strongest counter to the "engineering team replacement" prediction, and it's backed by actual history rather than speculation.

---

## Critical Analysis

**This thread is better empirical data than most AI benchmark papers.** 1,631 practitioners reporting grounded, domain-specific successes and failures against real work. The pattern is consistent enough to be actionable: success depends on domain training-data density + codebase entropy + verification infrastructure. None of these are model properties — they're environmental properties.

**The "LLMs are just search engines" framing is reductive but rhetorically necessary.** It's technically wrong — compression that preserves cross-abstraction relations is doing something more than retrieval — but it serves a crucial function: cutting through AI CEO marketing that insists on describing autocomplete as proto-consciousness. The framing is a negotiating position, not a technical claim. [[A Non-Anthropomorphized View of LLMs]] makes a more precise version of the same argument from a mathematical direction.

**friendzis's entropy/convergence framework should be as widely cited as anything in the thread.** It's the rare HN comment that introduces a genuinely new analytical lens. "LLM prompting is less analyzable than code, so the fix cycle converges slower" is a testable claim with direct engineering implications. It predicts that [[Compound Engineering]] — adding systems that increase analyzability (types, tests, linters, formal specs) — is the correct response. It also predicts that [[A Practical Guide to Brownfield AI Development]]'s "reduce entropy first" strategy is correct. The framework explains why [[Feedback Loop is All You Need]] works at a theoretical level, not just an empirical one.

**20k's physics critique is the thread's most important cautionary tale.** Every AI enthusiast who dismisses skeptics as "bad at prompting" should read the ADM/BSSN breakdown. Here is a domain expert testing the best available models on questions they know the answers to, getting "timecube levels of nonsense," and reporting that newer models have gotten *worse* — more confidently incorrect. This isn't a prompting failure. It's a fundamental limitation: in domains with sparse training data, LLMs don't degrade gracefully; they fabricate with increasing confidence.

**The intelligence debate is a waste of everyone's time.** omnimus gets it right: it's marketing for AI CEOs. Whether you call it "intelligence," "lossy compression," or "learned cross-abstraction relations," the engineering question is the same: under what conditions is this tool net productivity positive? The thread's answer: when domain training data is abundant, codebase entropy is low, and verification is systematic. That's actionable. The rest is philosophy.

**The verification paradox is real and underappreciated.** SQLite's 590:1 test ratio finding bugs in point releases proves that "passes tests" is not the same as "correct." The LLM workflow that most enthusiasts describe — generate, test, fix, repeat — converges to whatever the tests check, not to correctness. The spec quality becomes the ceiling. This is the strongest argument for [[Spec-Driven Development]] and [[Specifications as the Product]]: if the spec is wrong, everything downstream is wrong, and LLMs will confidently implement wrong specs as readily as right ones.

**daxfohl's "engineering team replacement within ~2 years" is the thread's least credible claim** and the one that drew the most pushback, for good reason. eloisant's historical parallel (3GLs and VB didn't eliminate programmers) and sublinear's symmetry argument (if companies can replace engineers, customers can replace companies) are independently fatal. But the subtler point from OkayPhysicist deserves attention: AI may be "bad for salaries" because it destroys the "looks difficult" moat. Software development is actually hard, but looks easy with AI — historically, that combination hurts bargaining power.

---

## Related

- [[HN Opus 4.5 Is Not the Normal AI Agent Experience]] — the companion HN thread from the same era; compiler-as-guardrail and training-data proximity as the other half of the story
- [[A Non-Anthropomorphized View of LLMs]] — Flake's mathematical "functions through R^n" framing as the precise version of the lossy-compression argument
- [[Agent Coding Workflow]] — the maturity spectrum from vibes to compound engineering that this thread populates with data points
- [[Compound Engineering]] — friendzis's entropy framework provides the theoretical justification for adding systems rather than manual review
- [[A Practical Guide to Brownfield AI Development]] — "reduce entropy first" as the operational version of friendzis's theory
- [[Feedback Loop is All You Need]] — linters beat prompts; compiler-as-guardrail closes the convergence loop
- [[Spec-Driven Development]] — simonw's spec.md → TDD workflow as the answer to the verification paradox
- [[Specifications as the Product]] — if LLMs implement wrong specs as readily as right ones, spec quality is everything
- [[Guardrails and Feedback Loops]] — the engineering response to the analyzability problem
- [[Harness Engineering]] — context engineering as feedforward, verification as feedback
- [[Designing Agentic Loops]] — simonw names the meta-skill: choosing tools and guardrails so YOLO-mode agents converge
- [[Addy Osmani's Workflow]] — start with spec.md, work in focused chunks, review like a senior engineer
- [[How to Write a Good Spec for Agents]] — the spec as the ceiling on correctness
- [[Scaling LLMs to Larger Codebases]] — Gill's feedforward/feedback framework for where to invest engineering effort
- [[Vibe Coding and the Maker Movement]] — "evaluative anesthesia" as the risk when verification is absent
- [[Cognitive Debt]] — what accumulates when the convergence loop is loose
- [[AI Coding Tools Create More Bugs Than They Fix]] — what happens without systematic verification
- [[The Mythical Agent-Month]] — agents attack accidental complexity but generate new accidental complexity
- [[Slowing the Fuck Down]] — deliberate friction as a feature when convergence is slow
- [[Radical Accountability]] — taste and specification quality as the remaining differentiators
- [[Write Only Code]] — when convergence fails, slop accumulates
- [[Code Field]] — resist the urge to over-specify; the counterpoint to spec-maximalism
- [[TextForge Case Study]] — six-layer discipline for greenfield LLM development with verification at every layer
- [[Grok 4.3 (HN Discussion)]] — another HN thread as accidental focus group
- [[Smart Models Dumb Pipes]] — LLMs as judgment machines in the end-to-end principle applied to AI
- [[AGI Is Here (Robin Sloan)]] — Sloan's declaration that AGI arrived with GPT-3; the opposite end of the spectrum from IAmGraydon
- [[Talking to Transformers]] — domain language as compression; the mathematical frame in practice
- [[Claude's System Prompt]] — understanding what's actually being managed behind the text generation
- [[Two Kinds of User Are Emerging]] — the power user vs. casual user split visible in the comments
- [[The Next Two Years of Software Engineering]] — junior employment declining while senior roles hold; the labor-market version of the thread's concerns
- [[The Future of Everything is Lies I Guess]] — Kingsbury's catalogue of LLM harms; the dark side of the lossy-compression model
- [[Cyborgs Will Kill the Corporation]] — sublinear's symmetry argument taken to its logical conclusion

---
*Sources: [[raw/hn-dont-fall-into-anti-ai-hype]]*
*Last updated: 2026-05-15*
