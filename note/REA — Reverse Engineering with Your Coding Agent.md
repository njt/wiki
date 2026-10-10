# REA — Reverse Engineering with Your Coding Agent

REA gives a coding agent the tools to inspect a running program and explain what it does, packaging reverse engineering as an agent capability instead of a specialist discipline. The landing page proves it with two verified examples — recovering Chrome dinosaur's acceleration rule from live JS, and Windows Calculator's `%`-button semantics from x86-64 assembly — each ending in a rebuilt artifact that reproduces the original behaviour.

---

The pitch is one sentence: "REA gives your agent the tools to inspect a program and explain what it does." The interesting design choice is distribution — there is no product to download. You paste a prompt into your coding agent and the agent itself runs `npx rea-agents@latest setup`, shows you the setup plan, and verifies its own installation. The tool assumes from minute one that the agent is the operator and you are the approver.

Two worked examples carry the page, and both are careful to show **verification**, not just extraction:

> "We called the original game's update function in a controlled browser check, with obstacles and automatic scheduling disabled. After 4,000 updates, speed was 10.0; after 10,000, it was 13.0."

That is the dinosaur game: REA pulls the actually-loaded `index.js` with a digest over a local debugging connection, the agent reads out `ACCELERATION: 0.001, MAX_SPEED: 13, SPEED: 6`, and the claim is then checked against the running original before a rebuilt mini-game keeps the same rule. The Calculator example goes further, from decompiled branch logic (`IDC_MUL 92`, the `0x64` = 100 constant) up to a human-comprehensible rule: after `+`, the percentage is taken of the previous operand; after `×`, it becomes a multiplier. 200 + 10% = 220 stops being a mystery and becomes recovered code.

**Key themes:** #tool #pattern #concept

## What is actually novel here

Reverse engineering tooling is old (IDA, Ghidra). What is new is the *interface*: REA exposes native binaries, JavaScript/Electron/ASAR, and browser runtime activity as things a general-purpose coding agent can drive, with the LLM doing the hypothesis-forming that used to be the reverse engineer's craft. The page's provocations make the ambition plain — "clone this website for me", "reconstruct this game from its executable" — understanding as a precursor to rebuilding, not an end in itself.

## The critical read

- **The demos are curated.** A toy game and a calculator are maximally legible targets. Real RE work is dominated by ambiguity, packed binaries, and anti-analysis — none of which a landing page has to survive. The "trace the CSV export of our Notes app" starter is the honest tier of the offer.
- **Verification is the load-bearing rhetoric.** Both examples end in a controlled check against the original, which is exactly the discipline the wiki's harness-writing argues for — and exactly what most "agent understands your codebase" products skip. Whether REA enforces that loop or merely demos it is unknowable from a landing page.
- **It quietly answers "why does legacy resist agents?"** Agents fail where they can't see. REA is a bet that the missing sense is *runtime observation of programs you have no source for* — the complement to source-level context engineering.

## Related pages

- [[Kuna — Agent-First Decompiler]] — the closest cousin: Kuna rebuilds the decompiler itself around an LLM, while REA keeps classic RE tooling and hands it to a general agent; together they bracket the question of whether the model lives inside the tool or beside it.
- [[Understand to Participate]] — Litt's argument that understanding is the prerequisite for remaining an active collaborator; REA operationalises exactly that for code you don't have the source to.
- [[The Archaeologist's Copilot]] — brownfield archaeology on codebases you *do* have; REA extends the same instinct across the binary boundary.
- [[Giving Your Agent Eyes with Game Boy Hacking]] — the same pattern of agent-driven inspection of a running system, one level of abstraction lower and considerably more delightful.

---
*Sources: [[raw/rea-tools]], [[summary/rea-tools]]*
*Last updated: 2026-10-10*
