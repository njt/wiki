# Weird Machines in Transport Layer Security

An arXiv paper (submitted 13 Aug 2026) extending "weird machine" theory — the idea that latent computation emerges from the composition of architectural components — to the TLS handshake and its two dominant implementations, OpenSSL and BoringSSL. The authors argue that legitimate TLS primitives compose into Turing-complete systems whose computation is coupled to authentication and trust decisions, a coupling they name **trust actuation**, and demonstrate it with two working demos — a defensive sentinel and an authentication bypass — built on real OpenSSL code paths with no memory corruption or malware.

---

## Key Quotes

> "Weird machines are latent computational capabilities that emerge from the composition of architectural components."

The one-sentence definition of the whole field. "Latent" is the load-bearing word: the computation was never designed in; it is a side effect of how components interact.

> "legitimate TLS primitives, including session cache entries, renegotiation logic, extension parsing, and certificate verification steps, compose into Turing-complete systems whose computation is coupled to authentication and trust decisions rather than physical actuation."

The domain jump. Prior weird-machine work coupled computation to physical actuation — industrial-control systems where a wrong signal moves a valve. TLS couples it to *trust*, which is cheaper to reach than a physical actuator and more valuable than a crash: a wrong "yes" is the entire ballgame.

> "We formalize this coupling, which we call trust actuation."

The named contribution. Trust actuation reframes the TLS handshake from a fixed procedure into a programmable substrate whose outputs are trust decisions.

> "any TLS implementation providing session storage, arithmetic on sequence counters, conditional branching on handshake state, and iteration through resumption or retry loops satisfies the conditions for arbitrary computation."

The sufficiency claim. It is disarmingly general — those four features describe most stateful protocols, not just TLS. The claim is as much a warning about statefulness as it is about TLS specifically.

> "The first, a sentinel system, composes standard TLS primitives into a defensive mechanism that detects anomalous handshake behavior. The second, an authentication bypass, composes the same class of primitives into an attack that defeats a cipher-strength policy check through mid-connection renegotiation, without any memory corruption or external malware."

The rhetorical move that matters most: the same primitives, two opposite intents. The machine is value-neutral — a lever that defense and attack both get to pull.

## Key Themes

- **#concept** — weird machines / latent computation: capabilities that emerge from composition rather than design
- **#concept** — trust actuation: computation coupled to authentication and trust decisions instead of physical effects
- **#tool** — TLS, OpenSSL, BoringSSL: the substrate under study
- **#pattern** — Turing-completeness from composition: four innocuous features add up to arbitrary computation
- **#security** — offense/defense duality: one machine, a sentinel and a bypass

## Critical Analysis

**The inversion worth keeping.** The paper's real contribution is inverting how we read the TLS handshake: not as a fixed sequence of steps but as a machine you can program with the protocol's own features. Every feature — session resumption, renegotiation, extensions — becomes an instruction. That reframes security review from "does this step check the right thing" to "what computation does the composition of these steps make possible," which is a harder and more honest question. It rhymes with the insight in [[Security and Sandboxing]] that trusted components contain capabilities nobody designed in.

**Trust actuation is the high-value generalization.** Weird-machine work began in memory corruption (x86), then moved to physical actuation (ICS). Coupling computation to *trust* is the logical endpoint: a trust decision is cheaper to exploit than a physical one and more valuable than a crash — no actuator, no payload, just a wrong "yes."

**The dual demo is the strongest evidence and the weakest.** Building both a sentinel and a bypass from the *same* primitives is rhetorically devastating — it proves the machine doesn't care about intent. But as ingested this is an abstract only: no proof, no code, no review. "Turing-complete" claims need the actual construction to be checked, and "defeats a cipher-strength policy check through mid-connection renegotiation" sits uncomfortably close to TLS renegotiation's documented history of weaknesses — the 2009 renegotiation attack was exactly a legitimate-feature-abuse bypass. The honest question is whether this is a novel class or a sharp re-derivation of a known failure mode.

**Why it belongs in an AI-heavy wiki.** The concept travels. An agent chaining legitimate tool calls into an unauthorized outcome — the Least Agency problem in [[Zero Trust for AI Agents]] — is trust actuation in another form: no exploit, no malware, just composition. And [[Common Expression Language (CEL)]] is the deliberate opposite: a language engineered to be non-Turing-complete precisely because its designers knew unconstrained composition smuggles in arbitrary computation. This paper is the accidental version of what CEL refuses on purpose.

**The sentinel is the least developed idea, and possibly the most useful.** Detecting anomalous handshake behavior by composing TLS primitives is a genuinely different detection target from the event-count approach in [[Anomaly Detection]] — it watches protocol *state transitions*, not volumes. But the abstract gives it one sentence, and "detects anomalous handshake behavior" is doing a lot of unstated work about what "anomalous" means when the attacker is, by construction, using only legitimate primitives.

---

*Sources: [[raw/2608-13685]], [[summary/2608-13685]]*
*Last updated: 2026-08-21*
