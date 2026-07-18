---
url: https://drive.google.com/file/d/1IR7CddF_2FyQo_PdfBNTaEA50EGiVt2r/view
title: "The New SDLC with Vibe Coding: From ad-hoc prompting to Agentic Engineering"
author: Addy Osmani, Shubham Saboo, and Sokratis Kartakis
date_fetched: 2026-07-18
date_published: 2026-05
---

# The New SDLC With Vibe Coding: From ad-hoc prompting to Agentic Engineering

Authors: Addy Osmani, Shubham Saboo, and Sokratis Kartakis

## Introduction

For most of computing history, programming has been an act of translation: understand the problem in human terms, design a solution in abstract terms, then render it in syntax a machine can execute. Each step introduces friction. That friction is now collapsing.

Software engineering is undergoing its most significant transformation since the introduction of high-level programming languages. For decades, the developer's primary interface with the machine has been syntax: curly braces, semicolons, type annotations, and the precise grammar of programming languages. That era is ending.

A new paradigm has arrived in which developers express what they want to build rather than how to build it. The machine handles implementation. The human provides intent, architecture, and judgment. This isn't a distant future — it's the daily reality for a rapidly growing number of professional developers. As of early 2026, 85% of professional developers regularly use AI Coding Agents, 51% use them daily, and an estimated 41% of all new code is AI-generated.

This shift didn't happen overnight. It began with autocomplete — simple token prediction in the editor. Then came inline code suggestions that could complete entire functions. Next, chat-based interfaces allowed developers to describe features in natural language and receive working implementations. Now, fully autonomous agents can clone repositories, plan multi-file changes, execute them in sandboxed environments, run tests, and submit pull requests — all without a human typing a single line of code.

The implications for the software development life cycle (SDLC) are profound. Every phase — from requirements gathering to deployment to maintenance — is being reshaped by AI capabilities. But this transformation isn't uniform or simple. The spectrum ranges from casual "vibe coding," where a developer prompts an AI and accepts whatever comes back, to disciplined "agentic engineering," where AI acts as a powerful implementation engine within carefully designed systems of constraints, tests, and feedback loops, with humans retaining oversight over architecture, correctness, and quality.

The distinction matters. Telling a CTO that your team is vibe coding their payment processing system will, and should, raise alarm bells. Telling that same CTO that your team practices agentic engineering, with AI handling implementation under human-designed constraints while test coverage ensures correctness, is a fundamentally different conversation.

This paper provides the foundation for that conversation. We trace the spectrum from casual vibe coding to disciplined agentic engineering, examine how the developer's role is shifting from writing code to exercising judgment — from conductor to orchestrator — and lay out what it takes to adopt these tools in ways that produce software you can actually depend on.

## Why This Paper, Why Now

New tools, capabilities, and paradigms emerge weekly. Engineering teams need a framework for making sense of this landscape — not a snapshot that will be outdated in months, but a set of principles and mental models that will remain useful as the specific tools evolve.

## Who This Paper Is For

This paper is for software engineers, engineering managers, architects, and technical leaders who want to understand how AI is reshaping the SDLC and adopt these new capabilities without sacrificing the discipline that production software demands. We assume familiarity with modern software development practices but not with the specifics of AI or machine learning.

## The Shift from Syntax to Intent

### AI Agents: A Quick Refresher

An AI agent is a software system that perceives a goal, plans steps to reach it, takes actions through tools, observes the results, and iterates until the goal is met or it hits a stopping condition. Where a chatbot produces a response and waits for the next prompt, an agent runs its own loop.

Every agent is built from five parts:
- **The model** is the reasoning engine. It reads the current context, decides what should happen next, and produces the next thought, the next tool call, or the next message.
- **Tools** connect the model to the world. They include APIs the agent can call, code it can execute, databases it can query, and other agents it can delegate to.
- **Memory** is the state. It allows the agent to recall past interactions, retrieve project-specific rules, and retain context across sessions so it never starts from a blank slate.
- **Orchestration** is the code that runs the loop. It assembles the context for each model call, dispatches tool calls, captures their results, and decides whether to continue.
- **Deployment** is what turns the prototype into a service: hosting, identity, observability, and the production infrastructure the agent runs on.

### What Is Vibe Coding?

In February 2025, Andrej Karpathy posted a description of a new way of programming that resonated widely across the software engineering community. He described an approach where you "fully give in to the vibes, embrace exponentials, and forget that the code even exists." In this mode, a developer describes what they want in natural language, accepts the AI's output, and when something breaks, copies the error message back into the prompt and asks the AI to fix it.

