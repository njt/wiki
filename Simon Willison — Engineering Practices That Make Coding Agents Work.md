# Simon Willison — Engineering Practices That Make Coding Agents Work

Simon Willison's Pragmatic Summit talk (February 2026) is the most concentrated statement of his engineering philosophy to date. In 28 minutes he covers the adoption ladder (newest rung: don't read the code), TDD as the agent unlock, conformance-driven development, the lethal trifecta of prompt injection, why managing agents is so exhausting it might save our careers, and how open source is being reshaped — uncomfortably — by a world where you can vibe-code a date picker faster than you can evaluate a library. It's Willison at peak form: the practitioner who's shipped more with agents than almost anyone, naming patterns in real time while admitting exactly where the boundaries are fuzzy.

---

## Key Quotes

> "The new thing, as of what, three weeks ago, is you don't read the code. … That is a wildly irresponsible thing to do. … But I think it's possible, if you figure out how to get the agents to prove that the code works."

Willison calls StrongDM's "nobody writes code, nobody reads code" policy "clear insanity" for a security company, then immediately explains why it might work anyway. This is the talk's defining move: he refuses both the extremist position and the reactionary one. "Don't read code" is insane — and it's where we're headed, so we'd better figure out how to make it safe. The tension between these two truths powers everything that follows.

> "Tests are no longer even remotely optional. Tests are—they're free now. … I hated TDD my entire career. And then I started working with agents and I was like, oh, this is—now I get it."

The most important five words in the talk: "use red-green TDD." Five tokens that dramatically raise the probability of working code. Willison's conversion story matters because he's the exact person who would resist TDD — a pragmatic solo developer who moves fast — and agents changed his mind. The old objection ("tests are extra work") dissolves when you're not the one doing the work. See [[Agentic Manual Testing]] for the companion practice (manual verification by the agent).

> "I end up with code that is way better than the code I would have written by hand because I'm a little bit lazy."

The most honest thing anyone has said about code quality with agents. If a refactoring would take you an hour and you'd skip it, the agent does it while you walk the dog. The constraint isn't the agent's capability — it's whether you bother to ask. This reframes code quality as a choice you make (or don't), not a property of the tool. Pairs with [[Radical Accountability]]: taste is all that's left when the machine does the work.

> "The lethal trifecta is when you've got a model which has access to three things: private data that you care about, the ability to be fed malicious instructions, and some kind of channel by which it could exfiltrate that private data. … The only guaranteed solution is to cut off one of the legs."

Willison coined both "prompt injection" and "the lethal trifecta" and remains the field's clearest thinker on AI security. His own behavior is contradictory — he runs Claude with dangerously skipped permissions on his Mac while defaulting to Anthropic's hosted containers on his phone. He copes to the contradiction: "I am the world's foremost expert on why you shouldn't do that." See [[Security and Sandboxing]], [[yolo-cage]], [[Moltbook]].

> "I try not to predict more than a week ahead. … I think it would take six months just to explore what Opus 4.6 can do."

The anti-prediction. While LinkedIn fills with "10 AI trends for 2027," the person who's shipped more agent-built projects than almost anyone refuses to look past next week. His advice: every time a model fails at something, bookmark it and retry in six months — you might be the first to discover a new capability. This is an epistemic stance dressed as humility.

> "I think that might be what saves us. I think the fact that no, you can't have one engineer and have him do a thousand projects because after three hours of that he's going to literally pass out in a corner."

The most original argument about AI and employment I've heard. Running three or four agents in parallel requires "firing on all cylinders" — after two hours, Willison is done for the day. The bottleneck isn't the AI; it's human cognitive stamina. This inverts the standard fear (one engineer replaces a thousand) into a physiological limit. Compare with [[Probabilistic Engineering and the 24-7 Employee]] for the training-crisis dimension, and [[Don't Wait for Claude]] for the session-management exhaustion.

> "Don't learn it, just start writing code in it."

On learning new programming languages. Willison shipped three Go projects in two weeks despite not being a fluent Go programmer. His method: scan the agent's output well enough to verify intent, rely on TDD loops for confidence, and trust that you'll absorb the language through exposure. This is either the democratization of polyglot programming or a recipe for [[Cognitive Debt]] — probably both.

---

## Key Themes

#concept #tool #pattern #person #coding-agents

### The adoption ladder's new rung

Willison traces six stages of AI adoption, from asking chatbots questions through agents writing most of the code, to policies where no human writes code, to the newest: no human *reads* the code either. Each stage has been dismissed as irresponsible until it wasn't. The "don't read" stage arrived weeks before this talk. See [[Five Levels from Spicy Autocomplete to the Dark Software Factory]] for a parallel taxonomy, [[StrongDM Factory Techniques]] for the company embodying the extreme, and [[The Cult of Vibe Coding Is Insane]] for Bram Cohen's rebuttal.

### TDD is the unlock

The talk's central practical claim. Willison's argument isn't ideological ("TDD is good practice") — it's empirical: telling an agent to use red-green TDD dramatically raises the success rate for negligible token cost. Tests become "effectively free" when you're not writing them. This is TDD as [[Compound Engineering]] — add a system (tests) rather than manual verification. See [[Spec-Driven Development]] for the full triangle (specs, tests, code), [[Feedback Loop is All You Need]] for the theoretical basis, and [[Designing Agentic Loops]] for how tests make agentic loops converge.

