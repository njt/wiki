# Fighting the AI Brain Fry (Kuhne)

Sergey Kuhne's fix for the exhaustion of juggling twenty parallel coding-agent threads: a git-backed priority queue where an agent works the board in cycles and all human judgment funnels through one owner inbox — with "closing" each item, context and all, as the load-bearing trick.

---

The failure mode is familiar to anyone running several terminal agents at once: replies spawn follow-ups, follow-ups spawn tabs, and suddenly your brain is the context window. Kuhne's diagnosis is that this way of working has two structural problems — *sequencing* and *context* — and his answer is to treat his own attention like a priority queue rather than a round-robin across tabs.

His architecture is notable for how boring and durable it is:

- **The queue lives in git.** Every ticket is a markdown file next to the code it talks about; the board's state is a committed snapshot. "Nothing important lives in a chat window and nothing lives in my head."
- **An agent works the board in cycles** — research, build human-tagged-safe tickets, run tests and a browser check, verify what's deployed, and branch-and-ask for sensitive code. It never deploys to production.
- **One owner inbox.** A script reads the whole board and hands him a prioritised list of exactly the items waiting on his judgment: review, answer, or rule.
- **One reading surface.** A Claude artifact rebuilt from git each cycle mirrors the board plus a review page per ticket, so he never reconstructs a change by scrolling a terminal. The page is a viewer; git is the record.

The part he calls "the trick that matters most" is *closing*: take one item, load its context, decide, put it back in the queue — and the context closes with it. One thing at a time. He is honest that it's still exhausting, just no longer scattered, and that the hard remaining problems are tooling ones: serving ticket context so loading takes a minute not twenty, and preventing tickets from looping while blocked on a ruling only he can give.

---

## Key quotes

> "You find yourself hopping between 20 threads, trying to keep all those contexts in your brain RAM. Until you can't."

Names the phenomenon most practitioners feel but few describe: the human as the memory bus between parallel agent sessions.

> "Nothing important lives in a chat window and nothing lives in my head."

The whole design in one line — state externalised to git, attention reserved for judgment.

> "I take one item. I load its context. I make a decision or request an action. And I close it… The context closes with it."

Closing as the mechanism: it's not throughput that saves him, it's that decisions terminate and release their context.

> "It never deploys to production — that part is mine."

A clean one-line autonomy boundary, drawn where blast radius is highest rather than where capability ends.

## Key themes

#concept #pattern #project

- **Human attention as a queue, not a switchboard** — sequencing and context as the two structural failures of tab-juggling.
- **Git as the single source of truth** — tickets, verdicts and board state as committed artifacts; every surface a viewer over the record.
- **Judgment funnelled to one inbox** — the agent can do almost everything except decide; the script surfaces exactly what waits on a human.
- **Honest residual cost** — still exhausting, tooling still raw, tickets can loop while blocked on the one judge.

## Analysis

This is a refreshingly humble artifact in a field full of dark-factory maximalism. Kuhne hasn't built an autonomous pipeline and subtracted himself; he has built a *decision surface* for himself and automated everything around it. The git-as-record discipline is the strongest idea here — verdicts returning to the card means the audit trail and the working context are the same object, which is exactly what makes tickets cheap to load later. The owner inbox is essentially a personal escalation queue, and the admission that tickets loop while waiting on him is the tell that the real bottleneck isn't agent capability at all — it's ruling throughput, which no harness can fix.

The piece also quietly argues against the tab-metaphor tools themselves: the terminal multi-session UX actively manufactures brain fry by making parallelism free and closure expensive. His counter-design makes parallelism cheap for the *agent* and closure cheap for the *human*. That asymmetry — optimise the machine's concurrency, protect the human's seriality — deserves to be a design principle for every agent harness.

## Related pages

- Strengthens [[108 PRs in Eight Days — Accidentally Discovering Loop Engineering]]: same markdown-board-plus-agent-cycles pattern, but Kuhne adds the piece Ellich's loop only implies — a prioritised human-decision inbox as the bottleneck's intake.
- Complicates [[Own the Outer Loop]]: Kuhne operationalises outer-loop ownership as a single git-tracked inbox with a closing ritual, showing what "owning the outer loop" looks like at one-person scale.
- Nuances [[Human-in-the-Loop is Tired]]: Summers diagnoses supervision fatigue as reward-function breakage; Kuhne's closing ritual is a concrete fatigue mitigation — not fewer judgments, but judgments that terminate and release context.
- Extends [[Optimizing for Decision Points]]: Simister argues for surfacing taste-sensitive decisions; the owner inbox is exactly that surfacing, automated by a script that reads the whole board.

---
*Sources: [[raw/fighting-ai-brain-fry]], [[summary/fighting-ai-brain-fry]]*
*Last updated: 2026-11-09*
