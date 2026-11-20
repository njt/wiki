# LLM Assisted Reverse Engineering (slowchronicle)

A two-decade hardware and software hacker's field report on how LLMs rebuilt his reverse-engineering practice in 2026: a DeepSeek + Ghidra-MCP stack, markdown documentation as compaction insurance, a self-written "hypothesis debugging" skill that forces a stuck agent into a scientific-method loop, and six months of results that would previously have taken years each.

---

The author's workflow is disarmingly simple: point the agent at a binary, use dedicated skills, and let the loop run. The substance is in the failure handling. Base analysis (binary type, main functions, imports) proceeds by disassembly alone — he notes current models handle pure x86/ARM surprisingly well. When stuck, he escalates in a fixed ladder: dynamic breakpoints, injected logging probes, giving the agent outside access (computer use, emulator APIs), asking the agent to switch approach *specifically*, and — the most human move — reading the agent's own conversation traces to spot it heading into a blind alley.

Three artifacts carry the piece:

**Documentation as memory.** Every finding goes to markdown files, serving three purposes: notes for the human, continuity across compaction and agent handoff, and self-steering so the agent doesn't go in circles.

**Hypothesis debugging.** His own published skill forces a stuck agent to write a hypothesis into an on-disk CSV table, design a decisive experiment (probe, logging), record the result, and repeat. "No endless guessing and trying random things." This is scientific method as a harness constraint — the agent's own work becomes its verification loop.

**Tooling choices justified by agent ergonomics.** Rust is chosen not for performance but because "the compiler is very strict and points the agent immediately to any possible memory or syntax issues" — compiler strictness as a fast feedback loop for the *agent*, not the human. Python+UV for throwaway probes and decompilation helpers.

The results list is the credibility: Arturia MiniFreak and OP-Z custom firmware, full protocol maps of Sony Alpha cameras and an EmWave biofeedback device, an Alphasmart NEO hack, cracked abandonware. He closes on the dissolving boundary — decipher a proprietary save format and you slide into ordinary software development, building a whole new editor (his own Photoshop, he notes, a year before the ArtCraft suite).

---

## Key quotes

> "Rest usually takes care of itself."

On the maturity of the loop — pointing an agent with the right skills at a binary is now sufficient setup for the base pass.

> "They help the agent not lose the track after compaction / handoff to different agent."

The markdown-documentation habit is precisely the compaction-amnesia fix catalogued at length in [[Decompiling MW2 with Four Claude Agents]] — arrived at independently by a solo RE practitioner.

> "This is variation of scientific method and it significantly helps agents when they are getting stuck... No endless guessing and trying random things."

Hypothesis debugging is the standout idea: forcing the agent to commit to falsifiable hypotheses turns an open-ended search into an experiment ledger on disk.

> "it provides brutally good feedback loops to the agent when it iterates - the compiler is very strict."

A tool-selection criterion almost never stated this plainly: choose languages for the quality of the error signal *the agent* receives, not the developer.

## Themes

#concept — hypothesis-driven debugging as a harness pattern; markdown documentation as external memory across compaction
#tool — Ghidra MCP bridge, radare2, Kuna, DeepSeek harness, UV, Rust
#pattern — escalation ladder for stuck agents; reading agent traces as human supervision

## Analysis

This is the rare practitioner writeup where the interesting content is almost entirely orthogonal to the model choice. DeepSeek appears for cost and speed; the actual technique lives in the skills and the discipline around them. The hypothesis-debugging skill deserves to be abstracted well beyond RE: it is a cheap, general-purpose guardrail for any stuck agent, converting unbounded thrash into a bounded experiment loop — kin in spirit to the deterministic verification loops elsewhere in this wiki, but requiring no infrastructure beyond a CSV file.

It also complicates the "agents need boring, training-data-dense stacks" argument. RE is the opposite of that: proprietary binaries, no documentation, adversarial formats — and yet agents thrive, because the feedback comes from the artifact itself (disassembly output, breakpoint hits, probe logs) rather than from training-data familiarity. The binary is the oracle.

The honest coda is that expertise still steers: he explicitly credits his twenty years of experience with spotting when the agent is "going into a blind path," and calls reading the traces "skill of your own to have." LLMs didn't remove the need for deep assembly knowledge here — they removed the tedium of applying it.

## Relations

- [[Decompiling MW2 with Four Claude Agents]] — the large-scale multi-agent version of the same discipline; this solo writeup independently arrives at the same compaction-amnesia fix (markdown documentation) and confirms it at one-hundredth the scale.
- [[Ghidra MCP Server (bethington)]] — the concrete infrastructure this author credits as "the best in the market" once agents can drive it via the MCP bridge; this piece is field evidence for that server's design bet.
- [[Kuna — Agent-First Decompiler]] — the author lists Kuna as a faster, possibly more agent-usable CLI alternative to Ghidra; a practitioner data point on the tool this wiki covers from the builder's side.
- [[Giving Your Agent Eyes with Game Boy Hacking]] — same finding from a different hobbyist RE project: give the agent an observable feedback loop over the artifact and surprising behavior falls out; here the probes, emulators and breakpoint access are exactly that loop.

---
*Sources: [[raw/llm-assisted-reverse-engineering]], [[summary/llm-assisted-reverse-engineering]]*
*Last updated: 2026-11-20*
