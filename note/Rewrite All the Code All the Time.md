# Rewrite All the Code All the Time

Adam's case that formal methods — not better natural-language AI — are the missing piece for a world where software is regenerated from specifications as routinely as we recompile today. The article connects the rising-abstraction history of programming to a specific claim: only mathematical rigour can eliminate the human oversight bottleneck that makes "rewrite all the code" economically infeasible.

---

## The Core Argument

The thesis has three moves:

**1. Code is becoming a throwaway artifact.** We should stop treating production code as scarce. The cost of regenerating significant codebases will drop to SaaS-subscription levels. Legacy code will steadily lose value as it becomes easier to replace automatically — code joins assembly language as a transient byproduct of the real long-lived artifacts.

**2. Natural language won't cut it.** The mainstream conversation assumes AI will generate programs from English. But natural language is "perversely hard": inherently ambiguous, possibly *designed* by evolution to be difficult to process (for its signalling role). EARS — the Easy Approach to Requirements Syntax — is the illustrative failure mode: `WHILE`/`WHEN`/`SHALL` keywords wrap freeform phrases, leaving ambiguity exactly where behaviour is decided. Tools that translate EARS-level specs into finer-grained specs create bottlenecks where "human attention must be applied" at every step. Regeneration needs to be push-button.

**3. Formal methods are the answer.** Only mathematically rigorous specifications enable *truly automatic* generation without human oversight. This is where Adam diverges from the mainstream: most people think "specs" means structured markdown or YAML. Adam means languages with clear formal semantics, where "any program meeting the specification will be acceptable."

> "The game is to capture requirements unambiguously enough that *any program meeting them will be acceptable*, allowing truly automatic regeneration of running systems after light-to-moderate requirements changes."

## Key Quotes

> "We need to stop thinking of production-ready code as a scarce resource. It may take a few years to get the tools up-to-snuff, but we'll reach a point where the cost of ongoing reimplementation of significant code bases drops to the levels associated with SaaS subscriptions today."

The economic claim in one sentence. This isn't "code is cheaper to write" — it's "code is cheap enough to *replace entirely on a schedule*." The SaaS comparison is deliberate: you don't own the code, you subscribe to its regeneration.

> "A specification gets written defensively, assuming the least-charitable reading by an implementer. In fact, the same specification, or its small evolutions, can be handed to a succession of adversarial implementers, which can, in theory, be an entirely safe way to maintain a program."

The two-contractor thought experiment. The spec writer's incentive is to close every loophole because the implementer is *adversarial* — they'll take any ambiguity as permission to do the cheapest thing. This inverts the usual dynamic where spec and implementation co-evolve inside one team.

> "The quote is from 1962! The subject is the introduction of the first high-level programming languages and compilers, which automate the tedious writing of code processed directly by computer hardware."

The historical pattern argument. In 1962, "automatic programming" meant compilers, and programmers were indignant about commoditization. Today we take high-level languages for granted and worry about AI generating *those*. The lesson: the level of abstraction constantly rises, and today's "real programming" is tomorrow's compilation target. David Parnas made this observation in the 1980s.

> "Natural language-based spec formats strike me as that kind of hybrid beast that we'd be better-off replacing with a proper, unambiguous language."

The flowchart analogy. Flowcharts were an intermediate form with unclear semantics, replaced by high-level languages. EARS and similar formats are today's flowcharts — a half-step toward formal specification that we should skip past.

## The Reader's Extensions

The article includes a substantial comment from a reader (credited as the postscript) that adds two important dimensions:

**1. The evidence artifact.** The two-contractor model produces a spec and an implementation, but there's a missing third artifact: "the record of which claims were checked, against which version of the spec, under which assumptions, with which exclusions named." If the scope of proof has to be re-established by hand each cycle, some of the cost migrates rather than disappears. The deployed system and the toolchain sit slightly outside the proof's binding.

