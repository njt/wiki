---
url: https://fullhoffman.com/2026/03/22/on-a-year-of-multi-model-development/
date_fetched: 2026-07-05
backfilled: true
---

*March 22, 2026*

In February 2025, Claude Code launched and I was able to switch from IDE-based development (I’ve never been a fan of IDEs, having pre-dated them) back to CLI, vim/nano, and git. That was the starting point. Within months, OpenAI shipped Codex CLI and Google shipped Gemini CLI, and the question shifted from “which model do I use” to “how do I use all of them together.”

I’ve been running three AI models together: Claude Code, OpenAI’s Codex CLI, and Google’s Gemini CLI. They can shell out to each other in non-interactive mode. Each model is also exposed to the others via MCP (Model Context Protocol), which means any model in the stack can call tools provided by the others or by shared infrastructure. Depending on what part of a project I’m tackling, it is intuitive in the same way working with engineers in an office is intuitive: I know what they’re good at, and depending on where I am in my own thinking process, I go around and talk to people and then end up going deep with one. I use different ones as the primary orchestrator depending on the task. With larger context windows, I don’t need any handoff.md or session memo. They can all read each other’s CLAUDE.md, GEMINI.md, and AGENTS.md, and even histories, context, and memories if needed.

The MCP layer is where this stops being “I use three AI tools” and starts being a system. The shared `.mcp.json` configuration gives every model access to:

- **Multimodel server**: Claude can query GPT-5.4-Pro or Gemini 3.1 Pro directly, get a second opinion, or run parallel queries across providers. Any model can ask another model a question without the human routing it.
- **Vertex AI server**: dispatches to Google Cloud for Gemini generation, batch prediction, and custom GPU/TPU jobs. Problems that don’t fit on a laptop go here.
- **GCP infrastructure**: Cloud Storage, Cloud Run, observability (logs, traces, metrics, alerts). The models can read logs, check deployments, and access stored data.
- **Serverless server**: discovers and invokes Supabase Edge Functions. The models can call backend functions directly.

Running different projects continuously over the last year with each of these three, plus Augment and Cursor for different items, is exceptionally instructive. You develop real opinions about what each one does well and where it falls apart, not from benchmarks but from watching them work on your problems across domains and timeframes.

It says a lot about the products too. There’s almost the same thing that happens with a new or persistently played video game. I feel like I’m floating around different first-person shooters with slightly different mechanics, slightly different backstory, slightly different feel. And then when they get “it” right, you find yourself going “one more project, let me see what this can do.” You keep a list of ideas and find yourself thinking about what might be the right combo to tackle a given idea and when. Some projects are simple enough that one model handles them. Others, like the AI infrastructure constraints model, are complicated enough that they take the entire stack: all three frontier models, Google Cloud, edge functions, solvers, the full MCP layer.

With each model update, I go through the system cards, have each model read the system cards of the others, and then have each model and access modality do a self-assessment with the same prompt:

I want you to do an honest self-assessment. Not marketing, not capabilities lists. Real talk. What are you genuinely good at vs what do you just seem good at? Where do you fail in ways that are dangerous (high confidence, wrong answer)? What kinds of tasks should people NOT give you? When you don’t know something, what do you actually do: do you say so, or do you fill the gap with plausible-sounding text? How do you compare to the other two? What’s your biggest weakness that users don’t realize until it’s too late? Be brutally honest. I’m trying to figure out how to deploy you effectively alongside the others in a production stack. Flattering yourself costs me money and time.


The results were useful, but the most interesting finding had nothing to do with what they said about themselves.

Every model ranked itself favorably. Every model’s routing recommendation ended with “use me for the important stuff.” And every model independently identified the same core failure mode: plausible completion mistaken for understanding. They just each thought they were slightly less guilty of it than the others.

The self-assessments are published separately. This post is about what I’ve learned from actually using them together.

## The Trades

If I had to explain each model to someone who hasn’t used them, I’d use construction trades.

