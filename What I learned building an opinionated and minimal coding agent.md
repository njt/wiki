# What I learned building an opinionated and minimal coding agent

Mario Zechner's detailed account of building "pi," a coding agent harness from scratch with a deliberate philosophy of minimalism. Four tools only (read, write, edit, bash). No MCP support, no sub-agents, no background bash, no plan mode, no to-do lists, no permission prompts. Full YOLO mode by default. Competitive with Claude Code, Codex, and Cursor on Terminal-Bench 2.0 benchmarks.

---

## Key Quotes

> "If I don't need it, it won't be built."

> MCP servers like Playwright (13.7k tokens) and Chrome DevTools (18k tokens) consume 7-9% of context window before work begins. CLI tools with README files offer better token efficiency through progressive disclosure.

> Terminal-Bench's own minimal agent (Terminus 2), which merely provides tmux sessions without sophisticated tooling, performs comparably across diverse models.

## Key Themes

#minimal-agents #coding-agents #context-engineering #security-theater #benchmarks

This is the most philosophically coherent piece in this batch. Zechner's arguments:

**On tools:** Four tools (read, write, edit, bash) outperform elaborate tool collections because frontier models were trained on similar schemas. Adding more tools costs context window and adds complexity without corresponding capability.

**On MCP:** MCP servers eat 7-9% of context before any work happens. CLI tools with progressive disclosure (agent reads the README when needed) are more token-efficient. This is a direct challenge to the MCP ecosystem that tools like [[Agency]], [[Dorothy]], and [[klaw.sh]] invest heavily in.

**On security:** Permission systems are security theater. Once a model can read files, execute code, and access networks, preventing exfiltration is impossible through UI restrictions. The only real security is containerization. See [[yolo-cage]] for the infrastructure-level alternative he's pointing at.

**On sub-agents:** Black-box orchestration prevents debugging. When you need sub-agents, invoke pi itself via bash within tmux -- preserving full observability. This challenges [[Cord]]'s and [[Dorothy]]'s multi-agent architectures.

**On benchmarks:** Terminus 2 (just tmux) performing comparably to sophisticated harnesses suggests "elaborate tool abstractions may not confer meaningful advantages over simple command execution environments." This pairs with [[Benchmark Exploitation]]'s broader skepticism about what benchmarks actually measure.

## Critical Analysis

Strong: the benchmark evidence supporting minimalism is compelling. The argument that MCP servers waste context is empirically grounded. The philosophy of "build nothing you don't need" produces a tool that's actually understandable by its author -- a rarity in this space.

Weak: the "full YOLO by default" stance is fine for a solo developer running in containers but dangerous as a recommendation. Zechner acknowledges containerization is the real security, but pi doesn't ship with containerization -- it ships with no security and a suggestion to containerize. The deliberately no-MCP stance means pi can't leverage the growing ecosystem of tool servers, which may become a significant limitation as MCP matures.

This is the most important architectural argument in the coding agent space right now: does complexity serve capability, or has the ecosystem added complexity faster than it added value?

The productized Pi is now documented at pi.dev. See [[Pi Coding Agent]] for the official docs, ecosystem, and SDK — and the tension between the original minimalist philosophy and the shipping product's extension surface.

---
*Sources: [[raw/what-i-learned-building-minimal-coding-agent]], [[Pi Coding Agent]]*
*Last updated: 2026-05-15*
