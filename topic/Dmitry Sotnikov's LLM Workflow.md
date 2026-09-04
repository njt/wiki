# Dmitry Sotnikov's LLM Workflow

Dmitry Sotnikov (yogthos — author of *Web Development with Clojure*, builder of the Jolt Clojure compiler) distills months of daily LLM-assisted development into a concrete workflow. His distinctive framing: the agentic loop is a genetic algorithm where tests are the selection pressure, scaffolding is the genome, and the developer is the fitness function. The pieces worth stealing are the "evil genie" model of the naive-implementation trap, the rule that a wrong-first-shot agent should be reverted rather than debugged, and the case for elevating routing logic to state machines with the model as a leaf node inside a deterministic control structure.

---

## Key Quotes

> "In a way, the process is the inverse of regular programming. We tend to build up programs step by step when writing code by hand... LLMs tend to produce a lot of code out of the gate and the focus shifts to whittling the code down to what you actually need."

The inversion is real and under-discussed. Most advice treats LLM output as something to grow; Sotnikov treats it as something to prune. If you're not comfortable deleting generated code, you'll end up maintaining the agent's first draft forever.

> "A good way to look at the agentic loop is to view the process as a genetic algorithm... it gradually converges on a solution that fits the parameters being tested."

This is the most transferable mental model in the piece. It reframes tests from a safety net into the environment the code is evolving *against* — which is why Sotnikov can call them "the contract that the agent works against" and, later, "the selection pressures that drive the evolution of the code." The model is not building software so much as being bred by your test suite.

> "It is akin to an evil genie that will interpret your queries in the worst way possible, leading to the solution having a completely wrong shape. The trick is that you have to spell out the constraint, which incidentally forces you to think through the problem as well."

The evil-genie framing captures the naive-implementation trap better than "hallucination" does. The agent isn't wrong about details; it's wrong about *shape* — routing string calls through a dispatch table, re-deriving the receiver type per invocation, allocating a cell per element when the collection already knows its length. Spelling out the constraint is also a forcing function on the human, which is the quiet insight: the discipline of writing constraints *is* thinking.

> "If the agent does not get the solution mostly right on the first shot, it is unlikely to make it work properly later... it just adds kludges to fix your specific complaint and the problems tend to multiply... If it starts spiraling, then it is time to reframe your problem statement and start from scratch."

The strongest practical rule in the article. It cuts against the instinct to keep patting the agent toward a fix. Sotnikov's evidence: agents don't step back to re-understand the problem when you point out a bug — they layer kludges onto a bad foundation. Git is the safety net that makes "revert and reframe" cheap instead of painful.

> "You have to remember that the AI does not know the specific quirks of your project... The key to using LLMs effectively is to make sure you already have a solid understanding of what you are aiming to build before you start... LLMs are good at filling in the gaps and doing boilerplate, but you still have to do design and architecture the same way you always did."

The domain-expertise requirement, stated bluntly. Sotnikov's ability to build a Clojure compiler with these tools "stems from nearly two decades of experience working with the language." Without that, he says, you're "throwing darts at the board" — which is the practitioner's version of [[Claude Is Not Your Architect]].

> "Routing logic should be elevated to first class citizenship in the design. State machines are the natural fit for this, since they force the separation of what to do from how to do it. The control flow logic can be largely declarative and expressed as a graph such as the Mermaid diagram... while the implementation details live at each step in the flow."

The workflow's structural core. The Mermaid diagram isn't documentation — it's the constraint that confines the agent to your architecture. Breaking the flow into independent steps does double duty: smaller context per task *and* smaller review surface.

> "The main idea is to move from treating the model as the whole agent to making it a primitive, which produces a behavior. The workflow is then composed with a small set of classical control structures... even a local model can solve fairly complex tasks competently."

The most novel — and least-evidenced — claim. Sotnikov borrows from Behavior Trees to demote the model from decision-maker to leaf node, wrapping it in verifier/critic gates and a failure ladder. It's the logical endpoint of the guardrail-as-structure school, but it's one practitioner's harness (Dirge) plus one paper, not established practice.

---

## Key Themes

- **Genetic algorithm as the loop** — tests are selection pressure; the model is bred by the test suite, not guided by instructions. #concept
- **The evil-genie trap** — agents optimise for the letter of the prompt and get the *shape* wrong, not just the details; spell out constraints. #pattern
- **Revert, don't debug** — a wrong first shot only accrues kludges; reframe and start over under Git's safety net. #pattern
- **Scaffolding is the genome** — plan first, Mermaid diagram, state machines, low coupling, functional style: give the agent less room to invent structure. #pattern
- **Tests as contract** — TDD, storybooks, Playwright, and a benchmark suite form the fitness function the code evolves toward. #pattern
- **Model as leaf node** — Behavior Trees demote the LLM to a primitive inside deterministic control structures. #concept
- **Domain expertise as prerequisite** — the LLM amplifies an expert; it cannot substitute for one. #concept
- **Dirge** — Sotnikov's hand-built harness: Janet plugins, SQLite memory, critic role, mechanical paren repair. #tool

---

## Critical Analysis

Sotnikov's essay earns its place in the wiki not for novelty but for compression. Every idea here exists elsewhere — the genetic-algorithm framing rhymes with [[Structural Backpressure Beats Smarter Agents]]' Ralph Loop, the tests-as-contract with [[A New Era for Software Testing]], the scaffold-first discipline with [[Specifications as the Product]] — but nobody else has packed them into a single, coherent, phone-readable workflow from a decade-plus expert's daily practice. The value is the synthesis, not the ingredients.

The strongest claim is "revert, don't debug." It's falsifiable, it's cheap to act on, and it names a failure mode — the kludge spiral — that most workflow advice hand-waves as "iterate more." Sotnikov ties it to the genetic-algorithm framing: if the first genotype is unfit, subsequent mutation around a bad local optimum rarely escapes it. That's a concrete reason, not a vibe.

The weakest claim is the Behavior Trees conclusion. "Even a local model can solve fairly complex tasks" rests on one practitioner's harness and one cited paper, and the mechanism — the model as a leaf node in fixed control structures — is exactly the kind of thing that works in the author's own hands and may not generalise. It deserves to be treated as a promising bet, not a settled result, and it's notably thinner than the evidence base behind the deterministic-gate school it descends from.

There's also a real tension Sotnikov leaves unexamined. He argues both that the LLM must never see a blank canvas *and* that you should spike freely to learn — "when you hit a point where you are not sure what to do... spike up different ideas and see how they pan out." The Janet-to-Chez week-long spike is the reconciliation in practice: exploration and scaffolding are two different modes, and he switches between them. But he never says so explicitly, and conflating the two is precisely how teams drift into the [[The Enterprise Gap from Vibe Coding|enterprise gap]].

What's absent is the team dimension. This is a solo-expert workflow — one person, deep domain knowledge, a hand-built harness. The moment you have ten developers and shared scaffolding, the questions change: who owns the state machine, who reviews the PRs, whose domain expertise is the fitness function. The wiki's [[Agent Coding Workflow]] hub is where that question lives, and Sotnikov's piece is the solo baseline it builds on.

---

*Sources: [[raw/2026-08-17-llm-workflow-html]], [[summary/2026-08-17-llm-workflow-html]]*
*Last updated: 2026-09-04*