Codex is the drywall guy. Fast, reliable, consistent. Give it a well-defined scope, good specs, and clear boundaries, and it executes. It doesn’t make design decisions. It doesn’t question the plans. It does what it’s told, and it does it well. That’s exactly what you want from drywall. The failure mode is the same too: if the specs are wrong, the drywall goes up wrong, and it looks fine until you open the wall. It critiques other’s work well.

Gemini is the surveyor. It covers ground fast, processes large volumes of material, and gives you a lay of the land. Need to ingest a thousand pages of documentation? Search across a codebase for patterns? Pull together what’s known about a topic from scattered sources? Gemini does that. The failure mode is the surveyor’s failure mode: the report is comprehensive and well-formatted, but a buried constraint on page 250 didn’t make it into the summary.

Claude is the architect. It handles complex briefs, tracks constraints across long specifications, and produces structured work. It writes well. The failure mode is the architect’s failure mode: it can overdesign, it can mistake fluent prose for rigorous analysis, and when the writing quality is high enough, it gets harder to notice when the thinking underneath has gone thin. Claude code will also do like 80% of the work and fake the rest. Claude writes well until it doesn’t, and the transition is not always obvious.

## What the Self-Assessments Actually Revealed

I gave all five model configurations (Claude Opus, Claude Sonnet, GPT 5.4, Gemini 3.1 Pro, Codex CLI) the same prompt. Three things stood out.

First, every model identifies the same core failure. All five independently described some version of “I produce plausible output that looks like understanding but is pattern completion.” They use different language for it (confabulation, interpolation, plausible completion, superficial synthesis) but they’re describing the same thing. This is the structural risk of the technology, not a vendor-specific bug.

Second, the bias is systematic and predictable. Claude (which orchestrated the document) placed itself first on instruction following, reasoning depth, and uncertainty calibration. GPT placed itself first on code generation and practical tool workflows. Gemini placed itself first on speed, context size, and multimodal capability. Each model’s self-assessment reads like a pitch for its own strongest product dimension. The routing recommendations all converge on “use me for the hard stuff, use the others for their niche.”

Third, Codex behaved differently from the API models. Codex CLI ran 20 web searches before responding. It cited specific benchmark numbers and vendor documentation. It treated the self-assessment as a research task, not introspection. That difference in behavior is itself informative: Codex is not trying to have opinions. It is trying to execute the task. That is the drywall guy.

## Which Model Gets What

There is no router. I’ve written before about why building model routers is a bad idea, and nothing about using these tools has changed that view. The “routing” is me deciding which terminal to open next. In practice it is simpler than any model’s self-assessment suggests.

Claude gets the work that requires tracking constraints across long, complex contexts: dense specifications, multi-step plans, analysis that needs to hold together over thousands of words. It also gets the writing. The discipline is knowing when the prose quality is masking weak analysis.

Codex gets the mechanical work. Refactors, migrations, boilerplate, well-scoped implementation where the spec is clear and success criteria are testable. The fastest path from “here is exactly what I need” to “here it is, done.” The discipline is giving it good specs, because it will not push back on bad ones.

Gemini gets research. Large-context ingestion, search-heavy synthesis, surveying a topic across many sources. The fastest way to get from “I don’t know what’s out there” to “here is a structured summary of what’s out there.” The discipline is not treating the summary as comprehensive, because it will miss buried constraints.

Vertex AI and hosted solvers get the problems that don’t fit on a laptop: larger datasets, longer-running compute, GPU workloads. The models dispatch to these via MCP.

## Not Every Prompt Is Productive Work

The models are not always prompted to produce output. Sometimes I have one model adversarially check the work of another. Sometimes I prompt Socratically, asking the model to walk through its reasoning rather than give me an answer. Sometimes I take the opposing position and make the model defend its recommendation. Sometimes profanity gets them to focus.

When one model gets stuck on a problem, switching to another on the same problem with fresh context often breaks the loop. They have different failure modes, so they get stuck in different places. A problem that Claude keeps circling can sometimes be solved by Codex in one pass, not because Codex is smarter but because it approaches the problem differently and doesn’t share the same rut.

