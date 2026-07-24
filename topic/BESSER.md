# BESSER

An open-source, academically-led low-code platform that combines model-driven engineering with AI assistance. Model a system once using diagrams, forms, or natural language, then generate APIs, databases, back-ends, and AI agents across 15+ technology stacks. MIT-licensed, funded by an FNR PEARL grant, built by 11 researchers at LIST and the University of Luxembourg.

---

## Key Quotes

> "From a single model, BESSER produces working software: APIs, databases, full back-ends, AI agents, and deployment manifests."

This is the Model-Driven Engineering (MDE) promise that OMG's MDA made in the early 2000s, updated with AI as both modeling assistant and generation target. The "model once, generate everywhere" pitch is conceptually unchanged from two decades ago — what's new is the quality of the generators and the AI lowering the barrier to creating the model in the first place.

> "BESSER combines AI and low-code to accelerate modeling and to generate software systems that embed AI components."

The bidirectional AI story is the actual differentiator here: AI helps you build the model (image-to-model, text-to-model via chatbot), and the generated output can itself include AI components (PyTorch models). Most low-code platforms only go one direction — AI-assisted building — but stop short of generating AI-infused output.

> "Better Software, Faster"

The tagline is intentionally bland, but it captures the value proposition in an era where "faster" is table stakes. The real question: does "better" mean better-designed (the MDE argument) or just faster-to-ship (the low-code argument)? BESSER wants both, but the academic provenance suggests the former matters more to the team.

---

## Key Themes

#tool #low-code #model-driven-engineering #ai-assisted #open-source

- **Model-driven engineering, again** — MDE has been the perpetual next-big-thing since the 1990s. It works in constrained domains (automotive, aerospace, telecom) but has never crossed the chasm to general application development. BESSER is betting that AI-assisted modeling + free online access + 15 concrete generators changes the adoption math. The risk is that AI coding agents ([[Claude Code Mastery]], [[Grok Build]]) are eating the low-code space from above — if you can describe what you want in English and get working code, why model it first?

- **Academic rigor as differentiation** — 53 peer-reviewed papers, 2 EU Horizon projects, an FNR PEARL grant. This is not a startup chasing VC money; it's a research project with an open-source release. The rigor shows in the multi-dimension modeling (class diagrams, state machines, deployment specs) but also raises the sustainability question: academic tools routinely die when grant funding ends. The MIT license mitigates this somewhat — the code survives even if the lab moves on.

- **Free online editor as adoption strategy** — No-install, browser-based modeling removes the single biggest barrier that killed previous MDE tools (Rational Rose, Eclipse Modeling Framework, etc.). [[Budibase]] and [[Xano]] use the same playbook — get people building before they even think about deployment. But an editor is only as good as its onboarding; the gap between "here's a blank canvas" and "I built something useful" is where most low-code tools lose people.

- **Multi-target generation with AI output** — 15+ generators spanning Python, Java, React, and PyTorch is genuinely ambitious. The PyTorch generator is the most interesting: it's one thing to generate a CRUD API (many tools do this), another to generate a trained model pipeline. If this works well, it positions BESSER as a bridge between traditional software engineering and ML engineering — two disciplines that currently use entirely different toolchains.

- **The low-code/agent collision** — The most interesting tension is unstated: BESSER competes with AI coding agents ([[Components of a Coding Agent]], [[Agent Coding Workflow]]) for the same workflow step. Why model a system in a visual editor when you can describe it to Claude Code and get a working app? BESSER's answer is implicit: models are more precise than prompts, generators are more predictable than LLM output, and the generated code benefits from the constraints that make it maintainable. Whether this is true depends on whether the models are actually easier to create and maintain than just writing specs ([[Specifications as the Product]], [[AI Agents Need Clear Specs]]).

---

## Critical Analysis

BESSER is the most credible attempt I've seen to resurrect model-driven engineering for the AI era, but it faces three hard problems.

**First, the modeling bottleneck.** The fundamental issue with every MDE tool is that creating a precise model is about as hard as writing the code — you're just doing it in a different notation. BESSER tries to solve this with AI-assisted modeling (image-to-model, text-to-model), but that introduces a verification problem: if AI generates your model and then generators produce your code, you're two translation steps removed from what actually runs. Debugging across that chain — "why is the API returning 500?" → "which part of the model is wrong?" → "which prompt generated the wrong model?" — is a support nightmare.

**Second, the agent competition.** The window for low-code platforms might be closing faster than anyone expected. When [[Claude Code Mastery]] can scaffold a full-stack app from a CLAUDE.md and a conversation, the value of a visual modeling layer shrinks to the gap between what you can describe in words and what you need to specify precisely. BESSER bets that gap is large. The trajectory of coding agents suggests it's shrinking fast. The platform that survives isn't the one with the best generators — it's the one where the feedback loop (change model → see result) is fastest. On that metric, direct code generation from prompts currently beats model→code pipelines.

**Third, the academic sustainability trap.** 53 papers and EU funding look impressive on a website, but academic software projects have a well-established lifecycle: publish, release, move on. The MIT license is smart — it means the community can fork and maintain even if the lab doesn't — but community adoption of abandoned academic tools is rare. The exceptions (LLVM, Coq, Isabelle) became infrastructure. BESSER is positioning as a product, not infrastructure, and products need ongoing investment.

**What's genuinely interesting:** The bidirectional AI story. Most low-code platforms treat AI as an input accelerant. BESSER treats AI as both input (modeling assistant) and output (generated AI components). The PyTorch generator means you can model a system that includes a recommendation engine or classifier as a first-class architectural component, not as something you bolt on later. That's a genuinely novel integration point, and if it works well, it's a defensible differentiator — coding agents can generate ML code, but they don't model ML components as part of an architectural whole.

**Bottom line:** Worth watching if you care about the collision between MDE and AI-assisted development, or if you're evaluating tools at the "I need a full-stack app and don't want to write boilerplate" tier. But for most developers already using coding agents, BESSER solves a problem that agents are rapidly making obsolete — unless you buy the argument that models are better specs than natural language, in which case this is a hedge against prompt drift.

---
*Sources: [[raw/besser-pearl]]*
*Last updated: 2026-07-25*