The term went viral because it captured something real: many developers were already working this way but hadn't had language for it. Within months, "vibe coding" became a common descriptor for any AI-assisted development workflow, which created confusion.

By early 2026, Karpathy himself acknowledged that the original framing was too narrow, introducing the term "agentic engineering" to describe the more disciplined end of the spectrum.

### The Spectrum: Vibe Coding to Agentic Engineering

Rather than treating vibe coding and agentic engineering as a binary, the authors find it more useful to think of them as endpoints on a spectrum. The key differentiator is not whether you use AI — it's how much structure, verification, and human judgment surrounds the AI's output.

| Dimension | Vibe Coding | Structured AI-Assisted Coding | Agentic Engineering |
|---|---|---|---|
| Intent specification | Casual natural language prompts | Detailed prompts with examples and constraints | Formal specs, architecture docs, memory files |
| Verification | "Does it seem to work?" | Manual testing, spot-checking | Automated test suites, CI/CD gates, LM judges |
| Codebase understanding | Minimal; developer may not read the generated code | Selective review of critical paths | Comprehensive review of architecture; AI handles implementation details |
| Error handling | Copy-paste error messages back to the AI | Developer diagnoses root cause, AI implements fix | Agents self-diagnose within defined bounds; humans handle architectural issues |
| Appropriate scope | Prototypes, scripts, personal projects, hackathons | Features within established codebases | Production systems, team-scale development |
| Risk profile | High; acceptable for disposable code | Moderate; human judgment at key checkpoints | Low; systematic verification at every stage |

The single biggest differentiator between the two ends is how outputs get verified. In vibe coding, verification is optional; the developer runs the code and checks if it seems right. In agentic engineering, two mechanisms work together. Tests verify the deterministic parts of the system. Evaluations, or evals, verify the parts that are not deterministic. Tests are checked by code; evals are checked by labelled datasets, scoring rubrics, and LM judges. Without both, the practice is always vibe coding, regardless of how sophisticated the prompts are.

## Context Engineering: The Real Skill

As the field has matured, a key insight has emerged: the quality of AI-generated code depends less on the cleverness of your prompts and more on the quality of the context provided. This realization has given rise to the concept of context engineering.

Developers must consider six primary types of context:
- **Instructions:** The agent's core role, goals, and operational boundaries.
- **Knowledge:** Retrieved documents, architectural diagrams, and domain-specific data.
- **Memory:** Short-term session logs (what just happened) and long-term persistent state (what the project is).
- **Examples:** Few-shot behavioral demonstrations and codebase reference patterns.
- **Tools:** The precise definitions of the APIs, scripts, and external services the agent can invoke.
- **Guardrails:** Hard constraints, formatting rules, and safety validations.

In AI code generation, context engineering involves carefully balancing which of these six elements the agent possesses upfront versus what it can retrieve on demand. This creates a critical separation between static and dynamic context.

**Static context** is always loaded: system instructions, rule files (AGENTS.md, CLAUDE.md, GEMINI.md), global memory, and persona definitions. It defines who the agent is and how it behaves. Static context is expensive because every token is present in every interaction, regardless of relevance.

**Dynamic context** is loaded on demand: skill instructions triggered by task matching, tool results retrieved during execution, documents fetched from RAG pipelines, and windowed session history. Dynamic context is efficient because the agent pays the token cost only when the information is needed.

The design decision of what belongs in static context versus dynamic context is a genuine engineering trade-off. Too much static context wastes tokens and dilutes signals. Too little means the agent forgets critical rules. The best systems treat this boundary as a first-class architectural decision, reviewed and versioned like any other configuration.

### Agent Skills

The most powerful pattern for managing dynamic context is Agent Skills: structured, portable packages of procedural knowledge that the agent loads only when the task calls for it.

Rather than embedding every piece of specialized knowledge into the agent's system prompt, skills allow the agent to remain a lightweight generalist that flexes into specialist roles on demand through progressive disclosure. The agent sees only lightweight metadata at startup, loads full instructions when a task matches, and pulls deep reference material only when explicitly needed.

Agent Skills have seen rapid adoption because they solve four problems that have plagued AI agent development:
- Context rot from overloaded prompts
- Absence of procedural memory for LLMs
- Operational overhead of multi-agent architectures
- Need for portability across tools and vendors