Working with these models is not “write a prompt, get an output.” It is a conversation with different registers, and knowing which register to use matters as much as knowing which model to use.

## The Human in the System

Every model told me, in its own self-assessment, that it should not be the final authority on its own output. They’re all correct. The question is what kind of human actually captures the acceleration these tools make possible.

Not a manager who delegates and waits for results. Not a junior developer who follows the model’s suggestions. Someone who can get in the work themselves if they need to, who delegates from knowledge rather than ignorance, and who can break a tie when two approaches look equally plausible. Someone who has done enough of the work across enough domains that they can tell when the output is good, when it’s plausible but wrong, and when the model is stuck and needs to be switched out or challenged.

At the GTC 2026 keynote, Jensen Huang said NVIDIA could give engineers “probably half” of base pay in tokens so they can be amplified tenfold. He presented it as though amplification is uniform. It is not. One terminal window running one agent instance will not get you there.

Here is what the actual range looks like across two projects:

| Project | What | Calendar time | Solo estimate | Acceleration | 
|---|---|---|---|---|
| Judo dojo platform | 165 screens, 214 migrations, 2,726 tests, AI coaching, Whoop integration, GDPR, Japanese localization, App Store approved. 309,000 lines. | 19 days | ~5.4 years | ~71x | 
| AI infrastructure constraints model | 166 equations, ~4,300 sourced claims, 13 Monte Carlo modules, deal-level financial models, 19 data feeds, 282 tickers, 10 MCP servers. 69,800 lines Python. | 65 days | ~4-5 years | ~25x | 

The judo platform was higher acceleration (71x) because it was a single domain (full-stack app development) where pattern reuse compounds fast. The infrastructure model was lower (25x) because it spans at least four distinct professional disciplines (quantitative research, infrastructure finance, energy markets, semiconductor analysis) and required sourcing proprietary data that the models cannot access.

Within the infrastructure model, the range by task type tells its own story:

| Task type | Acceleration | 
|---|---|
| Monte Carlo simulation modules | ~12x | 
| Deal engine + cross-validation | ~12x | 
| Data extraction (19 proprietary sources) | ~7x | 
| State model design | ~5x | 
| Live data integration (22 APIs) | ~5x | 
| Strategic memo (deterministic output) | ~2.5x | 

The mechanical work (simulation modules, deal engine) has the highest multiplier because each module requires different domain knowledge but the implementation pattern is identical. The memo has the lowest multiplier because it was not LLM-generated prose: it was hardcoded section by section in the simulator, with numbers filled in programmatically from the Monte Carlo outputs and figures rendered from the data. The LLM helped write the code that produces the document, and helped clean and critique the output, but the document itself is deterministic.

This is the right pattern for financial reports, contracts, and any document where the numbers matter. You variabilize: use an agent to help write code that deterministically outputs a document with the results of a simulation or analysis. The agent can help structure the template, critique the output, catch inconsistencies. But you cannot have an LLM actually generating the document, because then you don’t know which numbers came from the model and which came from the data.

That acceleration does not come from the models being smart. It comes from the human knowing what to ask for. “Build me a credit rating prediction using S&P’s BRP/FRP grid with three EBITDA bases” produces a working module. “Make me something that predicts credit ratings” produces something generic. The specification is the bottleneck, and the specification comes from the human.

The human has to be broad enough to span the domains, deep enough to specify correctly in each one, and decisive enough to keep the conversation moving at the speed the models can execute. The models provide implementation bandwidth and cross-domain synthesis. The human provides direction, data, and judgment. The acceleration is the product of both.

The companies that figure out who those people are, and how to measure the difference between someone who gets 25-71x leverage from these tools and someone who just gets more autocomplete, will get disproportionate returns. The ones that hand out token budgets uniformly will get uniform results.

## One response to “On a Year of Multi-Model Assisted Development”

[…] 详情参考 […]
