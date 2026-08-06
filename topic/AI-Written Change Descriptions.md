# AI-Written Change Descriptions

Kenton Varda's sharp diagnosis of why AI-generated PR and commit messages fail at code review: they describe *what* the code does (visible from reading the diff) while omitting *why* the change exists at all — the higher-level framing that makes review possible. Simon Willison amplifies it as a one-post quote, but the argument is dense enough to deserve its own page.

---

## Key Quotes

> "I declared a moratorium against AI-written change descriptions (such as PR and commit messages). I had found them to be worse than useless when it comes to reviewing code: they spend paragraphs describing things that are obvious from reading the code, while omitting the higher-level framing that I need to understand the broader purpose and decide if the changes make sense."

This is the whole post. Varda packs three distinct claims into two sentences: (1) AI descriptions describe the observable, (2) they omit the architectural *why*, and (3) without that framing, code review can't do its actual job. The word "moratorium" is doing real work here — it's not "I stopped asking AI to write commit messages," it's "I banned them." The severity of the judgment matches the severity of the failure mode.

> "worse than useless"

Not "unhelpful." Not "sometimes misleading." *Worse than useless.* An empty commit message leaves the reviewer knowing they have no context and forces them to reconstruct it. An AI-generated message provides *false* context — it reads like a competent summary and therefore suppresses the instinct to dig deeper. This is the same dynamic [[Claude Is Not Your Architect]] names as the "attaboy problem": AI outputs that read as coherent and confident actively prevent the skepticism that produces good work.

---

## Key Themes

- #concept **The framing gap** — AI describes what changed; code review needs to know *why* it changed and whether that makes sense. These are different information categories, and LLMs are only good at the first one. The failure isn't inaccuracy — it's category error.
- #concept **Worse than useless as a pattern** — AI output that is *plausible but structurally wrong* is more dangerous than AI output that is obviously wrong, because it suppresses the verification instinct. This rhymes with [[Dopamine Fracking]]'s "evaluative anesthesia" and the [[Guardrails and Feedback Loops]] principle that deterministic gates beat probabilistic promises.
- #pattern **The moratorium as engineering practice** — Varda didn't fix the prompt or add more context; he banned the practice. Sometimes the correct engineering response to a tool that produces structurally inadequate output is to stop using it for that task, not to optimize around it. [[Ponytail]] applies the same logic to code generation: "lazy senior dev" persona, YAGNI-first.
- #person **Kenton Varda** — Creator of Cap'n Proto, Protocol Buffers v2, Sandstorm.io, and Cloudflare Workers. His credibility on this claim comes from being one of the most respected systems engineers working, not from being anti-AI. He's not saying AI can't write code — he's saying AI can't write the *framing* that makes code review possible.
- #person **Simon Willison** — Amplifying Varda's quote continues Willison's pattern of surfacing the boundary conditions on AI-assisted development. He's the person who shipped more agent-built projects than almost anyone, who argues "tests are free now," who says you can learn languages by scanning agent output — and he's also the person who amplified [[Understand to Participate|Geoffrey Litt's talk on understanding as participation]]. His curation consistently draws the line between "AI is incredibly useful" and "AI cannot do everything."

---

## Critical Analysis

**Why this lands harder than it looks.** Varda's two sentences are doing something that most AI-code-review discourse misses. The AI-review conversation has focused on *correctness*: can an AI reviewer find bugs, spot security issues, detect anti-patterns? Those are all about evaluating the code itself. Varda's complaint is about evaluating the *change* — whether the delta between old and new makes sense given what we're trying to accomplish. That requires context no diff contains: product decisions, architectural direction, team constraints, the thing we learned last week that made us pivot. An AI can't know those things unless you tell it — and if you've told it, you've already done the work the commit message was supposed to capture.

**The worse-than-useless dynamic is underappreciated.** The standard AI criticism is that it produces confident bullshit. Varda's critique is more specific: AI commit messages aren't bullshit, they're *accurate* in a way that's counterproductive. They describe the diff competently — and that very competence makes the reviewer think they understand the change when they don't. A bad human commit message ("fix stuff") leaves you aware of your ignorance. A good AI commit message leaves you unaware. The second is more dangerous.

**What this implies for agent workflows.** If AI can't write useful change descriptions, the workflow that has an agent produce code + description + PR in one shot has a structural blind spot. Either the human writes the description (preserving the framing that only the human has), or the agent's description becomes noise that masks the absence of framing. This is a concrete argument for [[Vibe Coding as a Team Sport|bram-style approval gates]] where human-written plans are the durable artifact and agent-generated code is a disposable implementation detail.

**The commit message as the only survivor.** [[The Knowledge Chipper]] extends Varda's argument by naming what's *not* in the commit message: the entire context the agent built during the session. The agent scanned files, searched docs, built a rich understanding of the codebase — and when the session ends, the only artifact is "a commit message and whatever amount of code comments the LLM deemed fit." The commit message was always going to be inadequate; the real problem is that everything else was thrown away.

**The Willison effect.** Willison posting this as a standalone quote — no commentary, no "I agree," just the quote — is itself a signal. He doesn't do this for things he disagrees with or finds trivial. The format (quote-only post) is his highest-signal curation mechanism. When Simon Willison runs a quote without commentary, he's saying "this speaks for itself and I endorse it."

---

*Sources: [[raw/kenton-varda]]*
*Last updated: 2026-07-11*
