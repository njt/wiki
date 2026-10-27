# We Should Be More Ambitious (Dan Luu)

Dan Luu's essay argues that coding agents have quietly broken the cost model programmers carry in their heads: whole classes of project — a Postgres rewrite in Rust, a custom regex engine, a Rust interpreter for faster builds, a superhuman board-game AI — that would once have taken a year or more now take hours of human time. The problem is no longer capability but imagination: people still implicitly budget with pre-agentic costs, so they never attempt the projects that are now feasible. The counterweight, voiced by Yossi Kreinin in an appendix, is that most people should be *less* ambitious, since they lack clear pass/fail criteria and can't reason about black boxes — leaving the essay's real thesis somewhere in between: ambition is safe exactly where you can build self-checking verification.

---

## Key Quotes

> "There is, understandably, a lot of skepticism about projects like pgrust because there are so many low-quality AI-generated projects out there... if you look into it a bit, you'll see that in testing pgrust, he's testing thoroughly enough that he's even uncovering real bugs in Postgres."

The strongest move in the piece: Malis's pgrust is the perfect rebuttal to the vibeslop reflex, because its test regime has found real bugs in Postgres itself (SP-GiST numerics, JSONPath format cache, pgcrypto, memory safety in the dictionary loader). Even if pgrust dies, the attempt durably improves the ecosystem. Ambition with verification is net-positive regardless of whether the headline project ships.

> "As far as I can tell, these problems are problems because people are implicitly carrying around a pre-agentic cost model."

The essay's core diagnosis. Ten years ago the person who wrote a new compiler because theirs was slow "was the kind of person who never ships anything"; most productive programmers still have that aversion. Luu's own examples — a regex engine, a ripgrep fork, a Rust interpreter with a 50x cold-build speedup — took "a low single-digit number of hours" with zero domain expertise. The bottleneck has moved from implementation to deciding what's worth building.

> "I'm more productive and can work faster, but I find that, in terms of what I'm doing, I get farther from 'it just works' with each more capable model because the gap between what the model can verify effectively without significant handholding and what the model can implement has been increasing."

The sharpest and least obvious claim: capability growth expands the *ambitious frontier* faster than it expands the "just works" zone. Every model upgrade pushes Luu onto harder problems where models still struggle. This complicates the naive "software is solved" narrative he explicitly rejects, and it matches the verification-bottleneck literature — see [[Agentic Verification]].

> "Just because other people are pumping out vast quantities of barely working slop doesn't mean that you have to do the same thing."

Luu's answer to the vibeslop despair that dominates aggregator discourse. The market is flooded with slop benchmarks and non-working software, but that's an observation about other people's incentives, not a constraint on yours. With reasonable testing and benchmarking, "works for me" quality is genuinely achievable on projects that were previously absurd.

> "If you do exactly what you would've done five years ago, but with AI, the speedup is fairly poor because of Amdahl's law."

The economics framing that makes the essay more than cheerleading. Some tasks get speedups over 1,000,000x; some get nothing. The productive move is to *choose* what to speed up — targeting work that was previously too expensive to attempt at all, rather than accelerating a workflow whose bottleneck now lies elsewhere.

## Key Themes

- **#concept Pre-agentic cost model**: The invisible budget in your head, calibrated to what building used to cost, now systematically understates what's feasible. Luu's anecdotes (complaints that Rust compilation is slow, when a Rust interpreter is now hours of work) suggest the constraint is imagination, not capability.
- **#pattern Self-checking setups**: The load-bearing exception to Kreinin's caution. Fuzzing, property-based testing, and consistency checks across contradictory specs let a non-expert verify agent output cheaply — Luu's Guards of Atlantis 2 rules reconciliation is the worked example, at "only" a month or two of a $200/mo subscription.
- **#person Michael Malis**: pgrust author and the essay's trigger, whose credibility (in Luu's eyes) substitutes for the due diligence most consumers can't afford — an uncomfortable point about trust in an era of slop.
- **#person Yossi Kreinin**: The loyal opposition. Most people "should be less ambitious... or they'll experience abject failure"; the black-box problem is real even for smart people, and his analogy to microservices (some need it and are good at it; most aren't) is fair.
- **#concept Works-for-me software**: A tier between throwaway and product that barely existed before — software good enough to use daily but never polished to open-source bar. Peter Geoghegan's Neovim formatter and David Turner's podcast ad-skipper exemplify it: "insanely uneconomical to write any other way."

## Critical Analysis

The essay is strongest as an economics claim and weakest as an instruction. "Be more ambitious" is easy to misread as "agents make everything easy" — which Luu explicitly disclaims — but the actionable content is buried: the operative skill is designing verification you can trust, which is why the Kreinin appendix isn't a footnote but half the argument. The honest title of the piece is something like "we can be more ambitious where we can set up self-checking loops."

Luu's examples are also self-selected in a way he half-admits: he's a skilled engineer with unusual taste for tooling work and the model access to match. His own aside that Anthropic's seven-person GitHub replacement "doesn't tell you much about what you can accomplish today without unlimited funds" applies to some of his own projects too. And the speedup numbers ("low single-digit hours") are unaudited — the post was cranked out in half an hour without rigorous benchmarking, as he says up front. That honesty is welcome, but it means the essay is a manifesto, not a study.

Still, the direction of the argument survives the caveats, and two details are under-discussed elsewhere. First, the model-comparison material: GPT-5.1/5.2 "pretty much every suggestion was wrong" on board-game AI, while Fable 5.1/Opus 5.5 could read papers and implement real techniques — a concrete, dated snapshot of the capability step-change. Second, the observation that personal tooling now obsoletes itself in months (his codex-restart utility, the Rust interpreter fork OpenAI will outbuild with $10B of Cerebras) reframes building as *sampling the frontier* rather than accumulating assets — a point that cuts against the durable-artifact ethos elsewhere in this wiki.

## Related

- [[How to Be More Ambitious (Rieseberg)]] — Rieseberg's taxonomy of ambition types (Speed, Altitude, Depth, Range) is the practical playbook this essay motivates; Luu supplies the cost-model argument for why the ambition gap exists in the first place.
- [[Agentic Verification]] — Osmani's verification-bottleneck thesis is Luu's "widening gap between implementing and verifying" stated as engineering doctrine; Luu's fuzzing and property-testing examples strengthen it.
- [[Dan Luu on AI Coding]] — Luu's earlier testing-methodology piece is the methodological foundation this essay assumes: the self-checking setups he prescribes here are the same harness-rigor argument, now applied to personal ambition rather than organizational testing.
- [[Ship Your Interpreter (Basis)]] — Basis's Lean-verified interpreter is the industrial-strength version of Luu's "self-checking setup" pattern, showing what happens when the verification loop is made formal rather than janky.

---
*Sources: [[raw/ambitious]], [[summary/ambitious]]*
*Last updated: 2026-10-28*