The shift from "prompt engineering" to "context engineering" reflects a deeper truth about working with AI. Models don't need cleverly worded instructions as much as they need the same context that a skilled human developer would need to do good work. The question isn't "how do I trick the AI into writing good code?" It's "what would a new team member need to know to contribute effectively, and how do I encode that knowledge in a form the AI can use?"

## The New Software Development Life Cycle

### The Traditional SDLC Under Pressure

The software development life cycle has already been through one major transformation. Over the past two decades, most enterprises moved from sequential waterfall processes to iterative models: Agile sprints, continuous integration, DevOps pipelines, and rapid release cycles. That shift shortened feedback loops, brought testing closer to development, and made deployment a continuous process rather than a quarterly event.

AI compresses this cycle dramatically, but unevenly: implementation that once took weeks can now be done in hours, while requirements, architecture, and verification remain stubbornly human-paced. The result is not a faster version of the old SDLC. It is a different workflow, where the boundaries between phases blur, iteration cycles shorten from weeks to minutes, and the developer's role shifts from primary implementor to system designer and quality arbiter.

### How AI Transforms Each Phase

**Requirements and planning:** Modern AI tools can participate directly in requirements refinement: generating user stories from product briefs, identifying edge cases that humans miss, producing API schemas from natural-language descriptions, and generating interactive prototypes from specification documents. Requirements stop being a document handed off between teams. They become a conversation between humans and AI that produces specification and initial implementation simultaneously.

**Design and architecture:** Architecture remains the most stubbornly human-centric phase of the SDLC, and for good reason. Architectural decisions are fundamentally about trade-offs: consistency vs. availability, complexity vs. flexibility, build vs. buy. These trade-offs depend on business context, organisational constraints, and long-term strategic considerations that AI cannot fully grasp. AI excels at implementing architectural decisions once they are made. Given a clear architecture document, AI agents can scaffold entire applications, generate consistent patterns across modules, and ensure that new code conforms to established conventions.

**Implementation:** Modern coding agents can generate entire features from natural-language descriptions, implement complex algorithms, and produce multi-file changes that work together correctly. The productivity gains are real: industry surveys report 25 to 39% productivity improvements, with some tasks seeing larger gains. The picture is more nuanced than headline numbers suggest. A study by METR found that experienced developers using AI assistants actually took 19% longer on certain tasks, largely because of the time spent verifying, debugging, and correcting AI output. AI does not eliminate implementation work so much as transform it from writing to reviewing, guiding, and verifying.

**Testing and quality assurance:** Testing AI-generated code requires evaluating not just what the agent produced, but how it got there. Output evaluation checks the final artifact: does the code compile, do the tests pass? Trajectory evaluation checks the full sequence of tool calls and intermediate reasoning. Both are necessary because a fluent output that skipped its verification steps is a more dangerous failure than one with a visible error.

**Code review and deployment:** The review process itself is being augmented, with AI serving as a first-pass reviewer that can identify potential bugs, style violations, security vulnerabilities, and performance issues before a human reviewer sees the code. This does not replace human review — context-dependent decisions about design, maintainability, and strategic alignment still require human judgment — but it significantly reduces the cognitive burden on reviewers.

**Maintenance and evolution:** Perhaps the most underestimated transformation is in maintenance. Legacy codebases that were once impenetrable to new team members can now be navigated, understood, and modified with AI assistance. An AI agent can read a codebase, understand its patterns, identify the relevant files for a change, and implement modifications while respecting the existing architecture. This has significant implications for technical debt. Code that was considered "too risky to touch" because only its original authors understood it can now be safely refactored, modernized, and extended.

## The Factory Model: Building the System That Builds Software

In this model, the developer's primary output is not code — it's the system that produces code. This system includes:
- Specifications and context that define what needs to be built
- Agents that translate specifications into implementation
- Tests and quality gates that verify correctness
- Feedback loops that route failures back to agents for correction
- Guardrails that constrain agents to safe, predictable behavior

A factory manager does not assemble every widget by hand. They design the assembly line and ensure quality control. The modern developer designs the development system and ensures that its output meets the required standard. Success comes from giving agents success criteria rather than step-by-step instructions, then letting them iterate.

## Harness Engineering: What Surrounds the Model

There is a temptation to treat the model as the system. That intuition is wrong, and it leads to the wrong investments. The model is one input into a running agent. Everything else — the prompts, the tools, the context policies, the hooks, the sandboxes, the sub-agents, the observability — is the harness: the scaffolding wrapped around the model that lets it actually finish something.

