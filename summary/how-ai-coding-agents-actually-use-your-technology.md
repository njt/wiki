---
url: https://developer.microsoft.com/blog/how-ai-coding-agents-actually-use-your-technology
title: "How AI coding agents actually use your technology"
author: Waldek Mastykarz
date_fetched: 2026-06-09
date_published: 2026-05-27
topics:
  - mcp-and-tool-protocols
  - agent-architecture
---

You ship an SDK, a CLI, an API, and developers use it. Now AI coding agents use it too, except they use it differently than humans do. Most of the time you have no idea what's actually happening between "developer types a prompt" and "agent generates code with your technology." Is the agent reading your docs? Is it calling your MCP server? Is it ignoring both and guessing from memory?

In the previous article, we introduced the AX stack: model, harness, and agent extensions. We talked about what's fixed and what you can influence. This time, let's trace through what actually happens, step by step, when an agent encounters your technology. Because until you see the mechanics, you can't fix what's breaking.

## What happens when a developer says "build me something"

A developer opens their coding agent, types a prompt: "Build me a REST API with authentication using Contoso Identity." Here's what happens next.

### Step 1: The harness assembles context

Before anything hits the model, the harness (Copilot, Claude Code, Cursor) assembles the context window. The VS Code team recently published a deep dive into how their harness works, covering context assembly, tool exposure, and the agent loop. The harness pulls together:

