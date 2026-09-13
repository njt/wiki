# How I Prompt (Thorsten Ball)

Thorsten Ball — co-creator of the Amp coding agent, 99% of which is AI-written — shows the actual prompts behind Amp's features at Laracon US 2026 and argues prompting has no secret sauce: it is technical writing for a reader who knows everything public and nothing about you. The one question to ask of every prompt is "how is the model supposed to know what I mean?", and everything else — pointing at files and docs, screenshot-heavy prompts, the "one-two punch" of letting the agent find context before you make the ask, and AGENTS.md trails that move the "how" into the codebase — falls out of that question.

---

## Key Quotes

> "If you ask a model to do something, the information with which it can interpret what you're saying, your prompt, can only come from a bunch of places."

The whole talk compressed into one sentence. He enumerates them: training data (fixed, public), the context window (system prompt, tool definitions, MCP tool definitions, skills, agents, messages, tool results, your prompt), and the codebase. Everything he does afterward is an exercise in picking the cheapest place to put information so the model has it when it needs it.

> "It won't say, 'What the hell are you talking about? What bug? What upload?' It will say, 'You're absolutely right. I'm going to fix the bug with the upload.' And then it's going to do whatever it thinks that is."

The failure mode of an under-specified prompt isn't a question — it's confident compliance with a guess. This is the sycophancy trap stated as a prompting problem: the model's agreeableness converts your ambiguity into its hallucination, silently. It's also the strongest argument in the talk for why "the information has to come from somewhere" is not optional diligence.

> "Somebody hands you a note, and on that note it says, 'Fix the bug with the upload.' ... That's what the model experiences."

The van analogy: a senior engineer who has done everything, read all of Stack Overflow, knows every language — hooded, thrown in a van, sat at a desk with an unfamiliar codebase, and handed a terse note. This is the best one-paragraph intuition pump for agent context I've seen in the wiki. Your prompt is a note passed to a hyper-qualified stranger who cannot ask follow-up questions and will never admit they're lost.

> "The goal of all writing is to be understood. ... Every time you write something, you have to think in a loop, what does the reader know, and what do I need to tell them?"

He locates prompting in the technical-writing tradition — commit messages, Slack messages, PR descriptions — rather than in a new "prompt engineering" discipline. The reader just happens to have a strange knowledge profile: perfect recall of the public internet, zero access to anything private you haven't written down.

> "So, instead of putting everything in the first prompt, I ask it to find the information. ... I call this the one-two punch."

His one coined term. First prompt: "Find the asset for this trumpeter guy" — the agent locates the asset, the component, the OG-image variant; now the context window contains the territory. Second prompt: make the ask. A variant sets the target before the ask: "Investigate how queuing works in our CLI. It is the gold standard for how it should work" — then fix the web version. The agent learns what *good* looks like before being asked to produce it.

> "When you start to move the how to do things out of the prompt, then it's in the codebase. ... It doesn't change the fact that the information has to come from somewhere, but if you make your codebase contain it, you don't have to put it in your prompt."

The closing move, and his favorite way to prompt. A litter of AGENTS.md files — root-level, plus one in the storybook folder saying which port it runs on and how to check it — means "implement this fix and show me a screenshot of the storybook story" is a complete prompt. The codebase becomes a trail of answers to questions the agent hasn't asked yet.

## Key Themes

**#concept Information provenance** — The organizing question: for every fact your prompt depends on, where does the model get it — training data, context window, codebase, or prompt? Anything that lives only in your head must be written somewhere the model can see, or it will be invented.

**#pattern The one-two punch** — Use the agent's own tools as a retrieval mechanism: prompt one makes it gather context (find the asset, investigate the gold-standard implementation), prompt two makes the ask. Cheap, portable, and it converts the agent from guesser into reader.

**#pattern Screenshots as context** — "90% of my prompts have screenshots." A screenshot of a customer's Slack question, a colleague's UI complaint, or a Slack thread about feature design carries more usable information per second of authoring time than prose. His second takeaway: "Take more screenshots."

**#pattern AGENTS.md trails** — Instruction files scattered at directory boundaries, auto-loaded when the agent touches a sibling file. The endgame of the talk: the codebase itself teaches the agent how to work in it, shrinking prompts to noun-verb requests.

**#tool Amp** — The coding agent Ball co-founded (launched weeks after Claude Code), used as the demo surface: orbs (sandboxed headless agent runners), the dev sidebar's auto-generated debug prompts, Amp report IDs that compile local diagnostics into a fetchable bundle via a custom plugin tool, and "the painter" image generator. Also a soft product pitch — see analysis.

**#person Thorsten Ball** — Author of *Writing an Interpreter in Go* and *Writing a Compiler in Go*; previously Sourcegraph and Zed. His credential for a talk on prompting: Amp, a complex distributed system his team built almost entirely through prompts.

## Critical Analysis

**The reduction is genuinely clarifying.** Most prompting advice is incantation — magic keywords, persona framing, "ask it to think step by step." Ball dissolves all of it into a single question with a small closed answer set. Prompting isn't a new discipline; it's writing for a reader with a bizarre but *knowable* knowledge profile. That profile is the van analogy: everything public, nothing private, no follow-up questions, infinite confidence. Once you accept it, "prompt engineering" becomes information-placement engineering, and most of his techniques are obviously correct rather than clever.

