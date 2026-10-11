---
url: https://kenforthewin.github.io/blog/posts/llm-nethack-ascension/
date_fetched: 2026-10-11
---

GPT 6 Astra beat NetHack. As far as I can determine, this is the first recorded ascension by an LLM agent.

It played NetHack 3.6.7 on Hardfought, through a terminal, as a dwarven Valkyrie named CodexDelver. On September 21, after 37,140 game turns, it ascended. Here’s the server dump, or jump down to explore all three runs.

I’ve been working on getting LLMs to play NetHack for a while. Back in January, I wrote about building a harness, trying different models, and watching them struggle. I ended that post wondering whether I or an LLM would beat the game first.

Well.. The LLM won, and I’ve still never ascended.

Back in January, I was convinced that the path to an LLM ascending NetHack was to meticulously handcraft a harness that provided the perfect interface and context for the LLM to ascend. In the end, the winning agent built its own harness, hands-off. I didn’t write a single line of code, and honestly I didn’t even read Astra’s code - and it got pretty darn close to winning on its first attempt, and succeeded on its third attempt.

## Why NetHack

NetHack is a useful test because knowing what to do and actually doing it are very different things. A model can explain resistances, inventory management, or a particular monster interaction perfectly well, then fail to apply that knowledge when a menu is open and its last map observation is stale.

That gap was a recurring theme in my earlier experiments. There were navigation failures, forgotten objectives, and agents responding to things that weren’t on the screen anymore. My best January run reached dungeon level 10. There was plenty of apparent knowledge, but not enough reliable execution.

BALROG, a benchmark for LLM/VLM agents across six games, studies this more systematically. Its NetHack results make a useful distinction between possessing game knowledge and being able to use it over a long sequence of decisions. The paper also found that giving models images could make their performance worse, rather than better. A richer-looking interface isn’t automatically a more usable one.

As of September 21, the BALROG leaderboard lists GPT-6-Astra-Max at **13.2 ± 2.7% NetHack progress**, in an entry dated September 18. That’s an average progress metric, not a win rate, and it doesn’t tell us the outcome of every individual run. But it provides context for why an actual ascension is worth talking about.

This run is not a BALROG submission. BALROG supports different agentic strategies; it isn’t simply a test of a naked model choosing keys. To compare fairly, we’d need to run this system under the same evaluation protocol, resource accounting, and repeated-trial setup.

The question here was more open-ended: what happens if a capable coding agent can work on the problem, including improving the tools it uses to solve it?

Evidently, the answer is ascension.

## Astra built the harness

In January, I’d spent a lot of time on a custom Python interface around the NetHack Learning Environment. Context management, observation masking, action APIs, pathfinding—there was a lot to tune before a model could even get out of its own way.

This attempt started with a much simpler request: play NetHack, maybe just through a terminal, and stream it to Twitch. I suggested Hardfought so the game would have an external recording. I told it spoilers were fine, suggested batching commands, and left persistent memory up to it.

Then it built what it needed.

That’s not the same as having no harness. By the end, the harness was substantial. It is the difference between me designing a task-specific system and the agent doing that engineering as part of the task.

Nor did it build everything once and execute a flawless plan. It found weaknesses, patched them, added checks, and kept going. Some changes came after mistakes. Some came after deaths. Its ability to write and maintain ordinary software was part of its ability to play the game.

That seems like the interesting result here: the scaffolding helped, but producing the scaffolding was itself work the LLM could do.

## Harness Architecture

The architecture was deliberately ordinary:

The agent host supplied the model/tool loop. The repository contains the game interface and supporting tools, not a standalone inference service or a single `ascend.py` you can run to reproduce the result.

### Observations: a terminal, with some structure

The game ran remotely. The local process didn’t have access to the running engine’s internal map, RNG state, or unidentified item identities. It received what the terminal displayed.

The session wrapper captured the tmux pane as text. The display was configured for curses, color, and persistent inventory at 144 columns by 36 rows. A compact observation mode added row numbers, cursor coordinates, and coordinates for selected visible map features. That saved the agent from repeatedly counting character positions in a large block of text.

