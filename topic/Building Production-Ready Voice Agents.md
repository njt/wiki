# Building Production-Ready Voice Agents

Shekhar Gulati's field report from building a production voice agent for university IT help desks. The headline number: 50% of development effort goes into the admin portal, not the voice agent itself. Observability, replay, configuration management, and debugging interfaces are the real product.

---

## Key Quotes

> "The demo shows the happy path -- a cooperative user, clear audio, no edge cases. Production shows you everything else."

> "Roughly 50% of our development effort goes into the admin portal -- not the voice agent itself."

> "Silence is death."

> "A fast agent that takes users in circles is worse than a slightly slower one that resolves their issue."

## Key Themes

#sre #observability #agent-architecture #error-handling

Eleven battle-tested lessons from a three-developer team:

**State machines for conversation management** -- model conversations as graphs where each node defines role messages, task messages, and available functions. Prevents "context pollution" across states. This is the voice equivalent of the pipeline thinking in [[The Dark Factory is a DOT File]].

**Avoid drag-and-drop flow builders initially** -- code-based flows are testable; visual builders break down with real complexity. Understand patterns first, abstract later.

**Admin portal as the real product** -- turn-level conversation analysis with timestamps, transcripts, function calls, latency breakdowns. Turn-level replay directly from the portal. This is [[The Future of Software Engineering is SRE]] made concrete.

**Latency is unforgiving** -- users hang up 40% more with responses exceeding one second. A typical turn involves VAD (100-200ms), STT (100-300ms), LLM inference (300-600ms), TTS (100-200ms), and network transmission. P95 at 1.5-2.5 seconds.

**Function calling as highest failure point** -- failures appear as hallucinations. Validate in code handlers, not the LLM. This connects to the guardrails theme across [[Spec-Driven Development]] and [[Write Only Code]].

**Voice-specific prompt engineering** -- cap responses at 30-50 words. Users can't scroll back. NATO phonetic alphabet for alphanumeric data. Normalize user input aggressively.

## Critical Analysis

This is one of the most practically useful pieces in this entire batch. It's not theory; it's a field manual from someone who shipped a voice agent with three developers. The specificity (exact latency numbers, exact team size, exact tech stack) makes it credible.

The 50% admin portal number is the headline, but the deeper insight is that voice agents are distributed systems problems with real-time constraints. Every lesson -- timeouts, circuit breakers, graceful degradation, idempotent operations -- is distributed systems orthodoxy applied to a new domain. Voice just makes the consequences of failure immediate and visceral: the user hangs up.

Missing: cost analysis and scaling patterns. How does this architecture hold up at 100x the call volume?

---
*Sources: [[summary/building-production-ready-voice-agents]]*
*Last updated: 2026-05-14*