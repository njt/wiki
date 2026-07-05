---
url: https://github.com/idyllic-labs/elements-of-agentic-system-design
date_fetched: 2026-07-05
backfilled: true
---

*A Conceptual Framework for the Design of Intelligent Systems from First Principles*

**Read the complete framework** | **Raw text (LLM-friendly)**

**Who this is for:** This is a design space map for people building agent frameworks, languages, SDKs, and platforms. It shows what users expect from agents, the code patterns that implement those expectations, and where agentic system designs vary.

Also useful for practitioners building agents on these systems, and for readers who want to understand how agentic systems map to concrete software components. Some familiarity with LLMs and software systems is assumed.

This repository contains the outline for **a forthcoming book**. **Interested in co-authoring?** Reach out to **william@idylliclabs.com**.

- The 10 Elements
- What This Framework Provides
- Motivation
- Key Relationships
- Quick Reference: Behavior → Implementation
- Claude Code Skill
- Related Work & Influences
- Contributing
- License

| # | Element | What it is | Where capability lives | 
|---|---|---|---|
| 1 | Context | Information available to the model for a single call | Token budget + context construction | 
| 2 | Memory | External storage for selective retrieval into context | Storage structures + retrieval mechanisms | 
| 3 | Agency | Translation layer from text to effects | Execution boundary + policy enforcement | 
| 4 | Reasoning | Grammar of call composition (chaining, looping, branching) | Call structure + interstitial computation | 
| 5 | Coordination | Communication and sequencing between reasoning structures | Execution flow + data flow | 
| 6 | Artifacts | Shared persistent state for coordination | Typed objects + operations + lifecycle | 
| 7 | Autonomy | What triggers execution, who owns the main loop | Trigger infrastructure + context reconstruction | 
| 8 | Evaluation | Determining whether the system succeeded | Quality signals + measurement functions | 
| 9 | Feedback | Gradient signals that steer behavior | Signal sources + injection points | 
| 10 | Learning | Feedback that persists to change future behavior | Learnable parameters + extraction pipeline | 

The 10 elements are indexed on **intuitive notions of intelligence**, not implementation categories. Each element is named for the behavior its code patterns produce.

Defined terms for each element (context, memory, agency, etc.) and the mechanisms that implement them.

A way to **take apart any agentic system**. Given an agent, you can break it down into these 10 elements and understand how it works mechanistically.

When something isn't working, the elements help you **isolate the problem**:

- Agent forgetting things? → Context or Memory problem
- Agent not doing what you want? → Agency or Reasoning problem
- Agents stepping on each other? → Coordination or Artifacts problem
- Agent not improving? → Evaluation, Feedback, or Learning problem

For each observed behavior, the framework names the corresponding code pattern.

This framework came from building agent frameworks and platforms, and from the frustration of working in a space that moves fast but lacks conceptual grounding. The excitement around the technology leads people to conflate model capability with system intelligence. As models improve, the bottleneck shifts to system design, but there's no shared vocabulary for where things should go.

When you want to implement something smart in an agentic system, you need to know where it belongs: is it a context problem, a memory problem, a coordination problem? Without a clean conceptual framework, every design decision feels ad hoc. You end up reinventing patterns or mislocating functionality because there's no map of the design space.

This framework provides that map. The model is a stateless text-to-text function. Everything else (memory, agency, reasoning, coordination, learning) is architecture you build around it. Every intelligent-seeming behavior traces to concrete code: loops, database queries, schedulers, policy checks. Once you see this, agentic systems become software you design, inspect, and debug like any other program.

A recurring **externalization pattern** appears throughout the framework:

- **Memory**= externalized context (storage for a single agent)
- **Artifacts**= externalized coordination (shared state for multiple agents)
- **Learning**= externalized feedback (patterns stored for future behavior)

**The improvement stack:**

- **Evaluation**measures quality
- **Feedback**steers the current task
- **Learning**persists feedback to steer future tasks

Each behavior in the left column corresponds to a concrete implementation pattern in the right column.

