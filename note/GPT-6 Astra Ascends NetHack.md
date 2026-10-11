# GPT-6 Astra Ascends NetHack

kenforthewin's report of the first recorded NetHack ascension by an LLM agent: GPT-6 Astra, run as a coding agent, won NetHack 3.6.7 on Hardfought on September 21, 2026, after 37,140 turns — and crucially, the author wrote none of the harness. The agent built its own terminal interface, checked-batching movement guards, route/Sokoban helpers, and Markdown file memory, then repaired those tools mid-run when they broke.

---

## What happened

Three runs. Run 1 died sliming on dungeon level 51 (33,302 turns); run 2 died to a death ray at the Castle; run 3 ascended at 37,140 turns. The author had spent January handcrafting a Python harness around the NetHack Learning Environment and topped out at dungeon level 10 — "The LLM won, and I've still never ascended." This time his entire contribution was a loose brief: play NetHack through a terminal, stream to Twitch, spoilers fine, batch commands, persistent memory is your problem.

## Key quotes

> "I didn't write a single line of code, and honestly I didn't even read Astra's code — and it got pretty darn close to winning on its first attempt, and succeeded on its third attempt."

The inversion of the entire harness-engineering premise. The author arrived intending to be the harness architect and discovered the agent was one. This is the strongest field data point yet for the thesis in [[Harness Engineering for Self-Improvement]] — that a strong coding agent can exploit harness design as a search space — arrived at independently, through a game.

> "That's not the same as having no harness. By the end, the harness was substantial. It is the difference between me designing a task-specific system and the agent doing that engineering as part of the task."

A useful scoping that prevents overclaiming. The scaffolding helped; producing the scaffolding was itself work the LLM could do. Harness design did not become unnecessary — it became delegable.

> "A batch meant fewer round trips through the model, not skipping all intermediate observations."

The guard design is the technical heart of the piece: preflight and post-step predicates over parsed terminal state (HP, position, nearby creatures, unexpected displacement) gate each step of a batched movement — deterministic checks, not a second LLM judging safety. This is exactly the guardrails-over-judgment pattern the wiki keeps converging on: the model decides *what to do when a batch stops*; code decides whether it stops.

> "This was plain Markdown and JSON on disk, not a fancy memory service. The difficult part was keeping the state accurate."

Memory engineering as mundane bookkeeping: a run journal plus compact emergency reference, run-scoped so a dead character's map knowledge didn't contaminate the next one, with working instructions for what to read after compaction. Reinforces [[Agent Memory]]'s point that the hard problem is judgment about state, not storage — here the agent had to distinguish observed from estimated wand charges and update stolen items promptly, and the memory was maintained, not merely retrieved.

> "Its ability to write and maintain ordinary software was part of its ability to play the game."

The sentence that generalizes beyond NetHack. Competence at a domain was mediated by competence at software: when the status parser failed on the elemental planes, the agent extended it, tested it, checkpointed it, and kept playing.

## Honest caveats

The author is unusually scrupulous. Open-book access to spoilers and NetHack source — this is not a closed-book knowledge test. No fixed budget; tools changed during play, so no clean attribution across model, harness, memory, and compute. BALROG's 13.2% average NetHack progress measures a different protocol. The "first ever" claim is hedged against BotHack's 2015 hand-coded ascension and rests on found records, not an exhaustive registry. The audit evidence (local hash chain plus independent Hardfought server recordings) detects inconsistencies but doesn't prove the machine's owner couldn't intervene — "the agent built the harness and played the game" must not expand into "no human ever said anything useful."

## Analysis

What makes this worth ingesting is not the flex but the shape of the result. Every piece of the harness — terminal observation with annotation, checked batching, graph-search helpers, file memory, audit logging — is deliberately ordinary software, and the piece's implicit claim is that this ordinariness is the point: an agent that can write and fix ordinary software doesn't need a bespoke interface built for it, it needs a boundary that keeps it honest (the remote terminal, not the internal game state) and permission to improve its own tools. The failure the author highlights — a fixed interface turning a mundane parser bug into a hard stop the player can't repair — is a general argument for agent-modifiable scaffolds over sealed ones.

The honest-caveats section is itself a pattern worth noting: an agent result reported with explicit non-claims, cost disclosure (~$500 total), and verifiable evidence rather than a screenshot. Compare that posture with the benchmark-bragging norm.

#concept #tool #pattern

## Relation to the wiki

This source strengthens [[Harness Engineering for Self-Improvement]] with a dramatic field confirmation of its central thesis — the harness as the exploitable design space — arriving from a game rather than a research lab. It nuances [[Agent Memory]] by showing file-based memory working at 37K-turn scale, with accuracy-of-state (not storage architecture) as the hard part, and it echoes [[I Stopped Coding and Started Architecting Agents (And Why You Should Too)]] in reverse: Bache argues the human should build the harness for the agent; here the agent built the harness for itself, which suggests Bache's flywheel is a stage, not a destination.

---

*Sources: [[raw/llm-nethack-ascension]], [[summary/llm-nethack-ascension]]*
*Last updated: 2026-10-11*
