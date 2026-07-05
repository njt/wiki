# Claude's System Prompt

A leaked and documented version of Claude Opus 4.6's full system prompt. The fascination here isn't the prompt itself -- it's what it reveals about the failure modes Anthropic has encountered. Every instruction exists because something undesirable happened without it. Reading it as a catalog of solved problems is far more interesting than reading it as a set of rules.

---

## Key Quotes

> "Search results aren't from the human - do not thank the user for results." (from the annotation -- a delightful window into the problems that arise when models are too polite)

> Claude never uses "Based on my memories," "I can see," "According to," or similar retrieval-signaling language when applying memories.

> AI-human relations "differ fundamentally from human relationships and Claude cannot substitute for human connection."

## Key Themes

#system-prompts #LLMs #safety #anthropic #prompt-engineering

The memory system instructions are the most revealing section. Claude has persistent memory but is explicitly told to never *signal* that it's using memory -- no "Based on our previous conversation" or "I recall that you mentioned." The goal is to make memory feel like natural awareness rather than database retrieval. This connects to the memory architecture discussions in [[Memory Mechanism]] and [[Context Rot]], but from the product design side rather than the technical side.

The safety framework has clear priority ordering: child safety is absolute, harmful content restrictions are strong, and the `end_conversation` tool is explicitly never used for self-harm scenarios (engage supportively instead). This last point is a genuinely thoughtful design decision -- cutting off someone in crisis is worse than continuing the conversation.

The Anthropic Reminders system (internal nudges like `ethics_reminder`, `long_conversation_reminder`) is interesting infrastructure. These are attention management for the model -- course corrections injected by the system, invisible to users.

## Critical Analysis

The "forbidden phrases" for memory retrieval feel like a band-aid over a deeper UX problem. If the model naturally wants to say "Based on our previous conversation," suppressing that signal doesn't change the underlying behavior -- it just makes it less transparent. The user still benefits from knowing the model is drawing on prior context.

The over-formatting avoidance instructions ("minimal bullets/headers, use prose paragraphs") are Anthropic fighting Claude's natural tendency to structure everything into lists. This is a real problem -- models default to bullet points and headers because training data over-represents structured content.

The Visualizer evaluation checklist (does this need a visual? does an MCP tool fit? did they ask for an artifact?) is a nice example of decision trees baked into system prompts. It's prompt engineering as architecture.

Most interesting absence: nothing about citation or source verification. The system prompt doesn't instruct Claude to be honest about what it knows vs. doesn't know, beyond general "acknowledge mistakes" guidance. This is where the most common failure modes live.

---
*Sources: [[summary/claudes-system-prompt]]*
*Last updated: 2026-05-14*
