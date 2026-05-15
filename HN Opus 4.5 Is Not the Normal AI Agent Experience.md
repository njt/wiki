# HN: Opus 4.5 Is Not the Normal AI Agent Experience

A 1,353-comment Hacker News thread that functions as an accidental focus group on the state of AI-assisted coding in early 2026. The OP built multiple working apps with Claude Code + Opus 4.5 without understanding how they were assembled. What follows is a Rorschach test: same model, radically different experiences depending on domain, language, and workflow maturity. The thread captures the moment when Opus 4.5 tipped a significant number of former skeptics into believers — and the patterns that distinguish success from frustration.

---

## Key Quotes

> "at least 99% all of the Rust code I've committed at work since Opus 4.5 came out has been from an agent. I basically don't write Rust code anymore. What I do instead is read a lot of Rust code."

— **rtfeldman** (Zed editor, ~1M lines of Rust, 150K+ active users). The most striking data point in the thread. Not vibe-coding — reading and judging agent output on a production codebase at scale.

> "It's a better *programmer* than I am, it's just not anywhere near as good a *software developer* for all of the higher and lower level concerns."

— **jaggederest**. The cleanest articulation of the programmer/developer distinction that runs through the thread. Agents can write functions; they can't decide which functions matter.

> "if your code is super easy to follow as a human, it will be super easy to follow for an LLM."

— **fullstackchris**. A hot take that's actually a design principle. Naming quality and structural clarity aren't just for human colleagues anymore — they're agent affordances.

> "Trying to one-shot large codebases is an exercise in futility."

— **kevin42**, who describes a 15-30 minute skeptic-conversion workflow: document architecture, create subagents per component, use planning mode. Everyone he's shown this to has responded positively.

> "2026 is going to be a wake-up call."

— **OldGreenYodaGPT**, who runs five scheduled workflow agents handling doc alignment, E2E coverage, and ticket triage alongside interactive coding agents. Describes this as infrastructure, not assistance.

> "Opus 4.5 really is at a new tier however. It just... works. The errors are far fewer and often very minor — 'careless' errors, not fundamental issues."

— **spaceman_2020**. The thread's central claim: Opus 4.5 crossed a threshold where errors went from architectural to clerical.

> "The agent has to have the tools to detect whatever it just created is producing errors."

— **theshrike79**, exasperated that people still paste code into ChatGPT and complain about hallucinations. The tools-in-a-loop insight that separates productive users from frustrated ones.

> "It would confidently insist that the code it wrote worked" but output was a blank screen.

— **ryandrake**, on trying to get Opus 4.5 to port OpenGL to SDL3_GPU. 3D graphics code is "really, REALLY bad" — a domain where visual verification is essential and agents can't see.

---

## Key Themes

### Compiler as Agent Guardrail #pattern

The thread's most actionable insight, repeated by a dozen commenters: **strongly typed languages dramatically improve agent output because the compiler catches mistakes at compile time, not runtime**. This creates a tight feedback loop that dynamic languages can't match without extensive test suites.

- gck1 switched from 15 years of Python to Rust for everything, even throwaway scripts: "Rust's compiler instantly screaming at claude when it goes off track"
- tezza: "types make it easier for agents to catch mistakes at compile time. javascript can still have ugly runtime WTFs"
- mikestorrent: strongly typed languages provide benefits "for 'free'" with AI — no longer need to invest hours in type design
- 348512469721: Rust with `cargo check` in a loop works. TypeScript's lenient compiler "lets it miss edge cases"
- gck1 wishes for "cargo-clippy for enforcing architectural patterns"

This is a concrete implementation of [[Feedback Loop is All You Need]] — the compiler is the ultimate linter, and it's already running.

### Training Data Proximity #concept

LLMs perform dramatically better on code that resembles their training distribution. This explains virtually all the success/failure variance in the thread:

- React/TypeScript: near-universal praise
- C#: "ubercharged" (ivm)
- Rust: strong results when combined with compiler loops
- C++ 3D graphics: "really, REALLY bad" (ryandrake)
- Novel protocols (MASQUE proxying): LLM "throws a fit" (JDye)
- Latest Swift: struggles "partially because of it being latest and partially because it's so convoluted" (ivm)

dpc_01234: LLMs are "fundamentally... non-deterministic pattern generator[s]." The fix for confusing similarly-named things? Rename them. The model is inference-based, not understanding-based.

### The Skeptic-Conversion Workflow #pattern

kevin42's reproducible skeptic-conversion recipe:
1. Create a CLAUDE.md for the app
2. Analyze libraries into architecture markdown
3. Create subagents referencing those docs
4. For bugs, use planning mode
5. Let Claude generate test cases and iterate

This maps cleanly to [[Harness Engineering]]: architecture docs as feedforward, compiler+tests as feedback. The prep work is the harness; the model is just the engine.

### Programmer vs. Engineer #concept

