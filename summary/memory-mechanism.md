---
title: "Memory Mechanism"
url: https://docs.z.ai/devpack/resources/memory-mechanism
date_fetched: 2026-05-14
section: "Random"
topics:
  - agent-memory-and-context
---

# Memory-Mechanism in Coding Agents

## Core Purpose

Memory systems enable coding agents to retain context across tasks and sessions, reducing redundant inputs and improving efficiency. Without external memory, traditional LLMs cannot preserve state between calls or adapt to user preferences over time.

## Key Architecture Pattern

Standard workflow:
1. **Memory retrieval** – Agent gathers relevant project memory and knowledge before starting
2. **Context construction** – Retrieved memories are assembled into complete working context
3. **Memory update** – After task completion, agent decides whether to record new learnings

## Five Core Memory Types

**Session Memory**: Contextual information for current tasks, including conversation history, recent tool outputs, execution plans, and files in scope—living in the model's context window.

**Project Memory**: Long-lived information about codebases stored in structured `.md` files, covering architecture, coding standards, build workflows, and conventions.

**Semantic Memory**: Factual reference knowledge implemented through RAG, enabling vector search across API documentation and knowledge bases.

**Episodic Memory**: Records past experiences like previous bug fixes and debugging strategies, helping agents learn from prior problem-solving approaches.

**Procedural Memory**: Stores step-by-step workflows and strategies for completing tasks, typically embedded in system prompts and workflow templates.

## Critical Distinction: Two Memory Categories

- **Instruction memory** (human-written): Coding standards, naming conventions, security rules—should remain stable and predictable
- **Learning memory** (agent-accumulated): User preferences, corrections, discovered patterns—improves decisions over time

"If these two types of memory are mixed together, system behavior often drifts over time."

## Hierarchical Memory Organization

Five layers:
1. **Organization-level**: Company-wide security and compliance rules
2. **Project-level**: Team-shared architecture and conventions (most critical layer)
3. **User-level**: Personal coding preferences across projects
4. **Local memory**: Machine-specific configurations not committed to version control
5. **Role-specific**: Separate memory scopes for different subagents

## Best Practices

**Modularity**: Split instructions into topic-specific files under `.claude/rules/` rather than monolithic documents, enabling path-scoped loading.

**Concreteness**: Use verifiable rules instead of abstract principles. Rather than "keep code clean," specify "use 2-space indentation in TypeScript files."

**Scale Management**: Keep main memory files under 200 lines where possible; use imports and symbolic links for shared rules.

**Scope Clarity**: Clearly define who owns each rule, who shares it, and who it applies to.

## Troubleshooting

- Memory files function as contextual instructions, not enforced configuration
- Auto-memory typically persists only in written `.md` files; conversation-only rules disappear during context compression
- Oversized memory files reduce agent adherence and increase instruction conflicts
- File paths and scoping restrictions may prevent memory files from loading

## Main Conclusion

Effective agent memory requires deliberate separation of instruction and learning memory, layered organization by scope, concrete rule specification, and modular file structure.