| "It seems to..." | Actually is... | 
|---|---|
| Remember what I said | Conversation history array in prompt | 
| Have long-term memory | Database + retrieval into context | 
| Do things in the world | Structured output → parser → function dispatch | 
| Think step by step | Multiple calls with state passed between | 
| Plan before acting | `plan = llm(task)`→`for step in plan: execute(step)` | 
| Check its own work | Generate → separate verify call → conditional retry | 
| Have multiple experts | Different system prompts routed by classifier | 
| Work while I sleep | Cron job triggers agent | 
| Learn from experience | Outcomes → extraction → stored → retrieved into future contexts | 

This repository includes a Claude Code skill that lets you analyze and design agentic systems using this framework.

**Personal (all your projects):**

```
mkdir -p ~/.claude/skills/intelligence-designer
curl -o ~/.claude/skills/intelligence-designer/SKILL.md \
  https://raw.githubusercontent.com/idyllic-labs/elements-of-agentic-system-design/main/SKILL.md
```
**Project-specific:**

```
mkdir -p .claude/skills/intelligence-designer
curl -o .claude/skills/intelligence-designer/SKILL.md \
  https://raw.githubusercontent.com/idyllic-labs/elements-of-agentic-system-design/main/SKILL.md
```
Once installed, use the skill in Claude Code:

```
/intelligence-designer Analyze how Claude Code handles context management
/intelligence-designer How does a typical RAG chatbot work?
/intelligence-designer Design an agent that monitors GitHub issues and auto-triages them
```
The skill injects the full framework into context and guides Claude to think mechanistically about agentic systems, tracing behaviors to code patterns, identifying elements, and spotting tradeoffs.

This framework builds on and is influenced by the following resources:

- **Intelligence Design**— Argues that intelligent behavior in AI systems is designed, not emergent. Understanding agentic systems means understanding the architecture that produces intelligent-seeming behavior.

- **OpenClaw vs Claude Code**— An analysis of two agentic coding systems using this framework.

- 
**Building Effective Agents**(Anthropic, 2024) — Anthropic's guide to building agents with simple, composable patterns. Emphasizes starting simple and only adding complexity when needed. Introduces workflow patterns: prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer.
- 
**12-Factor Agents**(HumanLayer) — Principles for building production-ready LLM applications, inspired by Heroku's 12-Factor App methodology. Key insights: own your prompts, manage context windows explicitly, own your control flow, small focused agents beat monoliths.
- 
**Agentic Design Patterns**(Andrew Ng, 2024) — Four key patterns: Reflection, Tool Use, Planning, and Multi-Agent Collaboration. Helped popularize the term "agentic" in the AI community.
- 
**Agentic Design Patterns: A System-Theoretic Framework**(arXiv, 2025) — Academic framework decomposing agentic systems into five functional subsystems: Reasoning & World Model, Perception & Grounding, Action Execution, Learning & Adaptation, and Inter-Agent Communication.

The above resources focus on **how to build** agents (patterns, best practices, implementation). This framework focuses on **analyzing** agents: decomposing their behavior into elements and mapping each to specific code patterns.

Contributions are welcome! This is an open project and we appreciate help from the community.

- **Report issues**— Found an error or unclear explanation? Open an issue.
- **Suggest improvements**— Have ideas for better examples or clearer framing? Let us know.
- **Add examples**— Real-world case studies that illustrate the elements are valuable.
- **Translate**— Help make this framework accessible in other languages.

This outline will be expanded into a book. If you're interested in co-authoring or making substantial contributions, please contact **william@idylliclabs.com** with:

- Your background and expertise
- Which elements you're interested in contributing to
- Any relevant writing samples or prior work

See CONTRIBUTING.md for detailed guidelines.

This work is licensed under Creative Commons Attribution 4.0 International (CC BY 4.0).

You are free to:

- **Share**— copy and redistribute the material in any medium or format
- **Adapt**— remix, transform, and build upon the material for any purpose, including commercially

Under the following terms:

- **Attribution**— You must give appropriate credit, provide a link to the license, and indicate if changes were made.

**Elements of Agentic System Design**

A project by Idyllic Labs.

Created by William Chen, based on over two years of experience building agentic systems.

AI tools assisted in the editing and structuring of this work, primarily Claude Opus 4.5 (Anthropic) and GPT-5.2 (OpenAI).

- **Email**: william@idylliclabs.com
- **Twitter**: @stablechen
