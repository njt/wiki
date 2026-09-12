# IO Factory

IO Factory is an arXiv framework for simulating AI-enabled influence campaigns as traceable lifecycles inside a controlled social-platform simulation. It splits a campaign into three actor categories (civilians, IO operators, a coordinating manager), routes content through a ten-phase lifecycle, and measures influence as "directional lift" — civilian belief-state movement in an active run versus a matched baseline with no operators. Its real contribution is epistemological, not empirical: it makes campaign assumptions, exposure paths, and measurement rules *explicit and auditable*, and refuses to let a simulator result masquerade as a real-world persuasion estimate.

---

## Key Quotes

> "The visible pieces of a campaign may look ordinary: a plausible post, a familiar account, or a real person repeating a message... the relevant object is collective campaign behavior across accounts, time, and exposure paths."

The paper's central move — stop looking at messages, start looking at *processes*. This is the same inversion Aphyr makes about information ecology: the harm isn't in any one artifact but in the coordination invisible from any single vantage point.

> "A post has no direct effect merely because it exists. It must become visible to a civilian and then be recorded as an exposure before it can enter the measurement pipeline."

The discipline that the rest of the field skips. Most "AI influence" discourse conflates message *volume* with *effect*. IO Factory enforces a chain — created → visible → encountered → measured → updated — and records each stage separately so campaign volume can't be confused with exposure or movement.

> "The manager... issues bounded directives through private coordination channels. These directives affect public behavior only through later IO-operator actions."

A quiet architectural admission that coordination is the thing being studied. The manager is the "AI swarm" made legible: a non-public controller whose influence must pass through public actors before it can touch platform state.

> "These readings are simulator measurements, not ground truth. They parameterize the update rule; the update itself is deterministic given the exposure record, judge output, civilian state, and experiment configuration."

The authors treat LLM judges as *measurement instruments, not oracles* — a distinction that runs through the whole paper. The LLM is used for narrative design, action selection, content generation, and judging, but every place it matters is wrapped in a deterministic rule or a recorded provenance link. The intelligence is bracketed; the bookkeeping is not.

> "All three directional-lift estimates are significant after Holm correction."

The empirical result is real but modest — lifts of 0.13–0.34 on simulator scales — and the paper bends over backwards to say it doesn't establish real-world persuasion. This is either admirable honesty or the paper pre-emptively disclaiming the thing a red-teaming tool is ultimately *for*.

---

## Key Themes

- **#concept AI swarms** — Persistent, coordinated LLM-agent groups with stable identities acting toward a shared goal. The paper's core threat model: campaign behavior that looks dispersed at the account level while remaining coordinated at the system level.

- **#concept Directional lift** — Influence as *relative* movement (active minus matched baseline), not absolute persuasion. The comparison protocol is what makes the simulation interpretable rather than theatrical.

- **#concept The campaign lifecycle as unit of analysis** — Ten phases, grounded in DISARM/RICHDATA/MITRE kill-chain lineage, gate what can happen and what's recorded. Phase state is part of the simulation record.

- **#pattern Deterministic update rule around an LLM judge** — The LLM supplies readings (relevance, stance, confidence, persuasiveness); a configured, deterministic rule turns them into state change. Intelligence is quarantined behind a deterministic gate.

- **#pattern Separation of environment from campaign model** — The simulated platform (built on OASIS) is kept distinct from the study models layered on top, so researchers can vary platform assumptions while holding measurement fixed, or vice versa.

- **#tool Red-teaming infrastructure** — IO Factory is positioned as a cyber-range for influence: reproduce, red-team, and benchmark coordinated-influence scenarios before they appear at scale on real platforms.

---

## Critical Analysis

**The framework is more interesting than its results.** The reported numbers — directional lifts of 0.13–0.34 — are the least important thing in the paper, and the authors seem to know it. The contribution is the *audit trail*: a chain from action to exposure to judge reading to deterministic state update to matched comparison, all preserved. That's a genuine answer to a real gap. Influence-campaign research has been starved of shared, reproducible apparatus; IO Factory is a reference implementation of one.

**But the honesty cuts both ways.** The paper says "simulator-scale outcomes under declared assumptions, not real-world persuasion" so many times it starts to read as a shield. If the civilians are LLM-driven, the platform is synthetic, the judges are LLMs, and the constructs are configured — then what exactly has been demonstrated? The authors would answer: *that campaign lifecycles can be represented, executed, and compared traceably at scale*. True and useful. But the distance between a modeled civilian's construct value and a real voter's attitude is where all the actual risk lives, and IO Factory deliberately does not cross it. Calibration against real observation is waved at as "future work."

**The strongest idea is the one the paper underplays: the manager.** The manager — a non-public controller issuing bounded directives that only reach the platform through public operators — is the closest thing we have to a formalization of what "coordinated but dispersed" actually means. The whole AI-swarms threat collapses into that single box. Future work should push hardest there: what coordination topologies, what obfuscation strategies, what detection signatures correspond to manager-to-operator structure. That's the part a defender would most want red-teamed.

**There's a reflexivity trap hiding in the methodology.** The same LLM family generates the campaign content *and* judges its persuasiveness *and* simulates the civilian that responds to it. The paper handles this by fixing judge settings across matched conditions — which supports *internal* comparison but says nothing about whether the judge is right. The authors admit this explicitly ("LLM judges are not ground truth"). It's a known limitation, well-handled, but it means a coordinated campaign and its simulated victim are, in a sense, the same model talking to itself.

**Read alongside SimPolitics, this is the same computational imaginary with the goal inverted.** McKelvey documents sixty years of people modeling politics to *predict and optimize* it. IO Factory models influence campaigns to *defend against* them. That reframing — from prediction to red-teaming — is quietly the most important thing about the paper, and it's the answer to McKelvey's "are other computer simulations possible?"

---

## Connections

- [[SimPolitics]] — IO Factory is a live instantiation of McKelvey's SimWorlds tradition, but the goal is inverted: not predicting political outcomes, but red-teaming the *manipulation* of them. It is one concrete answer to McKelvey's closing question of whether other simulations are possible.

- [[The Future of Everything is Lies I Guess]] — Aphyr theorizes information-ecology collapse and industrialized state propaganda; IO Factory is the defensive apparatus for studying exactly that collapse as a *process* rather than a pile of artifacts. Where Aphyr diagnoses, IO Factory builds a lab bench.

- [[AI Livestream Factories]] — The real-world dark factory for attention. IO Factory is its simulation twin: the same AI-at-scale persuasion logic, but instrumented and replayed under controlled conditions so it can be measured rather than just observed.

- [[Seeing Is Not Believing — AI Video and Perceptual Safety]] — Shares IO Factory's core premise that *content detection is not enough*. Where that paper shows disclosure doesn't stop perceptual erosion, IO Factory shows single-message detection can't see coordinated campaigns — both argue the relevant signal is behavioral, temporal, and systemic.

- [[1000 Players Simulate Civilization]] — Social simulation as a way of seeing emergent behavior. IO Factory is the same instinct — set initial conditions, observe emergence — applied not to inequality but to engineered consensus.

---

*Sources: [[raw/2608-10920v1]], [[summary/2608-10920v1]]*
*Last updated: 2026-08-14*