**2. Version control for specs.** Git carries more than storage — commits, blame, licences, the working answer to whose work counts. Specs are collectively authored and continuously evolved, and "asking of a spec what git answers about code — who contributed this constraint and what it is worth — has no obvious mechanism yet." The reader's conclusion: the durable artifact is the specification plus the evidence that scopes it, plus the record of why alternatives were rejected.

## Key Themes

#formal-methods #spec-driven #code-generation #natural-language #economics #abstraction #formal-verification #EARS

## Critical Analysis

**This is the strongest version of the formal-methods-as-economic-necessity argument yet.** Where [[The Coming Need for Formal Specification]] diagnoses the bottleneck and prescribes formal methods, Adam adds the historical pattern, the economic mechanism, and the thought experiment that makes the argument concrete. The two-contractor model is the freshest idea here — it gives you a way to *test* whether a spec is good enough (hand it to an adversarial implementer) rather than just asserting that it should be.

**The EARS critique is devastating and underappreciated.** The spec-driven development tools being built today — the SDD pipelines, the YAML acceptance criteria, the PRD-to-code generators — all share EARS's fundamental problem: structured keywords around unstructured natural language. The keywords are unambiguous; the phrases they govern are not. Adam's point is that this isn't a bug to be fixed with better prompting — it's a category error. Natural language can't do the job, period. This is a stronger claim than [[AI Agents Need Clear Specs]] makes, and it puts Adam in tension with the entire spec-as-structured-markdown tradition.

**Where the argument is weakest: the gap between "formal methods" as a concept and as a practice.** Adam says we need "languages with clear formal semantics" but doesn't name one. The article gestures at "reengineering of the stack for high-performance inference to work natively with formal logic instead of just linear algebra" — which is a hardware/compiler research program, not a tool you can adopt. [[We Have Proof Automation Now]] shows one path (Lean + LLM-generated proofs) but with a 10× performance penalty. [[Why Rocq Is Better Than Lean for Program Verification]] argues the ecosystem question is at least as hard as the generation question. Adam's vision requires solving all of these simultaneously.

**The economics are directional but underspecified.** "SaaS subscription levels" is evocative but vague. A SaaS subscription is $10–$500/month. Regenerating a significant codebase involves inference costs, verification costs, and the human cost of validating the new behaviour. The reader's extension about migrating costs is sharp: even if generation is free, *re-establishing the scope of proof* is not. Adam acknowledges the tools need work but doesn't estimate the gap.

**The historical pattern is the argument's strongest pillar.** The flowchart → high-level language → formal spec progression is genuinely compelling. Every generation thinks *their* abstraction level is the natural one and the next level up is "not real programming." Adam is asking us to see ourselves as the 1962 programmers who thought compilers were a threat to their craft. The question is whether the jump from high-level languages to formal specs is the same *kind* of jump as assembly to high-level languages, or a fundamentally different one. Assembly → C automated *translation*; C → formal spec automates *design*. Those are different cognitive acts.

**What this means for the wiki's existing spec-as-product thesis:** Adam's argument sharpens [[Specifications as the Product]] by adding a terminal destination. The spec-as-product thesis says specs outlive code. Adam says the spec had better be formal, because anything less leaves a human in the loop, and a human in the loop makes "rewrite all the code, all the time" economically dead. He's raising the bar on what "spec" means — and implicitly arguing that most of what we currently call specs (markdown PRDs, YAML acceptance criteria, BDD scenarios) are EARS-like hybrids that won't survive the transition.

The reader's extensions add something genuinely new to the conversation: the evidence artifact and spec version control. These aren't objections to Adam's thesis — they're the next two problems to solve if the thesis is right. [[The Knowledge Chipper]]'s diagnosis of context loss between sessions rhymes with the evidence problem: if the record of what was checked doesn't survive, you're back to re-deriving trust. [[DeltaDB]]'s bidirectional linking of code to conversations is the closest existing implementation of what spec version control would need to become.

---

*Sources: [[raw/rewrite-all-the-code-all-the-time]], [[summary/rewrite-all-the-code-all-the-time]]*
*Last updated: 2026-08-07*
