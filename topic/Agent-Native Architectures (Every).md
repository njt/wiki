# Agent-Native Architectures (Every)

Every's technical guide for building applications where agents are first-class citizens, not bolted-on afterthoughts. Five core principles (parity, granularity, composability, emergent capability, improvement over time) plus concrete implementation patterns for files as the universal interface, agent-to-UI communication, mobile resilience, and dynamic capability discovery. The best single-document articulation of agent-native application design I've found.

---

## Key Quotes

> "A tool is a primitive capability. A feature is an outcome described in a prompt, achieved by an agent with tools, operating in a loop until the outcome is reached."

This is the central insight. Features become prompts, not code. The test: to change behavior, do you edit a prompt or refactor code?

> "The test: Pick any UI action. Can the agent accomplish it?"

Parity is the foundational principle. Without it, nothing else matters. If the UI can do it but the agent can't, the agent is a second-class citizen. This is why most "AI features" feel bolted on—they violate parity from the start.

> "A really good coding agent is actually a really good general-purpose agent."

The Claude Code insight generalized: the loop + tools architecture that works for coding works for anything. This is the thesis that launched Every's pivot.

> "The agent becomes a research instrument for understanding what your users actually need."

Latent demand discovery inverts product development. Instead of guessing what users want and building it, you observe what they ask the agent to do and optimize the patterns that emerge. This is product management as data science on agent interactions.

> "Domain tools are shortcuts, not gates. The default is open; make gating a conscious decision."

A sharp design rule. Most systems over-constrain agents out of vague safety concerns. The right approach: primitives are always available, domain tools add vocabulary and guardrails, and restrictions are explicit and justified.

> "Silent agents feel broken. Visible progress builds trust."

Agent-to-UI communication isn't polish—it's the difference between an agent that feels alive and one that feels broken. Show thinking, show tool calls, show progress. No silent actions.

> "If a human can look at your file structure and understand what's going on, an agent probably can too."

The legibility heuristic. Files are self-documenting in a way databases aren't. Design for what agents can reason about, and the best proxy is human legibility.

## Key Themes

- #concept **Parity**: agent can achieve anything the UI can. This is non-negotiable.
- #concept **Tools vs. features**: tools are atomic primitives; features are outcomes from prompts + loops.
- #concept **Primitives over workflows**: break decision logic out of tools and into prompts. Domain tools add vocabulary, not judgment.
- #concept **Emergent capability**: agents composing tools to do things you didn't anticipate. The product discovery flywheel.
- #concept **Improvement without deployment**: prompt refinement ships capability without code changes.
- #pattern **Files as universal interface**: the filesystem is the most battle-tested agent interface. Human-legible = agent-legible.
- #pattern **context.md**: portable working memory file the agent reads at session start and updates as state changes.
- #pattern **Completion signals**: explicit `.complete()` rather than heuristic detection of "done."
- #pattern **Dynamic capability discovery**: `list_available_types()` + `read_data(type)` instead of one tool per endpoint.
- #pattern **CRUD completeness audit**: for every entity, verify create/read/update/delete is available.
- #tool **Model tier selection**: not every agent operation needs the most powerful model. Route by task complexity.
- #tool **Checkpoint and resume**: mobile agents need to survive app backgrounding. Checkpoint after every tool result.

## Critical Analysis

**What it gets right**: This is the best articulation of agent-native architecture I've read. The five principles are well-chosen and the operational detail (anti-patterns, success criteria, mobile patterns) shows hard-won experience. The "files as universal interface" section is particularly strong—it's a design philosophy that scales from simple apps to complex systems, and it aligns with how agents actually work rather than how we wish they worked.

**The blind spot**: The guide is shaped entirely by mobile (iOS) development experience and Claude Code. The file-first philosophy is excellent for single-user, mobile-first apps but underdeveloped for multi-user web applications, where databases, permissions, and real-time sync dominate. The authors flag this honestly ("Dan doesn't have a strong opinion there yet"), but it's a significant gap for anyone building agent-native SaaS.

**The tension it doesn't resolve**: The guide preaches atomic primitives and keeping judgment in prompts, but also acknowledges that common operations should graduate to optimized code. The boundary between "domain tool" and "graduated code" is fuzzy. At what point does a frequently-used prompt become a domain tool, and at what point does that tool become a feature? The framework doesn't provide clear decision criteria.

**The emergent capability bet**: The claim that agents will discover latent demand by doing things you didn't anticipate is compelling but unproven at scale. It assumes users will freely experiment with open-ended agent requests. In practice, most users interact with agents through constrained interfaces—they don't know what's possible, so they don't push boundaries. The "Excel model" (simple to start, powerful to explore) works for spreadsheets but may not generalize to agent-native apps where the capability surface is invisible.

**On self-modification**: The guide acknowledges that agents editing their own prompts and code is "emerging" and needs safety rails. This is the area where the principles collide: if agents can modify prompts without code deploys (principle 5), and those modifications become the system's behavior, you've created a self-modifying system with no deployment pipeline. The tension between "improvement without shipping code" and "make self-modification legible" is the most interesting unsolved problem here.

---

*Sources: [[summary/agent-native-architectures]]*
*Last updated: 2026-05-15*