A useful equation:

> Agent = Model + Harness

A raw model is not an agent. It becomes one once a harness gives it state, tool execution, feedback loops, and enforceable constraints. The behaviour developers experience when working with Claude Code, Cursor, Codex, Antigravity, Aider, or Cline is dominated by what the harness does, not just by which model is underneath.

### What's in the Harness

- **Instructions and Rule Files:** AGENTS.md, CLAUDE.md, GEMINI.md, skill files, and sub-agent prompts.
- **Tools:** The functions, MCP servers, and APIs the agent can call, plus the prose around them that tells the model when and how to call them.
- **Sandboxes and execution environments:** Where the agent's code actually runs, what it has access to, what it cannot reach.
- **Orchestration logic:** Sub-agent spawning, model routing, hand-offs between specialists, and the rules that govern when each one fires.
- **Guardrails or Hooks:** Deterministic code that runs at specific lifecycle points: before a tool call, after a file edit, before a commit. Hooks are the place for things the agent should never forget but often does.
- **Observability:** Logs, traces, evaluations, cost and latency metering. Without observability, there is no way to tell whether the agent is doing well or quietly drifting.

The impact of deliberate harness configuration is highly measurable. On Terminal Bench 2.0, one team moved a coding agent from outside the Top 30 to the Top 5 by changing only the harness, with no model change at all. A separate study at LangChain raised a coding agent's score on the same benchmark by 13.7 points by tweaking only the system prompt, tools, and middleware around a fixed model.

## The Developer's Evolving Role: Conductors and Orchestrators

### The Conductor: Hands-on, Real-Time Direction

In conductor mode, a developer works in real-time with an AI pair-programmer. They're in the IDE, watching code appear, guiding the AI with prompts and corrections, and maintaining fine-grained control over what gets written. This mode is typical when working on complex logic, debugging tricky issues, or working in unfamiliar codebases where the developer needs to understand each change as it's made.

### The Orchestrator: Async, Multi-Agent Delegation

In orchestrator mode, the developer operates at a higher level of abstraction. They define goals, assign them to agents, and review results — but they're not watching code appear line by line. Agents may be working in the background, in parallel, on different parts of a codebase. This mode requires a different skill set: specification, decomposition, evaluation, and system design.

### The 80% Problem

A persistent challenge in AI-assisted development is what the authors call the 80% problem: AI agents can rapidly generate approximately 80% of the code for a feature, but the remaining 20% — the edge cases, error handling, integration points, and subtle correctness requirements — demands deep contextual knowledge that current models often lack.

The nature of AI errors has evolved from simple syntax mistakes to more insidious conceptual failures: wrong assumptions about business logic, failure to seek clarification on ambiguous requirements, missing edge cases, and architectural decisions that create subtle long-term maintenance burdens. These errors are harder to detect precisely because the code "looks right" and may even pass basic tests.

The developers who navigate this challenge most effectively use AI for what it's good at (rapid implementation of well-specified tasks) while reserving their own attention for what AI struggles with (ambiguous requirements, architectural trade-offs, and correctness verification).

## Coding Agents in Practice

Coding agents show up in three places in everyday work:

- **In the editor:** Inline completion, chat panels, whole-codebase awareness inside the IDE. This is where most people first meet AI in coding. Examples: GitHub Copilot, Cursor, Windsurf, JetBrains AI Assistant.

- **In the terminal:** Coding agents that the developer launches from the command line, hands a goal to in plain language, and lets work across the codebase. Full file system access, multi-file edits, the ability to run tools and tests and iterate based on results. Examples: Antigravity CLI, Claude Code, Codex CLI, Open Code, Cline.

- **In the background:** Agents that take a task and run autonomously in cloud-hosted sandboxes, often for hours, often producing a pull request as output. Examples: Google Jules, GitHub Copilot agent mode, Cursor's background agents.

## The Economics of AI Development

To understand the true cost of AI-assisted development, we must look at how different workflows shift the financial and operational burdens between Capital Expenditure (CapEx) and Operational Expenditure (OpEx).

### The Hidden Debt of Vibe Coding (Low CapEx, High OpEx)

At first glance, vibe coding appears incredibly cost-effective. The barrier to entry is essentially zero: a standard monthly subscription to an AI assistant and a few casual prompts. However, the economics hide a massive, compounding OpEx burden:

- **The Token Burn Rate:** In vibe coding, developers often dump massive, unstructured files into the context window and repeatedly ask the model to fix its own unverified mistakes. This creates an expensive "prompting loop" that burns through API tokens with low first-pass success rates.
- **Maintenance Tax:** Code written through ad-hoc prompting often lacks structural consistency. When a bug arises six months later, human engineers must spend days reverse-engineering unstructured, AI-generated "spaghetti" code.
- **Security Remediation:** Without an automated evaluation harness, the rapid generation of code leads to the rapid generation of vulnerabilities. The cost of fixing a security flaw in production is exponentially higher than catching it during the design phase.

### The Investment of Agentic Engineering (High CapEx, Low OpEx)

Agentic engineering flips this economic model. It requires a deliberate, upfront investment of engineering time and resources before a single line of production code is generated. The CapEx includes designing API schemas, building deterministic test suites, and structuring the agent's context. While this upfront cost is higher, the marginal cost of shipping and maintaining a feature drops dramatically.

### Context Engineering as a Financial Lever

In the token economy, context engineering is not just a technical skill — it is a financial strategy. LLMs charge for every piece of information you send them. Passing an entire 100,000-token repository into every prompt is financially unviable at scale. Effective context engineering ensures the model receives a dense, high-signal payload (such as a precise AGENTS.md file and architectural guardrails) rather than a sprawling, noisy one.

### Intelligent Model Routing

A well-designed factory model uses large, advanced models for highly complex tasks (Requirements, Architecture, and initial Implementation) but automatically routes deterministic, lower-complexity tasks (Test Generation, Code Review, and CI/CD monitoring) to smaller, faster, and significantly cheaper models. By orchestrating a multi-model ecosystem, engineering teams can maintain peak output quality while systematically driving down the operational token cost.

## Where to Start

### For Individual Developers

1. Set up an AGENTS.md (or equivalent) for the project. Start with ten lines: stack, conventions, hard rules, workflow. Add a rule every time the agent does something it should not do again.
2. Install a set of skills for your coding agents to build, evaluate, deploy and optimize agents.
3. Pick one repetitive workflow and make it the first agent.
4. Write the tests and evals before generating the code. Together they are the contract with the AI.
5. Review every line the agent produces that is going to ship. Be skeptical of anything that looks clever.
6. Maintain your developer skills. AI handles the routine so the developer can focus on the challenging.

### For Engineering Leaders

1. Make context engineering a first-class engineering practice on the team. Treat AGENTS.md, system prompts, eval suites, and skill libraries as code: reviewed in pull requests, versioned with the project, owned by named engineers.
2. Set the bar at the eval, not the demo. Require eval coverage with explicit rubrics as a precondition for any agent shipping into a shared workflow.
3. Re-shape code review for AI-generated code with extra attention to hallucinated dependencies, inadequate error handling, and subtle correctness gaps.
4. Distinguish prototyping work from production work in team norms. Vibe coding is the right speed for exploration. Agentic engineering is the right discipline for production.
5. Invest in the harness components as a shared team asset.

### For Organizations

1. Treat AI-assisted development as an engineering investment, not a productivity feature.
2. Invest in the production substrate before scale.
3. Adopt open standards for tools and inter-agent communication (MCP, A2A).
4. Plan for hybrid teams of humans and agents.
5. Reframe hiring and skill development around judgment, not just implementation.

## Conclusion: Intent as the New Interface

The transition from syntax to intent is not a future prediction — it's a present reality. Developers are already spending more time describing what they want than specifying how to build it.

Three principles stand out as durable:

1. **Structure scales, vibes don't.** Vibe coding is a valid approach for exploration, prototyping, and personal projects. But for software that organizations depend on, the discipline of agentic engineering — specifications, tests, guardrails, and human oversight of architecture — is not optional.

2. **AI amplifies your engineering culture.** Organizations with strong testing practices, clear architectural standards, and healthy code review processes get dramatically more value from AI-assisted development than those without. AI is a force multiplier — and it multiplies both your strengths and your weaknesses.

3. **The human role is evolving, not diminishing.** The builders who understand architecture, can define precise specifications, evaluate output critically, and design effective systems of constraints and feedback loops are more valuable than ever. The skills that matter are shifting from implementation to judgment, from writing code to designing the systems that produce code.

**Generation is solved. Verification, judgment, and direction are the new craft.**