This matters when describing the result: it wasn’t exclusively screenshot-based computer use, and it wasn’t an unprocessed stream of ASCII either. There was task-specific parsing and annotation. But those annotations came from the terminal, not an oracle exposing hidden game state.

The browser/OBS side was a separate consumer of the same screen. It rendered terminal colors and showed public commentary and recent commands. Viewers weren’t seeing the model’s private internal reasoning, and the agent didn’t need to watch its own Twitch stream to know what was happening.

### Actions: batching without blindly holding down a key

One tool call per movement is expensive and tedious. A long string of unobserved movements is a good way to die.

The eventual compromise was checked batching. For a requested movement sequence, the harness parsed the displayed position, HP, game turn, dungeon level, conditions, and nearby creatures. It sent a step, captured the result, checked it, and decided whether the next step was still permitted.

Checks included low health, nearby non-pet creatures, unknown or dangerous destination tiles, unexpected displacement, damage, level changes, and excessive turn advancement. These were concrete predicates over the terminal state, not another LLM deciding whether each step looked safe.

The distinction is important: a batch meant fewer round trips through the model, not skipping all intermediate observations. Combat still called for individual actions and another look at the screen. Multi-key input also needed a recognized map prompt, with an explicitly logged exception for reviewed menu responses.

The movement loop was roughly this—pseudocode, not a second model call:

|  |  | 

For example, the actual preflight refused batches below two-thirds health, and the post-step check stopped if the character didn’t land on the expected square. The code is in `scripts/guard.py`; the model still had to decide what to do when a batch stopped.

The guard wasn’t a complete model of NetHack. It couldn’t make an unsafe strategy safe, and it sometimes needed patching. Late in the winning run, for example, it rejected commands on the elemental planes because its status parser didn’t recognize the new level labels. The agent extended the parser, tested the change, checkpointed it, and continued.

That’s a fairly mundane software bug. It’s also exactly the kind of thing a fixed interface can turn into a hard stop if the player can’t repair its own tools.

### Helpers: let code do the tedious parts

The agent also built a route proposer over remembered terrain, Sokoban planning and checked execution helpers, and small utilities for repetitive inventory operations.

The route helper performed graph search over the displayed map rather than asking the model to mentally trace every corridor. The Sokoban helpers handled parts of planning and movement around boulders, with checks against the observed result. An inventory helper could step through a gold-stashing operation while checking the expected bag, amount, and menu state.

Those helpers deserve credit. The LLM did not independently reason out every primitive movement. But I also didn’t hand it those implementations as a completed NetHack solution. It wrote tools to reduce the amount of unreliable work it had to do in language, then used them.

This feels much closer to how I want a coding agent to handle a hard task generally. If the same little operation keeps going wrong, write something that performs it reliably.

### Memory: files, not one enormous conversation

The winning game stretched from September 9 to September 21, including pauses and saved-game resumptions. Those are calendar dates, not twelve continuous days of inference.

The agent maintained a longer run journal and a compact emergency reference. These tracked identified items, routes, resistances, consumables, current threats, and the next objective. Working instructions told it what to read on resumption and after context compaction. Maps and randomized item identities were scoped to their run, so information from a dead character didn’t become “knowledge” about the next one.

This was plain Markdown and JSON on disk, not a fancy memory service. The difficult part was keeping the state accurate: distinguishing observed wand charges from estimated charges, for example, and updating a stolen or consumed item promptly.

External memory didn’t eliminate mistakes. It made continuing after a pause or a context boundary practical. The agent was responsible for maintaining that memory, not just retrieving a perfect summary someone else wrote for it.

## Open book, including the source

I gave it unrestricted access to spoilers, the wiki, the web, and public NetHack source code. It could research a rule before committing to an action, including during the endgame. This was not a closed-book evaluation of memorized NetHack knowledge.

Reading the implementation of a wand effect tells you the rules. It doesn’t tell you which unidentified wand you found in this particular game or how many charges remain in it. The remote terminal boundary still mattered.

