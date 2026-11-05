# Droid 1.0: Building a Frontier Harness (Factory)

Factory's Droid 1.0 announcement, framed as a product launch but structured as an engineering paper on harness design: programmatic tool calling (a sandboxed Script tool making hundreds of tool calls per model request), an internal Agent Factory that tunes the harness against every model family, prompt-cache discipline, base-prompt dieting, bounded-autonomy rules, and latency profiling — with head-to-head benchmarks against Claude Code and Codex.

---

## What the source argues

An agent harness is "everything around the model: the prompts, the tools, how tool results come back, and the loop that runs it all." Factory's bet is that the harness is the decisive layer and that tuning it across *every* model — rather than against one — produces improvements that actually generalize. The largest single change is programmatic tool calling: the agent writes JavaScript in a sandbox with no file, network, or system access, and the program performs the tool calls, keeping raw outputs as in-program variables so the model only sees what the program prints.

Key quotes, with commentary:

> "Every turn re-sends instructions, tools, and history."

The economics that motivate everything else in the piece. Once you accept that the per-request prefix dominates cost in long-horizon work, the rest follows naturally: keep the prefix cached (prompt-cache monitor, append-only discipline), shrink the base prompt (2 tool definitions up front instead of 11, 37% smaller), and amortize it over many calls inside one program (Script mode).

> "It makes the agent more reliable: checks and loops run deterministically in code, eliminating the chance that a model could be lazy and drop a case despite originally planning it."

The most under-hyped claim in the piece. Everyone quotes the token savings; the quality argument is stronger. A plan written as a program can't be lazily abandoned mid-execution the way a plan held in context can — the Linear cleanup compared 15 fields after every write and kept exact counts because code enforced it. This is verification moved out of the model's willpower and into the loop.

> "Working across models ensures the strongest improvements generalize and hold up beyond the model that inspired them."

The methodological core of the Agent Factory. The line-numbering example is the honest admission that makes it credible: numbering every read file cut Opus's cost 22% but *raised* cost 8% on GLM and 32% on Kimi. No single change moved every model; goals were met by combinations. This is a rare public acknowledgment that harness tuning is per-model work even for a model-independent harness.

> "Under 0.1% of the tool results reached its context."

The CI sweep is the piece's best exhibit: 550 tool calls, 2 requests, the model saw a five-number summary. This is context engineering as architecture rather than as prompt-craft — the interesting question isn't what to put in context but how to ensure the garbage never gets there at all.

## Key themes

**#tool — Droid 1.0**: Factory's model-independent coding agent harness, with Factory Router for automatic model selection and an SDK exposing the same daemon as the CLI.

**#pattern — Programmatic tool calling (PTC) / Script mode**: The agent's plan *is* the program. Raw data stays in-program; only printed output reaches context; independent calls run concurrently; long jobs run in the background with wake-ups instead of polling.

**#concept — Harness tuning as multi-model experimentation**: The Agent Factory loop — automation adds every new model, agents propose changes with stated hypotheses, controlled experiments measure every change on every affected model family, engineers review evidence and ship. Automation proposes; humans gate.

**#concept — Prompt-cache hygiene as first-class engineering**: 98% cache reuse in production, a monitor that names the part of any request that broke the prefix, tools appended at request end so mid-session MCP loads don't invalidate the cache, retries pinned to the provider that already holds the cached request.

## Opinionated take

This is the most thorough public account of harness engineering as an industrial process, and the benchmark methodology is genuinely careful (same tasks both harnesses passed in all three trials, list-price costs, disclosed quirks like Cursor's token undercounting and Devin's ask-a-question behavior). That care makes it trustworthy in a genre — vendor benchmark posts — where almost nothing is.

Two caveats. First, the head-to-head numbers are self-reported and only against tasks *Droid also passed* — a selection that flatters whoever controls the comparison. The cost-per-passed-task framing is legitimate but still vendor-shaped. Second, the "model-independent by design" claim sits in tension with the evidence: per-model tuning of prompts, tool descriptions, and Script-mode defaults is exactly what the Agent Factory does, and the line-numbering example shows changes are not portable across models. Model-independence here means *the harness runs on every model*, not that one configuration serves them all — a distinction the marketing language blurs and the engineering section quietly confirms.

The deeper significance is that PTC becoming the default validates what a handful of research harnesses bet earlier: the model-as-programmer framing. When the model writes the workflow as code, "harness" and "program" start to converge, and the loop's remaining job is scheduling, caching, and recovery — all deterministic infrastructure.

## How this fits the wiki

- [[Prime Agent (RLM Harness)]] makes the same bet from the research side — treating the model as a programmer whose program manages context and sub-agents. Droid 1.0 is the production-grade, multi-model validation of that thesis: the strongest evidence yet that programmatic control of tool calls generalizes across frontier families, and a nuance Prime Agent lacks — Factory's admission that each model needs its own tuning.
- [[Harness Engineering (OpenAI)]] documents harness engineering as discipline; Factory's Agent Factory strengthens its automation-proposes-humans-gate pattern with a concrete industrial loop — 20+ tracked models, hypotheses stated per change, controlled experiments per model family — while complicating its optimism by showing even well-run loops produce per-model changes that don't transfer.
- [[Model Routing Is Simple Until It Isn't]] treats routing as multi-objective optimization under real infrastructure constraints; Factory Router is the missing vendor-side counterpart, and the prompt-cache findings (cache warmth as the dominant cost variable) independently confirm that article's sharpest claim from a different producer.
- [[Everyharness]] catalogs the plugin-format fragmentation across twelve harnesses including Factory Droid; this piece is the vendor's own view of the same harness, showing why compilation-per-harness is necessary — the harness layer is where every model- and tool-specific adaptation lives.
- The topic page [[Agent Memory and Context]] collects context engineering; Droid's CI sweep (0.1% of tool results in context) and 16K-token compaction budgets are among the strongest concrete illustrations of context as the financial lever.

---
*Sources: [[raw/frontier-harness]], [[summary/frontier-harness]]*
*Last updated: 2026-11-06*