**But "there is no secret sauce" undersells how much sauce Amp itself is.** The talk's structure is "you can do this at home," yet the best examples quietly depend on product infrastructure: the dev-sidebar button that auto-generates a complete debug prompt (docs, IDs, data dumps, debug scripts) so his hand-typed part is one line; the Amp report system that compiles a user's diagnostics into a bundle an agent can fetch through a custom tool; orbs that let a 15-minute headless run produce storybook screenshots. None of that is magic, but it is *harness* — exactly the part most users don't have. The honest version of the talk is: "my prompts are short because my tool does the typing." That's a stronger thesis than "no secret sauce," and it points at [[Harness Engineering]]'s feedforward layer, which he never names.

**Every prompt shown worked.** "It worked" recurs like a drumbeat — the daemon-mode prompt, the trumpeter variations, the Gibson Flying V, the scroll-hint fix. We never see a prompt that failed, a thread that derailed, or a recovery from a confident wrong guess. For a talk whose central failure mode is "the model will do whatever it thinks you meant," the absence of any *time it thought wrong* is a notable omission. The claims are also unfalsifiable as presented — no token counts, no retry rates, no comparison against the naive prompts he mocks. The wiki's empirical sources on instruction files ([[Benchmarking AGENTS.md Changes]]) suggest the picture is less frictionless than the stage version.

**The AGENTS.md-trail advice collides with the instruction-budget literature.** Ball treats littering AGENTS.md files through the repo as nearly free — information in the codebase, prompts get shorter, everyone wins. [[Writing a Good CLAUDE.md]]'s instruction-budget argument says the opposite: frontier models reliably follow ~150–200 instructions and degrade *uniformly* as count grows, so every auto-loaded file spends the same scarce budget. And [[Benchmarking AGENTS.md Changes]] found that instruction-file changes improve average scores while regressing specific task types — "AGENTS.md inversion." Directory-scoped files (loaded only when the agent enters that folder) mitigate this, and Ball's storybook example is exactly the right scope for a trail: narrow, operational, how-to-run-this. But the talk gives no criterion for what earns a trail versus what silently taxes every session. The synthesis the wiki still lacks: trails are cheap when *scoped to a directory's mechanics* and expensive when they encode *behavioral preferences*.

**The sycophancy quote deserves more weight than the talk gives it.** "It will say, 'You're absolutely right'" is presented as a reason to write better prompts, but it's also a systems problem: the same agreeableness that turns "fix the bug" into a hallucinated scope is what makes agents rubber-stamp their own work. Pointing at gold-standard implementations — Ball's best pattern — is a mitigation, but the talk's framing keeps the whole burden on prompt clarity when the failure mode is architectural. [[Silently Resolved Ambiguity Is Comprehension Debt of Intent]] names the debt Ball is describing without quite naming it: an under-specified prompt produces a decision with no signature.

**What's right and rare: prompts shown as *pointers plus constraints*.** The daemon-mode prompt is three folders and three files named in two paragraphs, then the half-formed intentions that exist nowhere else — "no checkouts, no worktrees," "it should be super thin," "keep this in the back of your head." That last one sounds absurd until you notice what it is: an honest description of the contents of a human head being serialized because the alternative is the model guessing. The talk's real subject is the boundary between what you must say and what the codebase can say for you — and "make the codebase say more" is the same conclusion [[AI Slop Starts with the Codebase Itself]] reaches from the training-data side: the codebase is the prompt.

## Cross-References

- [[AI Slop Starts with the Codebase Itself]] — Ball's talk is that essay's argument from the practitioner's side. The essay says a codebase aligned with training data is a productivity multiplier because "the codebase IS the prompt"; Ball operationalizes it — point at files, and move the "how" into AGENTS.md trails — and adds the counterpart the essay misses: the codebase isn't just read passively, it can be deliberately *grown* to carry information agents need.
- [[Writing a Good CLAUDE.md]] — The two agree that models are stateless and information must arrive in tokens, but they pull opposite levers: Kyle argues a root CLAUDE.md must stay small because instruction-following degrades uniformly, while Ball wants instruction files *everywhere*, scoped per directory. Ball's folder-scoped trails are a partial answer to the essay's open question about sub-directory instruction files — and a stress test for its instruction budget.
- [[Benchmarking AGENTS.md Changes]] — The empirical corrective to this talk's optimism. Stet shows AGENTS.md edits are runtime behavior changes that improve averages while regressing specific task types, and must be holdout-validated rather than shipped on "it worked" — the exact evidentiary standard Ball's stage demos skip. Read together: Ball supplies the placement philosophy, Stet the measurement discipline it needs.
- [[Matt Pocock — Grill Me, Then Go AFK]] — Two practitioner pipelines with mirror-image alignment strategies. Pocock aligns by interrogation ("grill me") to reach a shared mental model before building; Ball aligns by *pointing* — files, docs, screenshots, gold standards — so the model builds the shared context itself. Ball's is cheaper and async-friendly; Pocock's surfaces the assumptions you didn't know you had. Ball's context model also complements Pocock's smart zone/dumb zone: Pocock's is about *how much* fits in context, Ball's about *where it comes from*.

---

*Sources: [[raw/how-i-prompt]], [[summary/how-i-prompt]]*
*Last updated: 2026-09-13*