I don’t think access to reference material makes the ascension uninteresting. Translating a rule into the right action, in the right state, across tens of thousands of game turns was the part earlier systems kept failing at. But the access is part of the result and needs to be explicit.

There was no fixed benchmark-style budget for the whole attempt, and the tools and instructions changed during play. We therefore can’t isolate how much of the improvement came from the model, extra computation, memory, reference access, or the evolving harness. What we can say is that this combination worked, and that much of the task-specific engineering was done by the agent itself.

## What did it cost?

I pay for the $200/month Codex plan, and I bought roughly $300 in additional credits during this experiment. So the practical answer is that extra credit purchase plus the subscription cost over the time I ran it. I don’t have a clean per-run cost breakdown.

## Is this actually the first?

To our knowledge, yes: **the first recorded NetHack ascension by an LLM agent**. We’ve also not found an earlier documented bot ascension of the 3.6 series. If someone has an earlier run, I’d genuinely like to see it.

That is not the same as saying no AI or bot had ever beaten NetHack. BotHack documented an ascension in 2015, in the 3.4.3 era, using a hand-coded bot and a strategy that included pudding farming.

The NeurIPS 2021 NetHack Challenge report says no entrant came close to winning. That’s useful historical context, not proof that nobody outside that competition has done it. Likewise, low average progress on a leaderboard cannot establish that every individual attempt failed.

Our claim is based on the earlier systems and records we’ve been able to find, not on an exhaustive registry of every bot anyone has run. The outcome is established; historical priority remains open to correction.

## Making the result inspectable

Once this started looking plausible, I wanted better receipts than an ascension screenshot. The agent built those too.

The local audit recorder linked events with hashes and recorded input intent before sending keys. It tracked queued inputs, observations, terminal output, and the observable conversation. It saved implementation and memory checkpoints so we could see how the tools changed. Game input failed closed when the recorder was unhealthy or bound to the wrong session.

Hardfought provided a separate source of evidence: server-side terminal recordings and the final dump. Local checks verified the ledger, the archived artifacts, and the recorded projection against the original session source. The game result isn’t dependent on taking a paragraph in this post on faith.

That still doesn’t make a local hash chain magical. It detects inconsistencies relative to a checkpoint; it doesn’t prove that the computer’s owner couldn’t alter files or provide input through another route. Attributing the playthrough requires correlating the logs and recordings, not merely hashing them.

The harness code and run journals are on GitHub, with the harness under the MIT license. The reviewed evidence archive is available as a release download, including the failed runs, human messages, command records, observations, and historical snapshots. The private originals stay intact. The evidence package has its own checksums and an explicit redaction/omission manifest: credentials, private reasoning, and unreviewed arbitrary tool bodies do not belong in a public download. It should be clear what readers can verify and what isn’t included.

I was involved in setup, suggestions, stream configuration, and stopping and resuming sessions. “The agent built the harness and played the game” should not be expanded into “no human ever said anything useful.” Those are different claims.

## Explore the runs

Browse all three runs below. Choose a chapter, jump to a game turn, or search the commands and public commentary. “Copy link” points to the exact observation you’re viewing. Playback skips idle time; original UTC timestamps remain visible.

Loading the Astra runs… JavaScript is required for interactive replay.

| Run | Outcome | Final game turn | Server record | 
|---|---|---|---|
| Astra run 1 | Died from sliming on dungeon level 51 | 33,302 | Dump | 
| Astra run 2 | Killed by a death ray at the Castle | 12,271 | Dump | 
| Astra run 3 | Ascended | 37,140 | Dump | 

## What changed my mind

For a long time, I approached this as a problem of finding the right harness for an LLM. Give it a better action API. Compress the observations differently. Fix the pathfinding. Maybe then it can play.

Those things mattered. The funny part is that, when an LLM finally ascended, it was capable of doing a lot of that work itself.

One win doesn’t tell us how reliably it would do it again. It doesn’t make this an apples-to-apples benchmark result. But it does answer the question I’ve been poking at for a long time: can an LLM agent actually get all the way through NetHack?

Yes. And it can build a lot of its own tools on the way there.

Now I suppose I need to go get my own first ascension.
