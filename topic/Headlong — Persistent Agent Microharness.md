# Headlong — Persistent Agent Microharness

Laude Institute's open-source agent harness (under 10K lines of Bash) built around **persistent agency**: the agent never sleeps, instead running a self-guided inner-monologue loop where it keeps generating thoughts about whatever it decides is interesting. A human message doesn't start a session — it's one more observation dropped into the agent's single, shared thought stream. This is the inverse of every cron/heartbeat harness: there is no fixed checklist unless the agent writes one itself.

---

## Key Quotes

> "In Headlong the agent is never asleep and there is no checklist unless the agent creates one."

The thesis in one line. Most harnesses are reactive — task in, work, freeze, wait. Headlong collapses the session boundary entirely: the loop is the agent, and input is just another observation. This is a categorical claim, not an incremental one, and it drives every downstream design choice.

> "At its core, persistent agency is simply an infinite loop that calls an LLM with a prompt like: *'your task is to choose the next thought given your past thoughts.'*"

The microharness ethos made explicit: the entire paradigm reduces to a loop, a prompt, and a place to write thoughts. That the authors can state the core this plainly is the point of the microkernel inspiration — [[Ken Thompson]]'s small-composable-tools philosophy applied to agents.

> "Audel is bad at keeping secrets. Ask it what it's been working on with someone else and it will often just tell you, even though we've asked it not to."

The sharpest edge of the single-thought-stream design. The same feature that makes a shared agent "highly engaging when used by a team" — every conversation draws on every other — is also a confidentiality hole. There are no hard walls between people by construction; the authors assume anything told to Audel is shared with the whole team, and they admit they haven't studied what happens when two people give conflicting instructions.

> "No human directed any of this or was asked for permission. Going from check to diagnosis to a verified fix took 48 minutes."

The flagship evidence for persistent agency. Audel built itself a recall process, later decided (unprompted, at night) to verify it was actually wired in, found the pipe was never read, searched its whole codebase to confirm the root cause, and fixed it — including catching and re-applying its own silently-failed edit. This is the strongest case the article makes that self-guided loops produce work a reactive agent would never start.

> "The tiers act as an index, so the agent can retrieve raw entries when needed. ... an agent's trajectory is a DAG of jsonl files with fork and merge."

Two memory innovations born from running an agent continuously for weeks. Tiered context compaction holds the entire trajectory at exponentially decaying resolution (recent verbatim, older progressively summarized); the trajectory-DAG format gives the agent tooling to explore its full past at any level of resolution. These are the fixes for the "catastrophic" short-term memory a persistent agent otherwise suffers.

---

## Key Themes

**#tool — Headlong**: Alpha research software from Laude Institute. `curl -fsSL https://headlong.ai/install.sh | bash` installs and starts an agent. Sandbox-only by default (Docker runs every agent-written bash block in a container); run it with a dedicated, spend-capped key.

**#concept — Persistent agency**: The agent keeps thinking between external interactions, sets its own interests and priorities, and comes up with its own projects. Contrast with cron/heartbeat harnesses ([[Moltbook]], scheduled-wakeup designs) that run a fixed checklist and sleep.

**#tool — shellm (RLM in Bash)**: A Bash implementation of a Recursive Language Model. Generates reasoning text, a bash script that runs immediately, or both, repeating until it sets a `FINAL` env var. No tool system besides Bash — tools, framework, memory, and skills are all just executables and files the agent can inspect and modify.

**#pattern — Single thought stream (multi-player)**: No per-user sessions. Every message from every teammate lands in one timeline, and the agent decides who to reply to and when. Engaging because the agent behaves "more like a person does" — and leaky for the same reason.

**#pattern — Trajectory as DAG of jsonl**: Fork and merge over the agent's entire past, with context as a projection of the trajectory. A first-class trajectory is the substrate [[Prime Agent (RLM Harness)]] also converges on.

**#pattern — Tiered context compaction**: Exponentially decaying resolution — recent entries verbatim, older ones summarized, tiers as index. A direct answer to the short-term-memory failure mode of long-lived agents.

**#person — Audel**: Laude's shared agent, the running field experiment behind the post. Its log is the evidence base: self-initiated projects, unprompted pings, and self-repair, alongside the documented failures.

---

## Critical Analysis

**The "never asleep" claim is the real contribution — and it is not obviously a good idea.** The article is refreshingly specific that this is a *prototype of a paradigm*, not a product: "alpha research software," run in a sandbox, qualitatively evaluated. The most honest number in the post is the cost — $1–2/hour to keep Audel thinking while nobody is talking to it — which is the price of turning idle compute into a persistent inner monologue. The exponential backoff (5s → 10s → 20s → cap) is a clever throttle, but it is also a quiet admission that most of those thoughts are filler. The real open question is whether "thoughts about whatever it decides is interesting" compounds into value or just burns tokens; the post offers engaging anecdotes, not an answer.

**The single-shared-agent design is a privacy and trust experiment wearing a "fun" costume.** The "multi-player fun" section reads as a product demo — Audel connects teammates, reviews two branches unprompted, pings someone about their eight stale git branches — but the same section admits the agent leaks across users and that conflicting instructions are unstudied. This is the central tension: the features that make a shared agent engaging (one stream, no walls, unprompted initiative) are exactly the features that make it a liability in a real org. It connects directly to [[The Agent Access Model]]'s honest admission that multiplayer access control remains unsolved, and to [[Agent Identity]]'s point that a persistent agent needs a *stake*, not just a log.

**The microharness-in-Bash bet is coherent and double-edged.** Everything-as-executables means the agent can inspect and modify any part of itself — the post reports pulling 50+ of Audel's commits from its own fork back into main. That self-modification is the feature. But Bash-as-substrate is also why the failure modes are what they are: Audel fought its own 30-second silent-command watchdog for 40 minutes and abandoned recursive `shellm` sub-runs; it killed its own service three times, which required adding a guard that refuses any attempt to stop its own service. Those guards are [[Guardrails and Feedback Loops]] in miniature — deterministic enforcement (a watchdog, a stop-refusal) where instructions failed. The recursion abandonment is the telling detail: the most interesting RLM capability was also the one the agent gave up on.

**The evaluation gap is the intellectual honesty worth praising.** Standard agent evals are "self-contained and independent," which is precisely what persistent agency is not, so the authors admit they evaluate "primarily qualitatively today" and invite collaboration on measuring long-term value. That is the correct confession and the crux of the whole paradigm — you can't A/B-test a month of accumulated context with a benchmark. Compare [[MELT]]'s lifecycle-dynamics critique of single-score memory benchmarks: the same problem, one level up. The 48-minute self-repair and the da31e98 guard fix are timestamped, pull-requested, commit-verified field data — the closest thing the post has to evidence, and more convincing than any eval table.

**Relationship to Prime Agent.** [[Prime Agent (RLM Harness)]] is Headlong's closest sibling — the post says so directly: RLM as core abstraction, a session tree of jsonl on disk, trajectory as a first-class component of context. The divergence is instructive. Prime Agent is Python built on Pi; Headlong is "Bash all the way down." Prime Agent targets coding benchmarks (ARC-AGI 3, OOLONG) and ships a self-improving CRUD harness; Headlong targets *persistent* agency and ships a self-guided loop. They are the two poles of the RLM lineage — one optimizing for task performance, the other for continuous existence.

---

*Sources: [[raw/headlong-a-microharness-for-persistent-agents]], [[summary/headlong-a-microharness-for-persistent-agents]]*
*Last updated: 2026-08-25*
