---
url: https://2389.ai/posts/the-dark-factory-is-a-dot-file/
date_fetched: 2026-07-05
backfilled: true
---

# The Dark Factory Is a .dot file

← All PostsSo StrongDM published a natural language spec for building a coding agent pipeline runner. Dan Shapiro built one. We built three. All of them — independently, in two languages, by different people with different goals — landed on the same three-layer architecture.

I keep coming back to that. Not the code. The convergence. That’s the weird part.

## The attractor pattern

In February, StrongDM open-sourced attractor: three natural language specs describing a unified LLM client, a coding agent loop, and a DOT-based pipeline engine. The specs aren’t code. They’re prose. About 5,700 lines of it. Detailed enough that you can hand them to a coding agent and say “build this.” And it will.

The name is borrowed from dynamical systems — an attractor is a state a system tends to evolve toward. StrongDM’s bet is that these specs describe a design so natural for the problem that independent implementations will converge on it. Bold claim! But uh, that’s exactly what happened.

They also released AttractorBench, which is a benchmark for measuring how well coding agents implement systems from natural language specs. It’s tiered — smoke test, then a unified LLM SDK, then a coding agent loop, then the full pipeline runner. Language-agnostic. Agents pick their own implementation language. The only contract is `make build`, `make test`, and a conformance suite against a mock LLM server. No real API calls. Deterministic verification. Cost-aware scoring. It doesn’t just ask “did you build it?” It asks “how well did you follow the spec, and what did it cost?”

Dan Shapiro had been thinking about this progression for a while. In January he published “The Five Levels: from Spicy Autocomplete to the Dark Factory”, borrowing the NHTSA’s driving automation levels for AI-assisted coding. Level 0 is vi. No AI. Every character yours. Level 2 is where most “AI-native” developers are living right now — pair-programming with a model, feeling productive. Level 4 is where you’ve become a PM. You write specs, argue about specs, leave for 12 hours, check if the tests pass.

Level 5 is the dark factory. Lights off. Nobody reviews the code. Nobody even looks at it.

The term “dark factory” comes from manufacturing — a factory run by robots where the lights are off because robots don’t need to see. Specifically Fanuc Robotics in Japan around ~2003.

Applied to software, it’s kind of chilling and kind of exciting in equal measure.

After StrongDM’s demo, Shapiro wrote “You Don’t Write the Code. You Don’t Read the Code Either.” and then went and built Kilroy. Local-first Go CLI, runs attractor pipelines in isolated git worktrees, uses CXDB for run history and checkpoint recovery. Another independent build. Same three layers.

## Dorodango, or: why we built three

Jesse Vincent wrote a blog post about dorodango — the Japanese art of polishing a ball of mud into a high-gloss sphere. Wikipedia’s disambiguation note for “mud ball” redirects to “Big Ball of Mud,” the software anti-pattern. Jesse leaned into it. I love this framing.

His point: codegen software is disposable. You spec it carefully, hand it to an agent, polish what comes out. When the result is fundamentally wrong, you don’t debug your way to salvation. You throw it away and rebuild from the spec. He described waking up to find an agent’s end-to-end test recording named `e2e-test-full-run-33.mp4`. Runs 1 through 32 were the agent working through problems one by one. Run 33 worked. Pretty cool.

This is the mental model that let us build three attractor implementations without thinking twice about it. Software is cheap now. Specs are the expensive part.

Mammoth and Smasher were built in parallel from the same spec. Mammoth, in Go, scope-crept in the best possible way — it grew a 21-rule DOT linter, fan-in nodes with configurable join policies (all-success, majority, first-success), verification nodes that run shell commands at zero token cost, and a 5-phase node lifecycle. It became this whole spec engine thing. Really cool, but also really big. Smasher, in Rust, stayed lean: five crates from LLM client to web dashboard, an HTMX frontend with live SSE streaming and graph visualization, six built-in agent tools, and a `smasher chat` REPL for when you just want to talk to the thing. Smasher is the one that actually gets used day-to-day.

Tracker came later. Simpler. Go, bubbletea TUI, automatic checkpointing to `.tracker/runs/`, retry with backoff. A weekend-scale implementation that still converges on the same shape.

Because they all do. Every single one of these — Kilroy, Mammoth, Smasher, Tracker — ends up with three layers:

| Layer | Kilroy (Go) | Mammoth (Go) | Smasher (Rust) | Tracker (Go) | 
|---|---|---|---|---|
| LLM Client | Provider adapters | `llm/`— unified OpenAI/Anthropic/Gemini | `smasher-llm`— streaming, retries, provider quirks | Provider client with trace introspection | 
| Agent Loop | Coding agent with tool dispatch | `agent/`— steering, loop detection, subagents | `smasher-agent`— 6 tools, steering rules, subagents | LLM-powered nodes with context injection | 
| Pipeline Engine | DOT parser, CXDB checkpoints, worktree isolation | `attractor/`— DOT parser, graph engine, node handlers | `smasher-attractor`— winnow parser, tokio broadcast | DAG walker, checkpointing, human gates | 

Nobody coordinated this. The spec pulled them there. That’s the attractor.

## The pipelines are the product

Ok so here’s the thing that’s been bugging me. The factory implementations are open source and multiplying. Great. But the pipeline files — the DOT graphs that describe what the factory actually builds — are mostly private. Everyone’s sharing the engine and hiding the blueprints.

one quick clarification - for my entire life a dotfile was .bashrc, or a .vim or whatever. we are talking about a graphviz .dot file. I first learned about it from Justin when he first showed me his factory. It is the grandparent of mermaid, sorta.


