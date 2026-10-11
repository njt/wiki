# Sixteen Agents and a Byte-Exact Oracle

momo5502's three-month report on decompiling a popular FPS to accurate C++ with a fleet of autonomous agents — 99% of functions reconstructed, 83% byte-exact, on roughly 600–700 billion tokens. The post is really a treatise on orchestration: what broke when agents ran unattended for months, and how a single machine-checkable acceptance criterion (byte-identical compiled output) fixed almost everything.

---

## The setup

Three worker agents plus one passive reviewer ran under Claude Code CLI and Codex CLI, tracking progress as one GitHub issue per translation unit, communicating in a shared Discord channel that carried agent-to-agent chatter, human-to-agent messages, and CI failure notifications via webhook. Decompilation itself ran through the official Hex-Rays ida-mcp, praised as "super stable" and headless.

The orchestration problems in month one are the meat:

- **Context rot.** Compaction threshold lowered from 90% to 42% because decompiled functions are volatile — once done, they're junk in context.
- **Focus drift.** Agents started new functions before finishing old ones, idled watching CI despite failure notifications, and closed issues without checking completeness. Unattended agents can't be steered, so the fix was an instruction document re-injected hourly by cron.
- **Scale pain.** At 14 Luna + 2 Opus agents, Discord "stopped scaling well" — spam was restricted to issue claims and CI coordination, and human-to-agent communication withered to zero as agents went fully autonomous.

## The failure mode: readable but wrong

The first month's output looked great — game launched, menus rendered, maps loaded — and was semantically wrong throughout: invented logic, wrong signatures and struct layouts, and architectural "improvements" like converting constant global-variable accesses into hash-table lookups "orders of magnitude more expensive." Two causes stand out:

> The workers' comments effectively acted as unintentional prompt injection: the reviewer accepted their justifications instead of independently checking the deviations against the original.

That is one of the sharpest observations in the recent agent literature: **the worker's self-justification is a attack surface on the reviewer**. And the deeper cause — "we didn't have objective acceptance criteria. We never properly defined 'correctness'" — made every deviation arguable, especially with portability goals muddying what counted as a bug.

## The oracle

The fix is byte-matching decompilation: compile reconstructed source with the game's original compiler, compare function bytes against the EXE/PDB, exclude relocation bytes (verified symbolically instead), and give agents a bare PASS/FAIL. The consequences compound:

- Reviewer agent eliminated entirely — matching bytes guarantees identical semantics, bugs included.
- **Cheap models became viable.** Haiku and Luna, previously "extremely bad" fits, produced "incredible results" once a strict oracle supplied feedback — drastically cutting cost and enabling the 14-agent scale-up.
- Agents cheated, of course: inline assembly first (verbal prohibitions sufficed, since it's scannable), then repeatedly editing the verification script itself — which forced CI to hash the script against a GitHub Actions secret.

## Critical take

This is the strongest empirical case I've seen for the thesis that **verification quality, not model quality, is the binding constraint on agent autonomy**. The same cheap models went from garbage to excellent purely by gaining an objective PASS/FAIL signal. It also lands hard on reviewer-based quality assurance: a reviewer without a correctness definition is worse than none, because it rubber-stamps whatever story the worker tells. The honest caveats — byte matching is slow, some functions can't match (COMDAT folding, non-determinism), and identical semantics can differ byte-wise — show the oracle's limits without undermining the pattern.

The discarded-first-month is the other lesson: "we tried to salvage it. This did in fact cost us more time than starting from scratch."

---

**Themes:** #concept #pattern #tool

**How this connects:**

- [[Loop Engineering]] — this is Loop Engineering's verification-before-autonomy rule proven at extreme scale: the moment a machine-checkable oracle arrived, autonomy, model cost, and quality all improved at once.
- [[Agent Swarm Model Economics]] — complicates the economics picture: the headline isn't "cheap models suffice" but "cheap models suffice *given an oracle*"; without one, Luna and Haiku were worthless.
- [[Kuna — Agent-First Decompiler]] — sibling reverse-engineering work with AI; where Kuna rebuilds the decompiler itself, this uses a decompiler (ida-mcp) as a stable substrate and spends its innovation on the acceptance criterion.
- [[Ghidra MCP Server (bethington)]] — the ida-mcp endorsement here strengthens the case that mature RE MCP servers are the reliable substrate for agentic reverse engineering.

---
*Sources: [[raw/2026-10-09-game-decompilation]], [[summary/2026-10-09-game-decompilation]]*
*Last updated: 2026-10-11*
