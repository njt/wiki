# The State of Context Engineering (Reunig)

Drew Breunig's AGNTCon talk is the best single-map survey of context engineering as a discipline: from the 2024 "RAG is dead" long-context bet, through the four failure modes that killed it (poisoning, distraction, confusion, clash), the six-tool harness toolbox that answered it, and on to the 2026 frontier of models that drive their own harnesses — which creates two brand-new failure modes, "fighting the weights" and "context inertia," cured with the same old tools. His closing claim is the thesis: every advance in model capability changes what we delegate and what systems must enforce, and the job is knowing when to be deterministic and when probabilistic.

---

## Key Quotes

> "Long contexts let you fit in more information, but they also let you fit in more errors."

The whole 2024–25 story in one line. The Pokémon example makes it concrete: Gemini 2.5 burned hours hunting tea for a guard who wanted lemonade, because *FireRed* knowledge in its weights poisoned the context, and once on the to-do list "the model could not get past it."

> "When you crack open a prompt and see repetition and all-caps, there's a good chance you're fighting the weights."

A sharp diagnostic heuristic. Claude Code's system prompt begs for parallel tool calls seven times precisely because RL training environments were serial. Prompt verbosity as an archaeology of training bias — you're not writing instructions, you're counteracting gradients.

> "Delete your context. I'm not joking. Around 200,000 tokens is my warning point."

The folk wisdom of 2026, stated without embarrassment. Three independent research efforts (Anthropic's Memento, Microsoft's Fork-and-Flush, Berkeley's *Scaling Discovery Requires Forgetting*) converged on the same fix within one week: periodically wipe the conversation, keep the artifacts. We trained models to be persistent, and persistence became the bug.

> "In 2023 we called it prompt engineering. In 2025, context engineering… then harness engineering. But the job is the same."

Breunig's framing dissolves the naming wars: each renaming marks work migrating from probabilistic model calls into deterministic code. The Stewart Brand quote — look for where language is being invented — lands, and his gloss is correct that the *goal* never moved.

## Key Themes

#concept #pattern #agent-architecture #context-engineering

## Analysis

The talk's real contribution is its periodisation. Rather than a list of tips, it gives a causal loop: model makers target a capability → it creates new failure modes → harness builders respond with (mostly the same) tools. The six-tool toolbox has been stable through two paradigm shifts, which is either evidence the toolbox is right or that the field is still rearranging furniture. I lean toward the former, but the 16-line DSPy demo cuts both ways: it shows the patterns are genuinely simple, and that the hard part was never the code — it's the judgment about *when* to prune, quarantine, or compact, which is exactly the part now being pushed onto the model.

The "fighting the weights" section deserves more attention than it's getting. It reframes prompt engineering as adversarial negotiation with RL training environments you can't see, and it implies labs should ship their training-environment tool conventions as documentation. The prediction about long-lived personal agents accumulating mutually conflicting skills and memories — "strategic forgetting" as the unsolved problem — is the most provocative claim here, and nobody has solved it.

## Relations

This talk is a state-of-the-field survey that strengthens [[Agentic Context Management]]'s lifecycle framing — Breunig's six tools are essentially that paper's primitives (compaction, scoping, anticipating) arrived at from practitioner experience rather than arXiv. It nuances [[Context Rot]]: that page treats degradation as a retrieval-quality problem to be scored and fixed; Breunig shows it's structural — even perfect retrieval can't save a context that's simply too full of its own history. And it complicates [[New Rules of Context Engineering]]: Anthropic's argument that newer models need *less* context scaffolding sits uneasily beside "fighting the weights," where more model capability means deeper baked-in biases that prompts must actively counteract. Finally, it adds the newest failure mode to the catalogue in [[Agent Memory and Context]]: context inertia, where the agent's own persistence is the contaminant.

---
*Sources: [[raw/the-state-of-context-engineering-html]], [[summary/the-state-of-context-engineering-html]]*
*Last updated: 2026-10-26*
