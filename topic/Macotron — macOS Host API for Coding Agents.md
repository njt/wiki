# Macotron — macOS Host API for Coding Agents

Macotron is a free, open-source macOS tool that gives a coding agent hands on the Mac through a scriptable host API rather than screenshots or accessibility trees. Built in Swift with QuickJS as its scripting runtime, it ships a catalog of 73 plugins, writes an `AGENTS.md` next to the ones you install so an agent discovers the API by itself, and exposes a single `macotron.*` namespace of Apple-shipped tools for tiling windows, reading sensors, and talking to models.

---

## Key Quotes

> "Free and open source. Built with Swift and QuickJS."

The two architectural bets compressed into one line. Swift means native macOS integration — the "Apple-shipped tools only" constraint is a *feature*, not a limitation. QuickJS means plugins are plain JavaScript files you can open in the Finder and read, which is what makes the whole thing self-documenting.

> "The catalog ships 73 built-in plugins. First launch copies the ones you pick into `plugins/`. Macotron writes an `AGENTS.md` next to them so a coding agent already knows the API."

The sharpest idea on the page. Most agent tools require the *human* to explain to the agent how to use them; Macotron hands the agent its own API contract as a first-class artifact. It's [[AGENTS.md]]-as-discovery-protocol: the plugins and their documentation are generated together, so the agent's knowledge of the API can't drift from the API itself.

> "Everything hangs off `macotron.*`. Apple-shipped tools only. Plugins call these namespaces to tile windows, read sensors, and talk to models."

The "everything hangs off one namespace" is the plugin-host discipline: a single, curated surface instead of raw OS access. "Apple-shipped tools only" is a scope decision — no third-party app scripting, no sketchy injection — that trades reach for reliability. This is the opposite bet from screenshot-vision computer use: give the agent *typed, named capabilities* and let it call them directly.

> "**Beta.** Macotron goes 1.0 when I, the author, consider it stable. Until then, expect things to move around."

Refreshingly honest versioning. "Stable when the author says so" is the indie-software norm, stated plainly. The beta flag is also the honest caveat about a capability surface that grants an agent OS-level control before its semantics are frozen.

---

## Key Themes

- **#tool** — A macOS capability layer for coding agents: a QuickJS plugin host exposing a `macotron.*` API of Apple-shipped tools.
- **#concept** — **Host API instead of vision or accessibility trees.** The three-way fork in computer use is now visible: screenshot vision ([[Cua — Computer Use Agent Platform]], [[Browser Use]]), structured accessibility trees ([[xa11y — Desktop Automation via Accessibility APIs]]), and curated host APIs (Macotron). Each trades flexibility against token cost and reliability differently.
- **#pattern** — **Self-documenting tool surfaces.** Writing `AGENTS.md` next to installed plugins so the agent discovers its own API is the same instinct as [[Extensible Software in the Age of LLMs]]: make the tool legible to a language model, not just to a human reading docs.
- **#concept** — **Capability layer, not agent.** Macotron doesn't run an agent; it gives an existing agent a way to act on macOS. It sits beside [[maclocal-api]] and [[Gemma Gem]] in the local-first stack, but answers the "act" half rather than the "infer" half.

---

## Critical Analysis

The interesting claim here is that the right interface between an agent and a desktop OS is a *host API*, not pixels and not accessibility trees. [[Computer Use is 45x More Expensive Than Structured APIs]] made the economic case against vision; [[xa11y — Desktop Automation via Accessibility APIs]] made the structural case for AX trees as the API-shaped alternative. Macotron is a third answer: don't make the agent *discover* the OS's structure at all — *publish* a curated namespace (`macotron.*`) of the operations you've decided are safe and useful, and document it in the agent's own language via `AGENTS.md`. It's the [[Smart Models Dumb Pipes]] instinct applied to device control: shrink the agent's surface to what a deliberate API designer chose to expose.

That curatorial discipline is also the ceiling. "Apple-shipped tools only" means Macotron can't drive Chrome's devtools protocol, can't reach a third-party app's internals, can't do the anything-goes exploration that [[Cua — Computer Use Agent Platform]] gets from its AX + vision + SkyLight hybrid. Macotron is betting the agents you actually want are the ones that need *reliable, named* actions — tile this window, read this sensor — not the ones that need to poke arbitrary pixels. For a coding agent that mostly lives in a terminal and occasionally needs to move a window, that's probably the right bet. For general "do my admin for me" agents, it's under-scoped.

The `AGENTS.md` move is the part worth copying, regardless of whether Macotron itself survives. Auto-generating an agent-facing contract next to the code is a pattern that should spread beyond this one tool: every plugin catalog, every CLI surface, every internal API an agent is expected to call could ship its own `AGENTS.md` and cut out the "paste my docs into the prompt" ritual. It's a small idea with a real payoff — the agent's model of the tool and the tool itself stop being two things that can drift apart.

The beta honesty is a warning dressed as a disclaimer. An agent granted the ability to "tile windows, read sensors, and talk to models" on your Mac is a [[Security and Sandboxing]] question, not just a UX one. The page says nothing about sandboxing, permissions, or what a malicious or merely buggy plugin can reach through `macotron.*`. Until the 1.0 story on *containment* is told, this is a capability layer I'd trust with a coding agent I'm watching, not one I leave unattended.

---

## Related Pages

- [[xa11y — Desktop Automation via Accessibility APIs]] — The accessibility-tree answer to computer use. Macotron skips the tree entirely and publishes a curated host API instead — same anti-vision thesis, opposite discovery mechanism
- [[Cua — Computer Use Agent Platform]] — The maximalist computer-use stack (drivers, VMs, per-model loops). Macotron is the minimalist counterpoint: one namespace, one runtime, no sandbox infrastructure
- [[Computer Use is 45x More Expensive Than Structured APIs]] — The economic argument Macotron operationalizes: named, typed capabilities over rendered pixels
- [[Extensible Software in the Age of LLMs]] — The self-documenting surface idea: design tools a language model can read, which `AGENTS.md`-generation is a concrete instance of
- [[Personal Agents]] — Macotron is the "act" half of the local personal-agent stack — the hands a local agent needs to actually do things on macOS

---
*Sources: [[raw/macotron-statico-io]], [[summary/macotron-statico-io]]*
*Last updated: 2026-09-08*
