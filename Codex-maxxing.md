# Codex-maxxing

Jason Liu's field report on pushing Codex beyond coding into general knowledge work. The core insight: an agent becomes a force multiplier not when it can write code faster, but when it can keep working after you walk away. He achieves this through a stack of interlocking techniques — durable threads, steering, voice input, Heartbeat automations, and an Obsidian vault as agent memory — that together turn a chat interface into an operating loop.

---

## Key Quotes

> "History, preferences, and old decisions that I do not want to recreate every time I come back."

Liu on durable threads — the same megathread, compacted for months, preserving context that would otherwise be lost. This is the practitioner's version of [[Agent Memory and Context]]: context continuity as work infrastructure, not convenience.

> "The real benefit of voice isn't speed — it's that I get the unedited version of my thinking."

Voice as a fidelity tool, not an efficiency tool. Dictating is closer to how you actually think; typing forces a cleanup pass that loses texture. This aligns with the journaling-adjacent patterns in [[Claude Code on the Go]] and the raw-first philosophy behind several [[Personal Agents]].

> "I do not need to wait for each step to finish before deciding the next one."

On steering — injecting the next instruction while the agent is still working. Combined with Heartbeats, this transforms the unit of work from "a prompt and a response" to "a small operating loop." This is the lived experience behind [[Designing Agentic Loops]].

> "Files force the agent to compress experience into a form that can survive the thread."

The memory architecture thesis. Liu's Obsidian vault structured as a GitHub repo (directories for people, projects, agent files, notes) with an AGENTS.md instructing the agent to update relevant pages. Codex's built-in Memories handle stable preferences; the vault replaces "evergreen threads." The git diff is his review surface for what the agent learned. This is the same pattern as [[LLM Wiki]], [[Immaculate Knowledge Graph]], and this wiki itself.

> "Slack threads, inboxes, and calendars are where a lot of work shows up before it ever becomes code."

Why he wires up $slack, $gmail, and $calendar connectors. The agent meets work where it arrives, not where it ends up. This is [[Intent Is the Interface]] in practice: design for the capability (respond to a Slack thread), not the surface.

> "Ambition without verification is just a wish."

On strong goals with real success criteria. Liu's example: migrating Python's Rich library to Rust, with the goal being "it must pass all the unit tests from the original library." Not "implement the plan" — pass the test suite. This is [[Specifications as the Product]] operationalized as a success criterion.

> "Not that an agent can write code for me, but that more of my work can keep moving after I leave."

The closing thesis. The agent's value isn't tool replacement (write code faster) but time multiplication (work continues during gaps). This reframes the entire agent discussion from productivity to continuity.

---

## Key Themes

**#concept Heartbeat Loops** — The signature technique. Thread-local automations that schedule themselves: draft email replies every 30 minutes, monitor for Slack feedback and re-render video, refresh a customer service page until a human joins then escalate to 1-minute polling. Each is a small control loop with its own cadence and termination condition. The [[Chief of Staff]] pattern (rule-based scanning + LLM classification) is the canonical example, but Liu's Heartbeats are more varied — they cross tool boundaries ($browser to @computer to @slack) and have concrete real-world outcomes (refund processed, video uploaded).

**#concept Durable Threads vs. Durable Memory** — Liu makes an implicit distinction that clarifies the memory architecture debate: the thread carries conversation context (what we're doing right now), the vault carries durable memory (what we learned). Threads compact; memory files persist. This maps to the "context window is RAM, filesystem is disk" model from [[Planning With Files]], but Liu adds a review mechanism (git diffs on the vault) that most implementations miss.

**#tool Side Panel as Work Surface** — The side panel is where "Codex stops being only a chat app." Liu uses it for three things: inspecting artifacts (spreadsheets, PDFs, slides), operating web surfaces ($browser with annotations), and reviewing changes. The key is that the agent controls these surfaces — Liu leaves annotations, the agent acts on them. This is a more integrated vision than standalone tools like [[Browser Use]] or [[surf-cli]].

**#concept The Operating Loop** — Steering + Heartbeats + durable threads combine into what Liu calls "a small operating loop." The agent isn't a tool you invoke; it's a continuous process you shape. When you leave, it doesn't stop — it follows the shape you set. This is the practitioner's version of the agent-as-process model that [[Slate]] formalizes and [[Mission Control — Bhanu's 10-Agent Squad on OpenClaw]] deploys at scale.

**#person Jason Liu** — Consistent voice across this piece and his earlier [[Advice to Young People (Jason Liu)]]. Pragmatic, anti-bullshit, focused on what actually works rather than what sounds impressive. The refund story (agent haggled with customer service while he showered) is classic Liu — concrete, slightly absurd, and more revealing than any abstraction.

---

## Critical Analysis

**The most actionable piece on agent workflow published this year.** Liu isn't speculating — he's documenting a system he's been running for months, with specific cadences (30min, 15min, 5min→1min), specific tools ($browser vs @chrome vs @computer), and specific failure modes (the Slack MCP server couldn't upload files, so he routed around it with @computer). This is what makes it valuable where most agent workflow writing is vague.

**The Obsidian-as-git-repo memory architecture is underrated.** Everyone talks about vector databases and RAG, but Liu's approach — markdown files in a git repo, agent writes to them, human reviews diffs — is simpler, more auditable, and requires zero infrastructure. The key insight is that *git diffs are a review surface for agent memory*. You don't have to trust what the agent learned; you can see exactly what changed. This pattern deserves its own page.

**The Heartbeat taxonomy is incomplete but generative.** Liu's three examples (Chief of Staff, monitor-and-re-render, customer-service escalation) suggest a taxonomy: *polling loops* (check state on a schedule), *reactive loops* (trigger on state change), and *escalation loops* (vary cadence based on proximity to goal). He doesn't name these categories, but the examples imply them. Someone should write that taxonomy.

**The gap: no mention of failure modes.** Liu describes what works but not what breaks. What happens when a Heartbeat goes rogue? When the vault gets corrupted by agent hallucination? When steering instructions conflict? This isn't a criticism of the piece — it's a field report, not a safety manual — but anyone replicating this system needs their own answers. [[Security and Sandboxing]] and [[claude-ctrl]] address parts of this, but the Heartbeat-specific failure modes are unexplored.

**The side panel as work surface is Codex-specific and underdescribed.** Liu lists five web surfaces (index.html, Storybook, Remotion Studio, Slidev, Streamlit) but doesn't explain how the agent interacts with them beyond "$browser with JavaScript." The annotations feature — human leaves comments on agent-controlled surfaces — is gestured at but not detailed. This is the part of the workflow most dependent on Codex's proprietary UI, and therefore the least portable to other agents.

---

*Sources: [[raw/codex-maxxing]]*
*Source URL: https://jxnl.github.io/blog/writing/2026/05/10/codex-maxxing/*
*Last updated: 2026-05-18*