parliament32 draws on [Goedecke's pure/impure engineering distinction](https://www.seangoedecke.com/pure-and-impure-engineering/) to argue that LLMs produce "programmers" not "engineers" — they can implement known patterns but can't handle novelty. Counterpoints:

- emodendroket: most software engineering is applying known tools to non-groundbreaking problems
- woah (sarcastic): "Tell that to the guys drawing up the world's 10 millionth cable suspension bridge"
- loandbehold: "great at what 95% of software engineers do"

The gatekeeping is tiresome but the underlying observation is sound: agents fail on novel work. The question is how much of your work is actually novel.

### The Bespoke Software Thesis #concept

ryandrake's provocative claim: the 10% usage adage (people use 10% of a software's features) can now be inverted — each user builds their own 10%. He vibe-coded an Android TV video player in Kotlin (a language he doesn't know) without opening a source file. apitman: AI-built "bad code" that's smaller than bloated alternatives with tailored UX could be compelling.

This connects to [[Simplicity in the Age of AI-Assisted]]: LLMs make it cheap to rebuild without inherited complexity. But also to [[Cognitive Debt]] and [[AI Coding Tools Create More Bugs Than They Fix]]: ryandrake's "no memory leaks, crashes, ANRs" claim after one day of testing is exactly the overconfidence pattern.

### Context Management as the Real Skill #pattern

Multiple commenters converge on context management as the meta-skill:
- kevin42: subagents keep context clean; main agent delegates, gets short answers
- andai: Claude Code doesn't inject context by default — "you need to use 3rd party integrations for that"
- pluralmonad: watching Claude read a 500-line file in 100-line chunks "just makes me sad"
- HDThoreaun: /init "definitely" doesn't maximize code understanding because context quality degrades when filled

This validates the thesis of [[Agent Memory and Context]] and [[How Hightouch Built Their Long-Running Agent Harness]]: context management, not model ability, is the real engineering challenge.

---

## Critical Analysis

**This thread is better data than most AI benchmark papers.** It's 1,353 practitioners reporting grounded, domain-specific successes and failures against a single model (Opus 4.5) on real work. The pattern is clear enough to be actionable: if your domain has abundant public training data and your language has a strict compiler, you're probably having a great time. If you're doing novel low-level work in a niche domain, you're not.

**The "compiler as guardrail" finding is underappreciated and has design implications.** It suggests agent harnesses should include not just linters and tests but language-level constraints. A TypeScript project with `strict: true` and extensive ESLint rules might close some of the gap with Rust. The ideal agent language has a pedantic compiler, a fast type-checker, and good LSP support — which is why Rust users are so happy.

**The skeptic-conversion pattern is consistent and teachable.** Every convert in the thread followed the same arc: tried raw prompting, got frustrated, added architecture documentation, added subagents, added test loops, became productive. This isn't a model capability story — it's a workflow maturity story. The gap between "I pasted code into ChatGPT" and "I run five scheduled workflow agents" is the gap between frustration and transformation.

**Parliament32's gatekeeping is annoying but useful.** The "programmer vs. engineer" distinction captures something real: agents handle implementation but not judgment. But the gatekeeping itself — "you're not doing engineering" — is status-preservation dressed as intellectual rigor. The more interesting question is whether the "engineer" category shrinks as agents improve, or whether it shifts upward into architecture, specification, and verification — exactly what [[Specifications as the Product]] predicts.

**The bespoke software thesis is half-right.** Yes, individuals can now build their own tools. Yes, this threatens bloated software. But "I tested it for a day and found no leaks" is not the same as production-ready. The missing piece is verification infrastructure — the linters, tests, and monitoring that turn a demo into a product. This is where [[Compound Engineering]] and [[Guardrails and Feedback Loops]] become load-bearing.

**The thread reveals a workflow maturity gap, not a model capability gap.** The same model produces "99% of my committed Rust code" (rtfeldman) and "blank screen, insists it works" (ryandrake on 3D graphics). The difference isn't the model. It's the domain's training data density, the language's feedback loop tightness, and the user's harness sophistication. This maps to [[Agent Coding Workflow]]'s maturity spectrum: the thread's most productive users are at the compound engineering end.

---

## Related

- [[Agent Coding Workflow]] — the maturity spectrum this thread populates with real data points
- [[Vibe Coding and the Maker Movement]] — ryandrake's Kotlin app as evaluative anesthesia case study
- [[Guardrails and Feedback Loops]] — compiler-as-guardrail is the cleanest implementation
- [[Feedback Loop is All You Need]] — linters beat prompts; compilers are linters
- [[Compound Engineering]] — kevin42's skeptic-conversion workflow as compound loop
- [[Harness Engineering]] — architecture docs as feedforward, compiler/tests as feedback
- [[Designing Agentic Loops]] — "tools in a loop" as the meta-skill
- [[AI Coding Tools Create More Bugs Than They Fix]] — what happens without the feedback loops
- [[Slowing the Fuck Down]] — architecture-documentation-first as deliberate friction
- [[Cognitive Debt]] — what vibe coders accumulate in a day of testing
- [[Two Kinds of User Are Emerging]] — the power user vs. casual user split in the comments
- [[AI Zealotry]] — the skeptic-to-believer arc documented in real time
- [[How Boris Uses Claude Code]] — creator's workflow vs. community practice
- [[Writing a Good CLAUDE.md]] — architecture documentation as CLAUDE.md
- [[Radical Accountability]] — taste and specification quality as differentiators
- [[acceleration-flow]] — the dopamine loop of near-misses
- [[On a Year of Multi-Model Development]] — multi-model comparison data
- [[Simplicity in the Age of AI-Assisted]] — bespoke software thesis
- [[The Mythical Agent-Month]] — agents attack accidental complexity but generate new
- [[Scaling LLMs to Larger Codebases]] — Gill's oversight/guidance framework
- [[Talking to Transformers]] — domain language as compression
- [[Specifications as the Product]] — spec quality as the bottleneck
- [[Claude's System Prompt]] — context for understanding model behavior
- [[Prefix Effects]] — naming conventions affecting LLM performance

---
*Sources: [[raw/hn-opus-4-5-coding-agent-experience]]*
*Last updated: 2026-05-15*
