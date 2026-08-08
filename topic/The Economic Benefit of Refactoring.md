# The Economic Benefit of Refactoring

Martin Fowler's controlled experiment quantifying the token-cost return on refactoring agent-generated code: 15 refactoring steps reduced input tokens for a representative change by 83% (159K → 27K), not by deleting code but by decomposition that let the agent read only what it needed. The first empirical measurement of refactoring economics in an agentic codebase.

---

## Key Quotes

> "The goal of refactoring an agentic code base is to spend tokens now in refactoring to make token consumption for future work lower."

The thesis statement, and the thing that distinguishes this from traditional refactoring rationale. Traditional refactoring is about human comprehension and maintainability; this is about *token economics*. The refactoring investment pays a dividend on every subsequent agent session.

> "Precisely because agents never learn this was now possible to run as an experiment. I could prompt a fresh agent to make exactly the same change after every refactoring stage. Unlike a human engineer, the experiment would not be tainted by learning from previous steps."

The experimental design insight is the most elegant part of the article. With humans, you can never measure the isolated effect of code structure on productivity — the human always learns. Agents don't, and that amnesia becomes a controlled variable.

> "Between the base line and the final refactoring, input tokens for the same task reduced from 159,564 to 27,360. A saving of 132,204 tokens, or 83%. And that saving is not a one-off. Every single change that touches the data access layer from this point forward now costs significantly less."

The headline result. The per-change savings are modest in dollar terms (~$0.40 at Sonnet 5 pricing), but the compounding matters: every future change is 83% cheaper to *start* — the agent reads less code, builds context faster, and gets to the work sooner.

> "This saving is because the agent has to read less code. But it is not because there is less code to read. The overall code in the data access layer as a whole has stayed fairly constant."

The mechanism isn't code deletion — total lines stayed flat. It's *decomposition*: splitting a 17K-line monolith into 19 files let the agent identify and read only the 1–2 files relevant to the change, rather than absorbing the entire module.

> "Randomly cutting the file into smaller files is unlikely to help as much: even if each file were smaller, the agent would be forced to read through many files looking for the relevant code."

The finding that the kind of decomposition matters — it has to be semantic. You can't just split on line count; the agent needs to be able to identify the right file with certainty. This echoes the SonarSource finding that *thin dispatchers over god methods* help agents, while *more methods without better decomposition* hurt.

> "Claude was not good at refactoring. If you read the prompt and the refactoring steps below, it's clear that the refactorings produced were directly in response to the prompt. Claude is unable to look at code, look at refactorings in general and work out which are suitable to apply: a human needs to actively guide it."

A candid assessment of Claude's limitations. The agent couldn't *discover* refactoring opportunities — it could only execute refactorings a human specified. This is a sharp contrast to the narrative of agents as autonomous engineers.

> "It was also bad at applying them. The mechanical act of refactoring was performed by writing Python scripts using grep and sed. These scripts frequently got confused by indentation. Oh, the irony."

The author's amusement at an AI writing fragile text-processing scripts to refactor AI-written code is the article's best joke, and also its most telling observation about agent limitations: the tools LLMs reach for are the wrong tools for the job.

---

## Key Themes

- **#concept Token economics of code structure:** The central contribution. Refactoring isn't just a human-comprehension practice — it has measurable, repeatable token-cost returns for AI agents working on the codebase. This makes refactoring an economic decision, not an aesthetic one.

- **#pattern Agentic refactoring as investment:** Spend tokens now (the refactoring itself) to save tokens on every future change. The ROI calculation needs both numbers — refactoring cost and per-change savings — and Fowler couldn't cleanly measure the refactoring cost (upper bound: 5M tokens). Future work should include that measurement.

- **#pattern Decomposition as the mechanism:** The savings come from the agent reading *fewer* files, not from there being *less* code. Semantic decomposition — where the agent can confidently identify which file to read — is what matters. The biggest single step (splitting store impls into per-trait files) happened late in the sequence, dependent on earlier extract-function and extract-class refactorings that made the split possible.

- **#concept Agent non-learning as experimental control:** The observation that agents don't learn between sessions is usually framed as a problem (the knowledge chipper, cold starts). Fowler inverts it: the amnesia means you can run controlled experiments that are impossible with humans. This is a genuinely novel methodological contribution.

- **#concept Claude cannot refactor autonomously:** Both discovering and applying refactorings required explicit human direction. Claude.ai was better than Claude Code at refactoring *planning* (spotted class extraction where Claude Code only saw function extraction), but neither was good at mechanical execution. This suggests refactoring is a frontier where agent capability still falls short of what the tool-chain promises.

