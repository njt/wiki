# Television (Agent GUI)

Television (television.run) is a Mac/browser application pitched as "the missing GUI for personal agents": a shared visual workspace where an agent pins its output as persistent *artifacts* — cards, interactive views, or external web pages — organized into *channels*, instead of everything scrolling away in chat. Any agent that can run commands and write files can drive it, and it ships shareable *skills* for building interfaces.

---

The page is a landing page, not an essay, so what it offers is a positioning claim more than an argument. That claim is nonetheless crisp and worth taking seriously: the chat transcript is a bad durable surface for agent work, and the fix is a visual space the human curates over time. The agent doesn't need a custom protocol — the bar is deliberately low ("if your agent can run commands and write files, it can use Television"), which makes Television a *harness-adjacent* product rather than a new agent: it assumes you already have one.

## Key Quotes

> "The missing GUI for personal agents."

Bold framing: it concedes the entire terminal-first world of coding agents and declares the next layer. The audience is explicitly "personal agents" — assistants and life/work automation — not software factories.

> "Artifacts let your agent pin important information in Television instead of losing it in chat."

This is the core UX thesis and it matches a real pain: agent output lives in an append-only scroll, and anything you want to keep has to be manually copied into files or notes. Making the agent write to a pinned, structured surface inverts the flow — the transcript becomes the log, the board becomes the product.

> "Organize your artifacts into channels: persistent spaces where you and your agent can collect, arrange, and return to the things you're working on."

Channels are the memory architecture in disguise: a persistent, human-arranged organization of agent output is a shared external memory between person and agent — closer to a wiki than a chat.

> "Supercharge your agent with skills: reusable instructions for creating beautiful interfaces that you can adapt and share."

The skills system is the same pattern the coding-agent ecosystem standardized — reusable instruction bundles — applied to interface-building, so the agent's presentation layer is itself a curated, shareable artifact.

## Key Themes

**#tool** — Television itself: a GUI for existing agents, Mac app plus browser fallback. **#pattern** — artifacts-and-channels as the durable-surface pattern replacing chat scroll. **#concept** — the agent-agnostic integration model: any filesystem-and-commands agent qualifies. **#concept** — interface-building skills, sharing the skill format conventions of the broader agent ecosystem.

## Critical Analysis

The strongest idea here is the two-surface model: chat for the conversation, channels for the durable result. That is genuinely different from most agent GUIs, which are timeline viewers — [[pi-gui]] resurfaces the session transcript as a structured review surface; Television resurfaces the *outputs* as artifacts. Both reject the raw scrollback as the product; they diverge on what deserves the surface.

The marketing does real work but also hides the interesting questions. How does the agent write an artifact — a file convention, a CLI, an MCP tool? Nothing is specified, and that protocol is the actual product boundary. What's the permission model when an agent can write into a space the human treats as their own curated workspace? And is "artifacts as external web pages" an escape hatch or a prompt-injection surface — a persistent page the agent controls is a channel of influence that lives longer than any single message.

There's also an unacknowledged tension with memory systems: channels-as-organized-artifacts overlap heavily with what agent memory architectures ([[Agent Memory]]) try to do inside the model. Television externalizes that to the human's arrangement instincts — bet on taste over retrieval.

## Relations

This strengthens [[Personal Agents]] as a topic: it's a pure play on the human-facing layer of a one-person agent stack, and it names the missing piece ("GUI") that most personal-agent projects leave as a chat window. It nuances [[pi-gui]] — both are desktop GUIs for existing agents, but pi-gui surfaces the *process* (timeline, sessions) while Television surfaces the *output* (artifacts, channels); the comparison sharpens what "agent GUI" can mean. It connects to [[Agent Memory]] by offering a human-curated external memory architecture — organized artifacts instead of retrieval — a competing answer to the same "agent output should persist and be findable" problem. And it echoes [[Intent Is the Interface]]'s concern with what the human actually looks at when working with an agent: Television bets that surface is a board, not a log.

---
*Sources: [[raw/television-run]], [[summary/television-run]]*
*Last updated: 2026-10-05*