### Execution as verification

Automated tests aren't enough. Willison instructs agents to start the server and exercise the API with `curl` — manual testing that catches real-world bugs the test suite missed. His Showboat tool (48 hours old at the talk) produces a markdown log of `exec` commands and their actual output. The deeper principle: LLMs hallucinate results they expected to see; capturing actual stdout/stderr is the antidote. See [[Agentic Manual Testing]] for the full treatment of this practice.

### Conformance-driven development

Two patterns. The simple one: give an agent a language-agnostic conformance suite (like WebAssembly's spec tests) and say "write code until this passes." Willison built a "janky but working" Python WebAssembly runtime this way. The advanced one: have an agent build a test suite by studying six existing implementations, then use that suite to drive a new one. He did this for file uploads in Datasette. The test suite IS the spec — see [[Specifications as the Product]].

### Code quality is a dial, not a binary

For one-off vibe-coded tools, spaghetti is fine. For maintained projects, tell the agent to refactor. Willison's key insight: agents will do the tedious cleanup you'd skip. The obstacle to high-quality agent-generated code isn't the model — it's whether you care enough to ask. Counterpoint: [[AI Coding Tools Create More Bugs Than They Fix]] documents what happens when nobody asks. See [[Write Only Code]] for the "slop radius" concept.

### Templates as straitjackets

Willison maintains half a dozen `cookiecutter` templates that scaffold projects with tests, README, and CI pre-configured. Even one or two tests in your preferred style shape everything the agent writes. The analogy: if you're first to use Redis at your company, you must do it perfectly because the next person will copy-paste you. Same with agents. This is [[Harness Engineering]]'s feedforward layer — shape the starting context, not the prompt.

### The lethal trifecta and the sandboxing gap

Willison's security framework is clear-eyed, and his personal practice contradicts it. He knows prompt injection can't be parameterized away (unlike SQL injection), he knows sandboxing is the only guaranteed defense, and he still runs Claude with `--dangerously-skip-permissions` because "it's so good, so convenient." The gap between what he knows and what he does is the gap the whole industry is living in. See [[Security and Sandboxing]] for the full landscape, [[Moltbook]] for the lethal trifecta in production, and [[yolo-cage]] for a sandboxing approach that doesn't require convenience sacrifice.

### Open source is being hollowed out from both ends

Two threads that Willison doesn't fully knit together. On one end: why use a date-picker library when Claude writes the exact widget you want? The market for paid component libraries "has collapsed." On the other end: maintainers are drowning in junk AI-generated PRs, with some wanting GitHub to disable pull requests entirely. The tool that makes libraries obsolete also floods their maintainers with noise. See [[I Don't Want Your PRs Anymore]], [[AI Killing B2B SaaS]], and [[Simplicity in the Age of AI-Assisted]].

---

## Critical Analysis

This is the best single-talk summary of Willison's engineering philosophy, better even than his written pieces because the live format forces compression. Every section earns its place. The talk has more actionable technique per minute than most conference keynotes have per hour.

**What lands hardest:** The TDD conversion story. Willison's authority on this topic comes from being a skeptic who changed his mind, not an evangelist who always believed. "Tests are free now" is a claim that changes behavior. The mental exhaustion argument is genuinely novel — I haven't seen anyone else make the case that human cognitive stamina, not AI capability, is the binding constraint on agent productivity. And the "don't learn it, just write code in it" approach to new languages is either brilliant or reckless, and Willison's track record makes it hard to dismiss as reckless.

**What's missing:** The talk raises "prove the code works" as the alternative to reading it, then never explains how. Conformance suites work for WebAssembly; they don't work for a CRUD app with business logic. The "don't read code" threshold is asserted but not defined — what's the heuristic for when it's safe? The security section is intellectually honest but practically thin: "I know this is dangerous and I do it anyway" is a confession, not guidance. And the open source section names two problems without exploring whether they're connected (hint: they are — the same tool that lets you skip libraries generates the PRs flooding maintainers; the agent is simultaneously consumer and polluter).

**The exhaustion argument deserves more scrutiny.** Willison frames cognitive stamina as the saving grace — one engineer literally can't run enough agents to replace a thousand. But this assumes (a) tooling won't improve to reduce cognitive load, (b) organizations won't structure around the limit (shift work, as [[StrongDM Factory Techniques]] documents), and (c) the economic incentive to push past the limit won't find ways to do so. The history of industrial automation is the history of human limits being engineered around. Betting on exhaustion as a durable constraint is a bet against tooling progress — and tooling progress is the premise of the entire talk.

**The Willison synthesis.** Across this talk, [[Designing Agentic Loops]], [[Agentic Manual Testing]], [[2025 in LLMs]], and [[Moltbook]], a coherent methodology emerges: shell commands over MCP, self-documenting tools via `--help`, execution-as-verification, TDD as the force multiplier, sandboxing as the only real security, and pragmatic heuristics over heavyweight frameworks. It's the most complete "school" of agentic development from any single practitioner. If you were going to adopt one person's approach wholesale, Willison's is the strongest candidate — with the caveat that even he doesn't follow his own security advice.

---

*Sources: [[raw/simon-willison-pragmatic-summit-engineering-practices]]*
*Original talk: [The Pragmatic Summit](https://www.youtube.com/watch?v=owmJyKVu5f8), February 11, 2026*
*Last updated: 2026-05-18*
