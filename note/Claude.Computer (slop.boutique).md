# Claude.Computer (slop.boutique)

Claude.Computer is an early-access app where Claude answers not in chat but by building on a shared spatial desktop — files, documents, spreadsheets, slides, whiteboards, and small custom apps that human and agent both see and both edit, one desktop per conversation. Its architecture is strikingly small: one HTML file wrapped in a claude.ai artifact, an agent that writes only a plain-text block format parsed while still streaming, and a turn that is exactly two calls — read one view document, write one turn document.

---

## What it is

The product's core move is spatial: "Instead of a long reply that scrolls away in the chat, Claude answers by making something on a shared, spatial desktop." The chat shrinks to a sentence; the work lives on the desktop, and either side can pick up where the other left off.

- **One HTML file** wrapped in a claude.ai artifact. Give Claude the link in a conversation and it publishes its own copy — every conversation gets its own computer with its own storage. No backend to speak of.
- **Content vs behaviour.** Files hold content; apps hold behaviour. Built-in apps (documents, spreadsheets, presentations, canvases, records, quizzes, forms, charts) come pre-built, and Claude writes only their *content* in a few plain lines — so "when Claude rewrites one, what you did stays." Custom *Tanke* apps are short JavaScript programs that show and change files by name; *programs* run on demand and can ask Claude mid-run.
- **The log.** Everything that happened and who did it; every change Claude makes is undoable from it.

## Key quotes

> "Claude never writes files, events or cursors directly, so nothing it writes can break the desktop."

This is the load-bearing design decision, and it's a sandboxing argument dressed as an architecture argument. The agent writes only a declarative block format (`@@doc Name`, `@@slides Name`); the page interprets it. Compare the whole lineage of agent-with-shell-access designs — this is the opposite pole: the model has *no* imperative interface to the environment at all, just a document to emit. Capability comes entirely from the interpreter's grammar.

> "Read the view. ... Write the turn."

Two calls a turn, both documents. The view document bundles "what you changed since Claude's last turn, those files in full, an index of every file, what you are looking at and selecting, how Claude's last turn went, and the id for the next turn." This is context engineering as a protocol: one flat snapshot instead of tool-call gymnastics. It also means the agent sees *your* edits — shared state, not a shared transcript.

> "The app's own logic for instant interaction. A program's `ask()` for small judgments mid-run. A full Claude turn for anything that needs memory, tools, the web or a new app."

The "three speeds of intelligence" is a latency-and-cost routing table stated as product philosophy: deterministic code at the bottom, a cheap model call in the middle, a full agent turn on top. Same allocation thesis as [[Thrifty (Tiered Delegation for Claude Code)]] and [[Stripes Knowledge AI Platform Kai]], but stated from the interface side rather than the infrastructure side.

> "Apps can hand the turn back to Claude. ... a click can be your answer and nobody has to type."

The inversion of the usual agent-UX direction: instead of the agent asking the human a question in chat, the UI submits the human's click as the agent's next prompt. The interface becomes bidirectional currency between the two parties.

## Key themes

- #pattern — **Declarative output as the whole tool interface.** The agent emits a block format; the environment interprets. No file writes, no event injection, no direct state mutation — impossibility by grammar rather than permission by check.
- #concept — **Shared substrate over shared transcript.** Human and agent edit the same files and see the same selection; the conversation is reduced to a thin control channel. This is multiplayer agency, matching the thesis in [[Real-Time Multiplayer Interfaces]].
- #tool — **Streaming-parse interaction.** The page reads Claude's text *while it is still being written*, showing drafts where they'll land. Provenance and undo come free because every change is a logged, interpretable block — the same turn-as-reviewable-artifact move as [[Shop — a Design Canvas Built on Code (html.moldandyeast.com)]].
- #concept — **Tiered intelligence as UX.** Three speeds of response (app logic, program `ask()`, full agent turn) built into the product's own description.

## Critical analysis

**This is a design essay pretending to be a product page, and that's its strength.** slop.boutique's page reads less like marketing than like an architecture document for a particular bet: that the right shape for human-agent collaboration is a shared *substrate* (files + views + a log), not a shared *conversation*. Most agent products still treat chat as the medium and artifacts as attachments; this one inverts it, and the inversion is consistent all the way down to the two-call turn protocol.

**The sandboxing-by-grammar idea deserves more attention than the product will probably get.** "Nothing it writes can break the desktop" is achieved not by permissioning but by the model having no imperative interface at all — it can only emit text in a format the interpreter understands. That is the same deep move as [[Behavior-Centric AI Design]]'s runtime-enforced behaviors and the declarative-output end of the harness-design literature, arrived at from the consumer-UX side. If the pattern generalizes, "the agent's only write path is a parseable document" is a security posture, not just an implementation detail.

**The caveats are the ones you'd expect.** The page is a vendor's own description with no user reports, no pricing detail beyond "early access for subscribers," and no discussion of what happens when the desktop's file universe outgrows a single artifact's storage or when two parties edit the same file concurrently (the "what you changed since last turn" diff model is hand-wavy). The name invites confusion with computer-use agents — [[Cua — Computer Use Agent Platform]] — but this is almost the opposite: not a model controlling an existing OS, but a purpose-built micro-OS both parties inhabit.

**The lineage is visible.** This is the converged endpoint of several threads in this wiki: HTML-as-artifact output ([[Make Pages Interactive]]), canvas-as-code with reviewable agent turns ([[Paper — the Connected Canvas for Teams Shipping with Agents]], [[Shop — a Design Canvas Built on Code (html.moldandyeast.com)]]), and turn-level logging with undo ([[The Log is the Agent]]). What's new here is the consumer framing — documents, quizzes, planners — and the radical minimization: one file, two calls, three speeds.

---
*Sources: [[raw/claude-computer]], [[summary/claude-computer]]*
*Last updated: 2026-11-02*