- the system prompt (harness-specific, you can't see or change it)
- environment details: the developer's OS, the full path to the working directory
- workspace files the harness thinks are relevant
- tool descriptions from installed extensions (MCP servers, skills, custom agents)
- conversation history
- any instruction files (.github/copilot-instructions.md, AGENTS.md, etc.)
- the developer's prompt

This is just an example, because every harness is different. Nonetheless, if you consider the context window size of any of the popular LLMs used for coding, you can start to see how such a setup quickly fills up the available tokens. The harness decides what makes the cut. If the developer has 20 extensions installed, the harness might summarize tool descriptions, drop some entirely, or rank them by estimated relevance. Your extension's description is competing for space before the model even sees it. If it exceeds the harness's length limit (each harness sets its own), it gets ignored entirely, no matter how relevant it is. And details you'd never think about, like the OS, or the directory path, influence the model's decisions. It'll generate platform-specific code, assume different toolchains, even pick different default configurations based on what it sees here.

### Step 2: The model reads the room

The model receives this assembled context and does something humans don't: it reads everything at once. The system prompt, the tool descriptions, the workspace context, the developer's prompt. It builds a mental model of what's available and what the developer asked for.

Here's where training data matters. If the model has seen your technology during pre-training, it already has opinions. It knows (or thinks it knows) your API patterns, your SDK conventions, your common error messages. If it hasn't seen your technology, it has nothing, and it'll either ask for help or guess based on similar technologies. Either way, the model's job at this point is to decide what to do first: does it have enough information to start coding, or does it need to call a tool?

It turns out, that this decision is a combination of the behavior encoded in the model and the instructions the harness adds on top. Some agents are more inclined to call tools straight away, while others rely on their own knowledge and only use tools when the user asks them to. Some agents tend to search for the latest information on the internet first, while others prioritize efficiency and start working on the task if they feel they know enough. So even if you ship a great extension, one agent might call it proactively while another never touches it unless the developer explicitly asks.

### Step 3: Tool selection (or not)

If the model decides it needs more information, it looks at the available tools and skills. This is where your MCP server's tool descriptions and skill definitions matter. The model reads each description and decides: does this help with what the developer asked? Notice, that this decision is based on semantic matching, not keyword search. The model is interpreting intent. If the developer said "authentication" and your tool is described as "configure identity provider settings," the model has to bridge that gap.

And even when your description matches the intent perfectly, the model might still skip it. If the task looks simple enough, or if the model feels confident it already knows the answer, it won't bother calling your tool. It'll just go with what it has. This is especially painful when the model has some training data for your technology but it's outdated: high confidence, stale information.

If the model doesn't select any tool or skill, it proceeds with whatever it already knows: it'll use pre-training data. It'll use whatever version of your SDK the model learned from, however many months ago that training cut-off was. If your API changed since then, the generated code is wrong and neither the developer nor the agent knows it.

### Step 4: Tool invocation

Say, the model selected your tool. Now it needs to call it correctly. It constructs the parameters based on the tool's schema and the developer's intent. But the model might map the developer's intent to the wrong parameters, or invent values that don't match the schema.

If multiple tools match the intent, the model might call either one or both. And it's not just about competition for relevance. Say there are two MCP tools from different servers that can provide more information. One is invokable directly, the other uses subrouting: a parent tool that returns information about subtools that actually hold the answer. This is a common technique in larger MCP servers to lower the number of tools exposed to the LLM and optimize token usage. If in the first turn one tool returns the actual information while the other returns routing instructions for the subtool, the model might decide it knows enough and never invoke the subtool, failing the routing entirely. This is why testing tools in isolation isn't enough, and you need to test them in the combinations developers actually use.

Your MCP server receives the call, processes it, and returns content. What you return matters a lot. Return too much content and the model ignores parts of it or gets confused. Return too little, and the model fills gaps with assumptions. Format the content in a confusing way, and the model misparses it. Provide wrong information and you can derail the agent from the original task entirely. We've seen extensions cause agents to upgrade a project to a different framework version than what the developer asked for, or switch to a different programming language mid-task. And all of it goes back into the context window, consuming tokens. If you return 3,000 tokens of documentation when 200 would do, you just pushed other relevant context out of the window. That's drag.

### Step 5: The model processes the response

The model takes your tool's response and integrates it with everything else in context. It now has the developer's intent, workspace context, and your tool's output, and it decides whether to generate code, call another tool, or ask the developer a question. If your tool returned clear, specific content (a code sample, a schema, step-by-step instructions), the model can proceed. If your tool returned a wall of reference documentation, the model has to extract what's relevant, and its extraction might miss the critical detail.

How would you know? You wouldn't. The tool got called, it returned content, everything looks fine from the outside. But the model latched onto the wrong paragraph and generated code that uses an internal-only endpoint. That's the worst kind of quality failure: invisible until someone runs the code.

### Step 6: Code generation

Next, the model generates code. This is where everything upstream either pays off or falls apart. If discovery worked, the model found your extension. If selection worked, it called the right tool for the task. If the tool response was good, the model has accurate, current information, and the generated code uses the right SDK version, the right patterns, the right authentication flow.

If any step failed, the model falls back to its training data. It might use an older API version, pick a competing SDK, or invent an endpoint that doesn't exist. The developer sees working-looking code that fails at runtime, and blames the agent.

Many agents integrate with language servers (LSPs) and respond to problems reported in real time, adjusting code as it's generated rather than waiting until the end. In VS Code, the agent also monitors the Problems panel where any active extension can surface diagnostics: type errors, lint violations, deprecation warnings. If your technology contributes to that feedback (through a VS Code extension, a language server, or a linter), you're influencing the agent's output during generation, not just after. That's another surface you control.

### Step 7: Iteration

The agent doesn't stop at first generation. If the harness supports it (and most do), the agent builds the code, runs tests, observes errors, and tries again. Your technology surface matters here too. If your CLI produces clear error messages, the agent can quickly and efficiently self-correct. If your build tooling or test runner returns helpful output with specific codes and suggestions, the agent fixes the problem. If your errors are cryptic ("Error: operation failed"), the agent is flying blind and might iterate in the wrong direction for 10 turns before giving up.

This is a part of AX that people overlook. Your error messages aren't just for human developers anymore. They're for agents, and agents have no intuition to fall back on. They take your error message literally.

## The information cascade

Notice the dependency chain:

1. Context assembly → determines what the model can see
2. Model interpretation → determines what the model thinks is available
3. Tool selection → determines whether the model uses your extension
4. Tool invocation → determines what information the model gets
5. Response processing → determines how the model uses that information
6. Code generation → determines what the developer actually receives
7. Iteration → determines whether the agent can self-correct

A failure at step 1 cascades through everything. If the harness drops your tool description, steps 3-7 never happen. Your tool might as well not exist. If step 3 fails (the model sees your tool but doesn't connect it to the task), it's the same result. If step 4 succeeds but the response is confusing, step 6 produces broken code.

This is why measuring only "was my tool called?" tells you almost nothing. Your tool can be called correctly and still produce drag if the response quality is poor. Your tool can never be called despite being installed, which means your discovery is broken. And between those two extremes? A dozen ways each step can silently degrade the next.

## Summary

Most AX failures are invisible. They happen upstream, silently, and you never see them. You can't diagnose them by looking at outputs alone: you need to know which step failed, because each one has a different fix. In the next article, we'll cover how to measure whether your AX work is creating lift or drag at each step.
