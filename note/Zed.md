# Zed

A code editor built from scratch in Rust by the creators of Atom, Electron, and Tree-sitter, positioning itself as the AI-era editor with first-class agent integration. Open source (MIT), native on macOS/Linux/Windows, and backed by Zed Industries.

---

## What It Is

Zed is a from-scratch Rust editor that treats AI not as a plugin but as a first-class substrate. The homepage positions it as "Your last next editor" — an ambitious claim in a market where VS Code has 75%+ share. But Zed earns its ambition through architecture: it was built after LLMs existed, so agentic editing, parallel agents, and model-agnostic AI integration are load-bearing design decisions, not bolt-on features.

The team matters. Nathan Sobo, Antonio Scandurra, and Max Brunsfeld previously built Atom (the editor that proved Electron could work), Electron (the framework VS Code runs on), and Tree-sitter (the incremental parsing library that powers syntax highlighting in Neovim, Helix, and now Zed itself). This is not a random startup. It's the same crew that shaped the last generation of developer tools, now building what comes next.

## Key Quotes

> "Zed has all of the features I could ask for in an editor." — Jose Valim (Creator of Elixir)

Valim's endorsement carries weight because he's a language designer — someone who thinks about tooling from the compiler's perspective. He's not impressed by flash.

> "I knew VS Code always felt sluggish, but I didn't realize how good things could really be." — Matt Baker (Principal Engineer)

This is the pitch that actually converts users. Not feature lists. Not AI promises. The visceral difference between a native Rust editor and an Electron one.

> "Lots of subtle innovations (multibuffers, inlay hints, collaboration). Thoughtful, precise design." — Mike Bostock (Creator of D3.js)

Bostock's eye for design precision makes this significant. He's calling out the compound effect of many small, well-considered decisions rather than any single headline feature.

> "I was able to go from idea to running experiment code in half an hour" — Ethan Perez (Adversarial Robustness Research Lead)

This is the AI-era productivity claim in one sentence. The editor itself recedes; what matters is idea-to-result latency.

## Key Themes

- #tool — A code editor, but one where AI is constitutive, not additive
- #concept — Edit Prediction via Zeta2: an open-weight model that predicts your next edit, not just the next token
- #pattern — Model-agnostic agent integration: Claude Agent, Codex, OpenCode via ACP; any MCP server
- #concept — Multibuffer editing: compose excerpts from across a codebase into one editable surface. This is the most original non-AI feature

## Critical Analysis

**Speed is the real moat, not AI.** Every editor is adding AI features. Claude Code has agentic editing. Cursor has inline AI. VS Code has Copilot. What Zed has that none of them can match without a rewrite is native Rust performance. The AI features are increasingly table stakes; the architecture underneath them is the differentiator that compounds.

**The "any agent" bet is smart but risky.** Zed doesn't lock you into one AI provider — ACP support means Claude Agent, Codex, and OpenCode all work. This is the right architectural instinct (protocols over products) but the risk is becoming the Switzerland that nobody optimizes for specifically. VS Code + Copilot is a tighter integration than VS Code + Zed's ACP bridge can ever be.

**Zeta2 as Edit Prediction is more interesting than inline chat.** Every editor has an AI chat panel now. Edit Prediction — an open-weight model that anticipates what you'll do next — is a different category of AI assistance. It's proactive rather than reactive. If it works well enough, it changes the interaction model from "ask AI to do something" to "AI suggests, you approve." That's a bigger UX shift than it sounds.

**The pedigree cuts both ways.** The team built Atom, which lost to VS Code. They built Electron, which enabled VS Code to eat Atom's lunch. They built Tree-sitter, which is genuinely foundational technology. The question is whether this team can build a *product* that wins adoption, not just excellent *technology*. Atom was excellent technology. Tree-sitter won because it was a library, not a product.

**VS Code's dominance is the structural obstacle.** Developer tools have network effects: extensions, community, docs, Stack Overflow answers, team familiarity. Zed has to be meaningfully better — not 20% better, but qualitatively different — to overcome switching costs. Native speed plus AI-first architecture might be enough. Might not.

**The Zeta2.1 claims deserve scrutiny.** "3x Fewer Tokens, 50ms Faster" is marketing without methodology. Fewer tokens than what? Faster on which benchmark? This matters because edit prediction quality is what differentiates Zed's AI experience from "we also have a chat panel."

## Cross-References

- [[VTcode]] — Also uses ACP for agent integration; Zed and VTcode are betting on the same protocol layer
- [[Zeroclaw]] — Another Rust codebase betting on native performance for agent infrastructure
- [[Dev Containers]] — Zed supports dev containers as first-class infrastructure
- [[Collaborator]] — Desktop agent canvas; a different answer to "what's the AI-native development surface?"
- [[Pencil]] — MCP-native design canvas inside the IDE; Zed's extension model could enable similar workflows
- [[Pi Coding Agent]] — Another tool wrestling with "minimal vs. platform" tension
- [[Compound Engineering]] — Zed's architecture choices (protocols not products, native not web) are compound engineering bets: each one may be individually small but they compound into a meaningfully different experience
- [[Claude Code Cheat Sheet]] — The terminal-agent approach vs. Zed's GUI-native approach; these are competing visions of AI-assisted development
- [[Coding Agents and Complexity Budgets]] — Lee Robinson's insight that "agents need grep, not GUIs" is the case against Zed's GUI-first approach
- [[AI Zealotry]] — The emotional register of switching editors for AI features
- [[Simplicity in the Age of AI-Assisted]] — Zed's Rust-native architecture as a simplicity bet: don't inherit Electron's complexity

---
*Sources: [[summary/zed-dev]], https://github.com/zed-industries/zed*
*Last updated: 2026-05-15*