A pipeline DOT file is a reusable blueprint. It describes the workflow: which steps need an LLM, which need a human gate, where to fork into parallel branches, what verification commands to run before proceeding. Standard Graphviz syntax. Nothing proprietary. And honestly? The pipelines are way more interesting than the runners.

We’ve been writing a lot of these, and two very different styles have emerged.

Here’s the first — a vulnerability analyzer (`vulnerability_analyzer.dot`) from Tracker’s examples:

```
digraph VulnerabilityAnalyzer {
  graph [
    goal="Run a deterministic static vulnerability scan against a known
          vulnerable application and emit a report with evidence.",
    rankdir=LR,
    default_max_retry=1
  ];
  Start [shape=Mdiamond];
  Exit  [shape=Msquare];
  CloneTarget [
    shape=parallelogram,
    label="Clone vulnerable target",
    tool_command="set -eu
      mkdir -p .ai/vuln
      git clone --depth 1 https://github.com/digininja/DVWA.git .ai/vuln/target
      printf 'ready'"
  ];
  StaticScan [
    shape=parallelogram,
    label="Run static scan",
    tool_command="set -eu
      rg -n 'mysql_query\\(|eval\\(|shell_exec\\(' .ai/vuln/target > .ai/vuln/findings.txt
      printf 'scanned'"
  ];
  WriteReport [
    shape=parallelogram,
    label="Write vulnerability report",
    tool_command="set -eu
      count=$(wc -l < .ai/vuln/findings.txt)
      echo \"# Report\" > .ai/vuln/report.md
      echo \"Finding count: $count\" >> .ai/vuln/report.md
      printf 'report_written'"
  ];
  Start -> CloneTarget -> StaticScan -> WriteReport -> Exit;
}
```
Every node is a `tool_command` — just a shell script. No LLM calls. No token cost. Deterministic, reproducible, runs in seconds. The graph *is* the program. It rules.

Now compare that to the other style, from Mammoth’s examples. This is `build_pong.dot`, a pipeline that builds a Pong game:

```
digraph build_pong {
    graph [
        goal="Build a two-player Pong TUI game in Go",
        retry_target="implement",
        default_max_retry=3,
        model_stylesheet="
            * { llm_model: claude-sonnet-4-5; llm_provider: anthropic; }
            .code { llm_model: claude-opus-4-6; llm_provider: anthropic; }
        "
    ]
    start [shape=Mdiamond]
    done  [shape=Msquare]
    plan      [label="Plan",      class="planning", prompt="Plan the architecture..."]
    scaffold  [label="Scaffold",  class="code",     prompt="Initialize Go module..."]
    implement [label="Implement", class="code",     prompt="Write the full game...",
               goal_gate=true, max_retries=3]
    compile   [label="Compile",   class="code",     prompt="Run go build and go vet..."]
    compile_ok [shape=diamond, label="Compiles?"]
    review    [label="Review",    class="review",   prompt="Review all generated code..."]
    start -> plan -> scaffold -> implement -> compile -> compile_ok
    compile_ok -> review    [label="Pass", condition="outcome=success"]
    compile_ok -> implement [label="Fail", condition="outcome=fail"]
    review -> done          [label="Pass", condition="outcome=success"]
}
```
This style is a build recipe. It leans on LLMs for every step — planning, scaffolding, implementation, review. There’s a `model_stylesheet` that maps CSS-like selectors to providers, which is clever as hell. It’s also expensive, slow, and nondeterministic.

We’ve come to prefer the first style. Tool nodes with shell commands for anything that can be deterministic. LLM nodes only where you actually need reasoning. The vulnerability analyzer runs in seconds and costs nothing. The Pong builder might take 20 minutes and $15 in API calls, and you won’t get the same game twice. Guess which one I want to run at 2am from my phone.

The most interesting pipelines combine both: deterministic tool nodes for setup, validation, and deployment, with LLM nodes only at the points where you genuinely need a model to think. Tracker’s sprint execution pipeline (`sprint_exec.dot`) does this — shell scripts for ledger management and build validation, LLM nodes for implementation and review, with three models critiquing each other’s reviews in parallel fan-out before a final synthesis decides whether to ship or loop back.

And then there’s `dotpowers.dot` — our attempt to clone Jesse’s Superpowers into a DOT file. The goal is to encode an entire software development lifecycle into a single DOT file. 53 nodes across 7 phases: brainstorm with a human, write a design brief, draft and audit a plan, set up a project, implement tasks in a TDD loop with escalation paths, run multi-model review with cross-critique, and finish by merging, creating a PR, or discarding. Human gates at every decision point. Three different LLM providers doing adversarial review. Retry budgets so the pipeline fails gracefully instead of looping forever.

One file. Standard DOT syntax. Runs on Mammoth. It’s the kind of thing that only makes sense once you stop thinking of the pipeline as a script and start thinking of it as a process definition. Less shell script, more BPMN diagram. It’s weird. I kind of love it.

## Share your dot files

The factory code is dorodango — polish it, throw it away, rebuild from spec. The pipeline files are the durable artifact. They’re the part worth sharing.

So share them! What does your “audit a Rails app” pipeline look like? Your “onboard a new engineer” graph? Your “ship a mobile release” DAG? Drop your `.dot` files in a gist, post them on your blog, open a PR somewhere. The dark factory pattern is real, it’s reproducible, and agents can build the factory from spec.

The question isn’t how to build the factory anymore. It’s what to build with it.
