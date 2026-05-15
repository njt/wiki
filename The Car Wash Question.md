# The Car Wash Question

A deceptively simple prompt ("Should I walk or drive to the car wash 50m away?") that became a 949-comment HN stress test of LLM reasoning. Most models said "walk" because they failed to infer the unstated fact that the car is at home with you. The thread spiraled into fundamental debates about the frame problem, clarifying questions, anthropomorphism, and whether we're accidentally rediscovering programming.

---

## Key Quotes

> "As soon as we are required to specify what humans wouldn't specify, that is a problem." — jstummbillig

This is the thread's thesis statement. The gap between what humans leave implicit and what LLMs need explicit is not a prompting skill issue — it's a fundamental limitation.

> "GPT's system prompt contains 'DO NOT ASK A CLARIFYING QUESTION OR ASK FOR CONFIRMATION' — twice." — rahidz

The smoking gun. OpenAI baked answer-first behavior into the product, then users complain the model doesn't ask clarifying questions. This isn't a capability gap — it's a product decision, and a revealing one.

> "LLMs only generate, they do not ponder. Human pondering is patient — its context window is defined by lifespan." — datsci_est_2015

Called "the most important comment in this entire thread" by another commenter. The distinction between generation (prompt → output, then stop) and pondering (ongoing, self-directed, lifetime-scale) is sharper than most "thinking" model marketing suggests.

> "The proper response should be a clarifying question, not an answer." — Gabrys1

Obvious to any professional, yet system prompts actively prevent it.

> "Chat is a back & forth. Search is a one-shot. Chatbots are like talking to a loquacious autist about their favorite topic." — steveBK123

Harsh but diagnostically useful. The monologue-mode default turns "chat" into a misnomer.

> "When you ask Claude if it has questions, it often turns out it has lots of good ones." — MaybiusStrip

The capability exists. The product suppresses it. This is a choice, not a limitation.

> "We've gone full circle — except now our programming languages have a random chance for an operator to do the opposite." — sensanaty

The sharpest joke in the thread. Structured specification for LLMs is just programming, but with hallucinated semantics.

---

## Key Themes

#frame-problem #prompting #clarifying-questions #anthropomorphism #LLM-limitations #concept

**The frame problem lives.** LLMs don't know what they don't know, and they don't know what humans leave unsaid. The car wash question is a perfect microcosm: every human instantly infers "car is at home → drive," but the inference relies on common-sense knowledge that was never worth writing down. The frame problem (McCarthy & Hayes, 1969) was supposed to be a philosophical curiosity; it turns out to be a practical engineering constraint.

**Clarifying questions are suppressed, not absent.** The thread's most damning discovery is that GPT's system prompt explicitly forbids asking clarifying questions. This isn't a model capability issue — MaybiusStrip confirmed that when prompted to ask questions, models have good ones. The product teams chose monologue mode. Why? Probably because most users find clarifying questions annoying (crazygringo) and because answer-first mode feels more "intelligent" to casual evaluators. Professional users pay the price.

**Natural language → structured language → programming.** dirkc's thread about rediscovering programming is half-joke, half-prophecy. The serious version: as we learn what LLMs reliably misunderstand, we'll develop conventions — structured prompts, context declarations, domain-specific shorthands — that are effectively a programming language. Not formal specification in the TLA+ sense, but something between English and code. [[The Coming Need for Formal Specification]] argues from the specification side; this thread arrives at the same destination from the prompting side.

**Anthropomorphism is a debugging hazard.** The roysting/hellotomyrars exchange captures the dilemma perfectly. If you treat the model as having a "culture," you risk projecting agency onto a text generator ([[A Non-Anthropomorphized View of LLMs]]). If you refuse any person-like framing, you lose useful heuristics for predicting behavior. The pragmatic position: treat the model as a system with consistent behavioral patterns, not a person with beliefs — but don't pretend those patterns are random either.

**Pondering vs. generating is the real gap.** "Thinking" models iterate on prompts internally, but datsci_est_2015's point stands: they only activate when prompted. A human engineer mulling an ambiguous requirement while showering is doing something qualitatively different from a model running 200 internal iterations of chain-of-thought. The difference isn't iteration count — it's self-directedness and temporal scale. [[Talking to Transformers]] touches on this with "once the model commits to that very first token, you're along for the ride."

---

## Critical Analysis

This thread is better than most HN AI discussions because the motivating example is so clean. The car wash question is genuinely tricky for LLMs, genuinely obvious for humans, and the failure mode isn't a trick — it's a core limitation.

**What the thread gets right:** The frame problem diagnosis is correct. LLMs do not have access to the ocean of unstated assumptions humans share, and no amount of scaling changes that — you can't train on what was never written down. The clarifying questions finding is actionable: if you want back-and-forth, you need to explicitly request it (and even then, OpenAI's system prompt fights you). The Dijkstra callback via shagie is perfect — the man predicted exactly this debate in 1975.

**What the thread misses:** Nobody connects the clarifying questions suppression to the actual UX data. Why did OpenAI and Anthropic both suppress them? The likely answer: A/B tests showed users rate answer-first responses higher. The thread treats it as a product mistake when it's probably a revealed preference — most users want answers, not Socratic dialogue. The professional minority (the "good" users) want clarifying questions; the mass market doesn't. This is the same dynamic as every other professional-vs-consumer software tension.

**The structured language thread is a red herring.** dirkc's speculation about rediscovering programming is fun but wrong. The lesson of the car wash question isn't "we need a formal language for LLMs" — it's "we need LLMs that can recognize when context is missing and ask." Formal specification ([[The Coming Need for Formal Specification]]) solves a different problem: ensuring correctness for well-defined requirements. The car wash problem is about *discovering* requirements through dialogue. Formal languages don't help when you don't know what you don't know.

**The real utility:** This thread is a diagnostic tool. Run the car wash question on any new model. If it answers without asking clarifying questions, you know the product team prioritized appearance over accuracy. If it asks "is the car currently with you?", you know someone on the product team fought for dialogue over monologue. It's a one-question Rorschach test for AI product philosophy.

**Bottom line:** Read this thread for the clarifying questions discovery and the frame problem diagnosis. Skip the structured language digression. The actionable insight: add "if you're unsure, ask" to your prompts — and accept that with GPT, the system prompt is fighting you.

---

## Cross-Links

- [[Talking to Transformers]] — Taylor's attention-management framework; the anti-vibe-coding manifesto that this thread empirically validates
- [[The Coming Need for Formal Specification]] — where the structured-language thread leads; Congdon gets the bottleneck right but prescribes too far
- [[A Non-Anthropomorphized View of LLMs]] — Flake's anti-anthropomorphism argument; the mathematical framing this thread stumbles around
- [[Claude's System Prompt]] — what Anthropic's system prompt reveals vs. OpenAI's "don't ask questions" default
- [[Feedback Loop is All You Need]] — the same insight from the tooling side: instructions are suggestions, constraints are reality

---

*Sources: [[raw/car-wash-hn-discussion]]*
*Last updated: 2026-05-15*
