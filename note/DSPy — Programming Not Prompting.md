# DSPy — Programming Not Prompting

DSPy is a Python framework from Stanford NLP that operationalizes the thesis that natural language prompts are structurally unfit as a specification medium for LLMs — and that the fix is not better prompt-writing but a different paradigm entirely: typed signatures compiled into optimized prompts by the framework, not the human. The tagline says it: *Program, don't prompt, your LLMs.*

---

## The Three Abstractions

DSPy decomposes the LLM-task problem into three layers:

**Signatures** — Typed input/output declarations replace prose prompts. Instead of "You are a helpful assistant that extracts events from text. Output JSON with fields…" you declare `text -> events: List[Event]`. The signature is portable across models, maintainable under change, and serves as the durable specification layer. The prompt becomes a disposable artifact the framework generates.

**Modules** — Execution strategies layered on top of signatures, independent of the task definition. A module might add chain-of-thought reasoning, run an ensemble of calls, invoke tools, or add a REPL loop. Crucially, modules compose with signatures — you change *how* a task executes without rewriting *what* the task is. This is the separation of concerns that hand-crafted prompts structurally prevent.

**Optimizers** — Given examples and a scoring function, DSPy automatically tunes the prompts that underpin signatures and modules until quality converges. This is the "search, don't write" prescription from [[Prompt Debt]] made concrete: the prompt surface area is too large for manual optimization, but a framework with a metric can explore it systematically.

> *"Give DSPy examples and a scoring function. It tunes your prompts automatically until quality converges."*

## Key Themes

#tool #concept #pattern

- **Signatures as the durable artifact.** In a world where code is disposable ([[Specifications as the Product]]), DSPy's signatures are the spec — typed, portable, and independent of any particular model's prompt idiosyncrasies. Prompts become compiler output.
- **Automatic prompt optimization.** DSPy's optimizers implement what Breunig called for in [[Prompt Debt]]: algorithmic search over prompt space rather than human craft. The optimizers close the loop between specification and execution without the human in the middle.
- **Modularity through separation of concerns.** Signatures define *what*, modules define *how*, optimizers tune the execution. Each layer can evolve independently — the same signature can be executed with different modules and optimized for different models without cascading changes. This is the architectural property that hand-written prompts cannot achieve.
- **Academic origin, production trajectory.** Stanford NLP → research community → production systems at "companies you've heard of." This is the same path that transformers, BERT, and RAG took — and it suggests DSPy's primitives may become standard infrastructure rather than a niche framework.

## Critical Analysis

**The homepage is the thesis, not the documentation.** The source is a product landing page — it states the vision and names the abstractions but doesn't show them working. This is worth noting because the gap between "typed signatures are the right abstraction" and "DSPy makes typed signatures work in practice" is where most framework pitches live. The claim that optimizers "tune your prompts automatically until quality converges" is doing a lot of work — convergence to what? Under what conditions? With how many examples?

**The hard part is the scoring function.** DSPy's optimizers need a scoring function — something that measures output quality. For classification and extraction tasks, this is straightforward (accuracy, F1, exact match). For the open-ended generation tasks that characterize most real-world LLM use — summarization, creative writing, explanation — the metric *is* the hard problem. Breunig's [[Prompt Debt]] diagnosis applies here: "specify behavior with measurements" is the right principle, but for many tasks the measurement is harder than the prompt.

**DSPy and the agentic gap.** The homepage focuses on structured task execution — signatures with typed inputs and outputs. This maps cleanly to classification, extraction, and structured generation. It is less clear how it maps to the open-ended, tool-using, multi-turn agent loops that characterize modern coding agents. The modules abstraction (ensembles, tools, REPL) gestures at this, but the homepage doesn't show DSPy being used for the kind of system prompts that dominate agentic applications. The gap between "DSPy can optimize a classification prompt" and "DSPy can replace Claude Code's system prompt" is vast — and the homepage doesn't address it.

**The compile-to-prompt metaphor is powerful but incomplete.** DSPy's framing — signatures compile to optimized prompts — draws an analogy to compilers that is rhetorically effective but mechanically imperfect. A compiler operates on a formal grammar with deterministic semantics; DSPy operates on natural language with probabilistic semantics. The "compiler" is searching a space of prompt candidates and scoring them against examples — it's optimization, not compilation. The distinction matters because compilers guarantee correctness; optimizers guarantee improvement over a baseline, which is weaker.

**What this means for the wiki.** DSPy adds a concrete framework name to several threads already running through the wiki. [[Prompt Debt]] names the problem DSPy solves. [[OpenAI Structured Outputs]] does for output formatting what DSPy does for task specification. [[Harness Engineering]] provides the vocabulary (feedforward vs. feedback, computational vs. inferential) that DSPy's architecture maps onto. DSPy is the input-side structured-specification analog to Structured Outputs' output-side schema enforcement — and together they define a stack where neither prompts nor output parsing are the developer's problem.

---

*Sources: [[raw/dspy-ai]], [[summary/dspy-ai]]*
*Last updated: 2026-08-08*
