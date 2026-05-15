# Slowing Down in the Age of Coding Agents

Manuel Odendahl's third piece in his series on AI-assisted development, following [[Simplicity in the Age of AI-Assisted]]. Where the first piece argued LLMs make simplicity cheap, this one argues the bottleneck has shifted from code production to thinking — and the only competitive response is to deliberately slow down. His method: e-ink tablets, annotation with a pen, and vocabulary tracking as quality control.

---

## Key Quotes

> "The bottleneck in agent-assisted development is no longer writing code. It's understanding what should be written."

Commentary: This is the thesis in one sentence. Every tool and workflow in the agent ecosystem exists to make agents write more code faster — but Odendahl says that's optimizing the wrong thing. The scarce resource is now human judgment.

> "A mediocre prompt produces mediocre architecture at high speed."

The corollary to the core thesis. If thinking is the bottleneck, prompt quality isn't a nice-to-have — it's the entire game. Typing prompts by hand forces you to decide what matters before the agent starts generating.

> "Every word it generates is basically a query of the training corpus."

The most original insight in the piece. LLMs don't invent vocabulary — they fetch it from training data and recombine it. Words like "Controller" and "Manager" aren't architectural choices the agent made; they're the linguistic gravity of a million Java Spring repos pulling your codebase toward enterprise bloat. Odendahl tracks these as they appear, the way you'd track introduced species in an ecosystem.

> "The physical act of forming letters slows me down enough to sit with a thought longer than I would on screen."

> "LLMs inherit the cargo cult — the architectural complexity baked into their training data."

Commentary: This connects directly to the earlier [[Simplicity in the Age of AI-Assisted]]: "The LLM will not question your architecture." But here the argument is sharper — it's not just that LLMs won't question YOUR architecture, it's that they'll silently import EVERYONE ELSE'S architecture too.

---

## Key Themes

#agentic-coding #workflow #annotation #vocabulary #complexity #prompt-engineering

**The annotation cycle.** Odendahl's workflow is a three-phase loop: agent produces design document → human annotates on e-ink tablet with pen → filtered list of corrections flows back to agent. 90% of annotations are process, not product — the physical act of writing forces the slow engagement that screen reading doesn't. This is the most concrete "slow down" workflow anyone has published.

**Vocabulary drift as early warning.** Words are architecture. When an agent starts using "Registry" where the codebase uses "Store," that's not a synonym — it's a pattern import from a different architectural tradition. Odendahl treats vocabulary as a leading indicator of architectural drift, catching it before it becomes [[Cognitive Debt]].

**Literate programming at commit granularity.** Design documents that contain file references, API signatures, and pseudocode become the durable artifact. "Having a literate programming for literally every commit is life-changing." This converges with [[Specifications as the Product]] from the opposite direction — Odendahl arrives at spec-as-product through workflow discipline rather than economic argument.

**Analog tools as cognitive guardrails.** The e-ink typewriter and tablet aren't Luddism. They're deliberate constraints that create "a different kind of cognitive space — slower, less reactive, more generative." Same principle as [[Compound Engineering]] but implemented in wetware: mechanical constraints that make speed-without-quality impossible.

---

## Critical Analysis

This is the best "slow down" piece in the wiki. Zechner's [[Slowing the Fuck Down]] diagnosed the problem and offered a philosophy; Odendahl provides a concrete, reproducible workflow with tooling you can actually adopt. The annotation cycle — design doc → e-ink tablet → pen markup → filtered feedback — is specific enough to try tomorrow.

The vocabulary-as-early-warning insight is genuinely novel and under-explored in the agent literature. Everyone talks about code review; almost nobody talks about word-choice review. Odendahl is right that "Controller" vs. "Handler" vs. "Manager" isn't bikeshedding — it's architectural intent leaking through language. [[Prefix Effects]] covers naming gravity within a codebase; Odendahl extends it to the gravitational pull of the training corpus itself.

The weak point: this workflow requires tools (e-ink tablet, e-ink typewriter) and habits (morning coffee shop reading) that are personality-dependent. Not everyone will annotate with a pen, and the "90% of annotations serve only the moment" admission means most of the friction produces nothing durable. The throughput cost is real. For someone who doesn't find handwriting generative, this workflow would just be slower, not better.

The deeper question Odendahl doesn't answer: is this workflow effective BECAUSE of the annotation cycle, or because it forces a design-document-first approach that would work equally well on a laptop? My bet: the design document is doing most of the work. The pen is enforcing the pause, but any pause would do. The value is in the spec-before-code discipline, not the pen.

---

*Sources: [[raw/slowing-down-in-the-age-of-coding-agents]]*
*Last updated: 2026-05-15*
