# Agentation

Agentation is a browser tool that turns UI annotations into structured context for AI coding agents: click an element, add a note, and paste out markdown the agent can act on — or wire it through MCP and let the agent see what you're pointing at live. The distinctive move is that each annotation carries machine-actionable coordinates (CSS selector, source file path, React component tree, computed styles), not just prose.

---

## Key Quotes

> "Agentation turns UI annotations into structured context that AI coding agents can understand and act on."

The one-line thesis, and the word to notice is *structured*. Most feedback tools attach a comment to a location; Agentation attaches the location itself in a form the agent can grep — selector, path, component, style. The annotation is both the complaint and the address.

> "Without Agentation, you'd have to describe the element ('the blue button in the sidebar') and hope the agent guesses right. With Agentation, you give it `.sidebar > button.primary` and it can grep for that directly."

The cleanest statement of the show-don't-tell argument anywhere in the agent-tooling space. Natural-language description is lossy compression of a rendered UI; a selector is exact. This is the same gap [[Make Pages Interactive]] names as "showing Claude instead of telling Claude," but Agentation closes it with deterministic coordinates rather than prose.

> "Your feedback becomes a conversation, not a one-way ticket into the void."

The bidirectional promise. With MCP and an Annotation Format Schema, annotations become addressable state — agents can list them ("what annotations do I have?"), ask for clarification ("should this be 24px or 16px?"), resolve them, or clear them. Feedback stops being an artifact you emit and becomes a shared workspace you co-edit.

> "With MCP, you can skip the copy-paste step entirely — your agent already sees what you're pointing at."

The maturation arc in one line: clipboard → MCP → conversation. The copy-paste version is a starting point for any agent; the MCP version makes the annotation tool an input device rather than a clipboard.

## Key Themes

- #tool — **Agentation as a feedback layer for coding agents.** It's not an agent and not an editor; it's a thin annotation surface that sits between the human's judgment and the agent's grep-able codebase. The whole product is the interface.
- #pattern — **Show-don't-tell, with coordinates.** Where [[Make Pages Interactive]] attaches browser comments to local HTML, Agentation attaches *selectors, file paths, component trees, and computed styles* — turning "fix the thing I'm looking at" into "grep `.sidebar > button.primary`." The feedback is machine-actionable, not just machine-readable.
- #concept — **Feedback as addressable state.** The Annotation Format Schema gives annotations IDs and lifecycle states (list, clarify, resolve, clear). That's a small schema with a large consequence: feedback becomes something both parties can reference and mutate, the missing primitive in [[Guardrails and Feedback Loops]].
- #pattern — **The clipboard → MCP → conversation arc.** Copy-paste is the universal fallback; MCP collapses the hop; the schema adds dialogue. This mirrors the wider movement of agent tooling toward [[Intent Is the Interface]] — pointing rather than describing, with the page itself as the interface.

## Critical Analysis

**This is convergent evolution, and the convergence is strong.** Paras Chopra's [[Make Pages Interactive]] and Codex's native in-app annotation (per that thread's replies) already established "comment on visual artifacts to steer agents." Agentation lands on the same pattern from a different direction — a standalone commercial tool rather than a skill — which is yet more evidence the pattern is real and durable.

**The structured-context piece is the genuine differentiator.** Make Pages Interactive stores prose comments; Agentation stores selectors, file paths, component trees, and computed styles. That's the difference between "here's my opinion" and "here's where, and here's what to act on." It also makes the feedback robust to the compaction problem [[Make Pages Interactive]] flags — a `.sidebar > button.primary` survives context compaction better than "the blue button I mentioned earlier," because it's a stable address, not a pointer into a conversation.

**The "agents talk back" loop is the most interesting idea, and the least specified.** Listing, clarifying, resolving, and clearing annotations turns feedback into a two-way protocol. But the page shows none of the schema, none of the MCP tooling, and no worked example of the dialogue. "Your feedback becomes a conversation" is a claim, not a demonstrated capability — and it's the part that would most reward a spec.

**It's a landing page, not a product deep-dive.** No pricing, no implementation detail, no schema, no integration docs. The licensing model (free for internal use, commercial license for redistribution) is refreshingly simple but unusual for a dev tool — most annotation tools don't think they're being redistributed inside someone else's product.

**Same blind spot as the rest of the genre.** The element you click must exist in a rendered page, so the tool is strong for visual feedback and silent on logic. "This query is wrong" has no selector. [[Make Pages Interactive]]'s critique applies unchanged: the interaction model covers the visual surface, not the logic underneath.

---

*Sources: [[raw/agentation-com]], [[summary/agentation-com]]*
*Last updated: 2026-09-11*
