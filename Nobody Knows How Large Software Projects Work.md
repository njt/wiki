# Nobody Knows How Large Software Projects Work

Sean Goedecke's observation that large software systems exceed anyone's mental model -- and that this isn't a bug but an inherent property of scale. The value of an engineering team isn't just writing code; it's the ability to answer questions about the system. And for large systems, answering questions requires investigation, not recall.

---

## Key Quotes

> "Simple questions like 'can users of type Y access feature X?' often can only be answered by a handful of people in the organization."

> "It's easier to write software than to explain it."

> "The ability to answer questions about software is one of the core functions of an engineering team."

## Key Themes

#cognitive-debt #observability #management

The complexity comes from value-generating features that interact with everything: self-hosting options, organizational controls, localization, regulatory compliance. Each "affect every single other feature you build," creating exponential complexity. The system becomes a swamp not because of poor engineering but because of successful product development.

Documentation fails because the system changes faster than anyone can write it down, and many behaviors don't have conscious intent behind them -- they just emerged from accumulated choices. This is a different failure mode than "we didn't document it." It's "the system is undocumentable at the speed it changes."

This reframes the value of [[Before Reading Code]] -- git archaeology as a substitute for institutional knowledge that doesn't exist. And it strengthens the case for [[Spec-Driven Development]] and [[OpenSpec]] -- if the system is unknowable through inspection, specs that stay synchronized with code are the only way to maintain any understanding at all.

The AI angle: if nobody understands the system today, and AI generates code 10x faster, the unknowability gets worse. [[Simplicity in the Age of AI-Assisted]] addresses this directly -- the solution is rebuilding simpler, not documenting harder.

## Critical Analysis

Goedecke names a truth that engineering organizations avoid confronting. The standard response to "nobody understands the system" is "we need better documentation" or "we need more senior engineers." His point is subtler: the incomprehensibility is structural, not a staffing or process failure.

The one thing missing: what to do about it beyond accepting it. Goedecke identifies the problem brilliantly but offers limited mitigation. The answer lies scattered across other pieces in this wiki -- observability ([[The Future of Software Engineering is SRE]]), spec-first development ([[Spec-Driven Development]]), and the radical simplification that [[Simplicity in the Age of AI-Assisted]] advocates.

---
*Sources: [[raw/nobody-knows-how-large-software-projects-work]]*
*Last updated: 2026-05-14*