- **#concept Output tokens are inelastic:** Input tokens dropped 83%; output tokens stayed essentially flat. Refactoring helps the agent *understand* the codebase faster, but doesn't make it *write* less code. At 5× the input price, output tokens are the bigger cost — and Fowler flags this as an open question: are there refactorings that reduce output token production?

---

## Critical Analysis

**What's established:** This is the first empirical measurement of refactoring ROI in an agentic codebase. It validates what many practitioners have suspected — that cleaner, better-decomposed code is cheaper for agents to work with. The experimental design (exploiting agent amnesia for controlled measurement) is elegant and reusable. The finding that total lines stay flat while costs drop is important: it recalibrates the conversation from "how much code" to "how is it organized."

**What's suggestive but not proven:** The single-experiment, single-change, single-codebase design means generalizability is unknown. Would the same pattern hold for complex changes that touch multiple modules? For greenfield code vs. brownfield? For TypeScript vs. Rust? The representative change was a straightforward CRUD feature; real-world changes are messier.

**The missing number:** Fowler couldn't measure the refactoring cost accurately. The 5M-token upper bound includes experiment design, planning, and various other tasks. Without knowing the cost, we can't calculate breakeven. If refactoring cost 5M tokens, at 132K tokens saved per change, you'd need ~38 changes to break even — plausible for an actively developed codebase but speculative without better accounting.

**The refactoring competence gap:** The most uncomfortable finding is that Claude couldn't refactor autonomously. If agents can't improve the code they produce — if refactoring remains a human-guided activity — then agent-generated codebases will accumulate the same structural decay that human codebases do, just faster. The development harness's explicit refactoring step *didn't prompt Claude into improving the file*. This is a gap that matters if agentic development is going to scale beyond throwaway projects.

**The output token puzzle:** Input tokens dropped 83%; output tokens were flat. Yet output tokens cost 5× more per token. The economics point to output reduction as the higher-leverage target, and Fowler's experiment provides no leverage there. This is a significant open question: are there structural changes that reduce output token production, or is that fundamentally bounded by the change's complexity?

**Connections to the broader literature:** The SonarSource minimal-pair study ([[Code Cleanliness and Coding Agents]]) found 7–8% token savings from cleaner code — modest but consistent. Fowler's 83% is an order of magnitude larger, probably because his refactorings were far more aggressive (15 steps transforming a 17K-line monolith into 19 files) than the static-analysis-level cleanup in the SonarSource study. The two papers together suggest a dose-response relationship: modest cleanup yields modest savings; aggressive refactoring yields aggressive savings.

The knowledge chipper ([[The Knowledge Chipper]]) diagnoses the waste of agents rebuilding context from scratch each session. Fowler's experiment doesn't fix that — each sub-agent still starts fresh — but it reduces the *cost* of rebuilding: the agent reads 27K tokens of relevant code instead of 159K tokens of monolithic file. Same cold start, cheaper warm-up.

The constraint decay paper ([[Constraint Decay]]) finds that "convention-heavy frameworks are a trap for agents." Fowler's experiment shows one escape route: aggressive extraction of the repeating core into its own files and modules, so the agent reads less convention and more essence.

---

## Cross-References

- [[Code Cleanliness and Coding Agents]] — The closest empirical cousin: SonarSource's minimal-pair study found 7–8% token savings from static-analysis-level cleanliness. Fowler's 83% suggests a dose-response relationship where aggressive semantic refactoring dwarfs surface cleanup.
- [[The Knowledge Chipper]] — Diagnoses the context-rebuild waste that Fowler's refactoring partially addresses: cheaper cold starts through better-decomposed code.
- [[AI Slop Starts with the Codebase Itself]] — The argument that codebase structure is an AI productivity multiplier. Fowler's experiment provides the first hard numbers for that claim.
- [[Agent Coding Workflow]] — Fowler's experiment is a compound-engineering move: spend effort now on structure, bank savings on every future agent session.
- [[Constraint Decay]] — Convention-heavy frameworks are agent traps; Fowler's decomposition strategy is one escape route.
- [[Code-First Developer]] — Khalil Stemmler's five-phase model of developer craft; Fowler's human-guided refactoring suggests craft still matters — agents can execute refactorings but not discover them.
- [[Push Ifs Up And Fors Down]] — Alex Kladov's two heuristics (push conditionals to callers, batch data operations) are concrete refactoring patterns that produce the kind of structural decomposition Fowler measures. Both rules centralize decision-making and reduce the surface area agents must read — the same mechanism behind Fowler's 83% token savings.

---
*Sources: [[raw/refactoring-economic-benefit-html]], [[summary/refactoring-economic-benefit-html]]*
*Last updated: 2026-08-07*
