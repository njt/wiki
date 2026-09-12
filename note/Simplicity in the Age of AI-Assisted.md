# Simplicity in the Age of AI-Assisted

The argument that LLMs don't change what good programming looks like, but they change our relationship to simplicity. The real unlock isn't faster generation -- it's the ability to cheaply rebuild systems without inherited complexity. When humans identify unnecessary complexity, LLMs make acting on that judgment nearly free.

---

## Key Quotes

> "The hard part is the human part: recognizing which complexity is essential and which is inherited."

> "The layers exist for communication, not computation."

> "The complexity justifies the roles. The roles produce the complexity."

> "The LLM will not question your architecture."

> "What LLMs actually give you is the possibility of simplicity: the removal of complexity that shouldn't have been there."

## Key Themes

#simplicity #cognitive-debt #agentic-coding

The argument unfolds in four moves:

1. **Complexity through linguistic layers** -- a single feature (sticky header toggle) gets expressed across user stories, CSS, JS, config files. Each layer serves human communication, not computation.

2. **Complexity explosion** -- add persistence and the feature balloons into databases, migrations, APIs, Kubernetes, monitoring, specialized roles. "All of this to remember whether the header should be sticky."

3. **Self-perpetuating cycles** -- infrastructure creates roles that justify the infrastructure. "The complexity justifies the roles. The roles produce the complexity."

4. **LLMs inherit, not question** -- models reproduce existing patterns because they were trained on millions of repos that perpetuate those patterns. They won't tell you your architecture is unnecessarily complex.

The punchline: the value of LLMs isn't that they generate code faster. It's that they make it cheap to **throw code away and rebuild simpler**. Once a human makes the call that complexity is inherited rather than essential, the LLM can regenerate a simpler version in minutes instead of weeks of careful refactoring.

This is the philosophical foundation for several other pieces: [[Write Only Code]] (if code is disposable, simplicity is the only durable value), [[The Claude C Compiler]] (Lattner's point about investing in structure), [[Prefix Effects]] (if early patterns solidify, start simple), and [[Elements of Code]] (comprehensibility as the primary virtue). [[Grug Brain Developer]] got there earlier and funnier: "complexity is spirit demon that enter codebase through well-meaning but ultimately very clubbable non grug-brain developers." Same diagnosis, different vocabulary — grug's "spirit demon" is this article's "inherited complexity."

## Critical Analysis

This is one of the sharpest pieces in the batch. The four-move argument is tight, the quotes are memorable, and the conclusion is both counterintuitive and correct: LLMs are more valuable as demolition tools than construction tools.

The limitation: the author doesn't address when inherited complexity is load-bearing. Not all organizational infrastructure is waste -- sometimes the Kubernetes cluster exists because you actually need multi-region deployment, not because a DevOps team justified its existence. The hard judgment is distinguishing load-bearing from vestigial, and the article acknowledges this is the hard part without offering a method for making the call.

---
*Sources: [[summary/simplicity-in-the-age-of-ai-assisted]]*
*Last updated: 2026-05-14*