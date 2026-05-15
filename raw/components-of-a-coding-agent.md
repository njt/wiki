---
title: "Components of a Coding Agent"
url: https://magazine.sebastianraschka.com/p/components-of-a-coding-agent
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
---

# Components of A Coding Agent - Sebastian Raschka

## Main Topic
How coding agents (like Claude Code and Codex) function by wrapping LLMs in sophisticated software systems that manage context, tools, memory, and execution.

## Key Arguments

**Agent Architecture Layers:**
The author distinguishes between LLMs (core models), reasoning models (enhanced inference), and agents (control loops around models). "The LLM is the engine, a reasoning model is a beefed-up engine (more powerful, but more expensive to use), and an agent harness helps us the model."

**Harness as Differentiator:**
Rather than model quality alone determining capability, "the harness can often be the distinguishing factor that makes one LLM work better than another." The surrounding system shapes user experience more than the base model.

## Six Core Components

1. **Live Repo Context** - Agents gather workspace summaries before working -- branch status, file structure, project documentation -- rather than starting without context on each task.

2. **Prompt Caching** - Systems maintain stable prefixes (instructions, tool descriptions, workspace info) that remain consistent across turns, only updating changing elements like recent transcripts and user requests.

3. **Tool Access and Validation** - Models emit structured actions that the harness validates for known tools, proper arguments, file path safety, and user approval before execution -- "giving the model less freedom, but it also improves the usability."

4. **Context Reduction** - Coding agents combat token bloat through clipping verbose outputs, deduplicating repeated file reads, and compressing older transcript history while preserving recent events.

5. **Structured Session Memory** - Systems maintain both a complete transcript (full durable record) and working memory (distilled summary of current priorities), enabling session resumption and task continuity.

6. **Bounded Subagents** - Agents can delegate tasks to constrained subagents that inherit sufficient context for usefulness but operate within tighter restrictions than the main agent.

## Notable Quotes

"Coding work is only partly about next-token generation. A lot of it is about repo navigation, search, function lookup, diff application, test execution, error inspection."

"A lot of apparent 'model quality' is really context quality."

## Main Conclusion
Coding harnesses succeed by combining intelligent context management, tool boundaries, and session continuity rather than relying solely on model capabilities.
