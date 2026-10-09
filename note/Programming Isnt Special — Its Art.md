# Programming Isn't Special — It's Art

Glyph Lefkowitz (Twisted, `Deferred`) argues that the excuse programmers use for embracing AI-generated code — "this is functional work, not art, so it doesn't matter" — rests on a false mystification of art. Programming is art by the same standard as any creative medium, and that means the resistance other creative industries are mounting applies to software too: erasing mundane creative work with AI destroys the practice ground where craft develops.

---

## The Argument

Every other creative field has organised against AI — writers' strikes, artist open letters, copyright lawsuits, YouTubers in open revolt. Programmers, uniquely, "do seem to hate it... but we are using it anyway," resigned on the theory that functional work doesn't deserve protection. Glyph's move is to collapse the dichotomy: programming is Art, and Art includes the mundane.

## Key Quotes

> We do seem to hate it, and it's making us all miserable, and what it's doing to our industry, but we are using it anyway.

An unusually honest sentence. Most pro-AI programmer writing is cheerleading; most anti-AI writing is prophecy. This is neither — it's a description of resignation, and the whole essay is an attempt to give that resentment a principled basis rather than letting it curdle into compliance.

> In John Berger's "Ways of Seeing", he names this type of distortion "mystification".

The Berger frame does real work: a commissioned group portrait was a poor old painter getting paid, not a divine relic. We mystify novelists but denigrate journalists, who deserve no less respect; the novelist/copywriter distinction is "an accident of commerce and opportunity." Applying this to software defuses the standard "but code is functional" rebuttal before it starts.

> `Deferred` is an intentional poem about asynchronous task execution, with a deliberate eye to the aesthetics of the problem and the experience of using it. It was influential *because* of its focus on aesthetics.

The strongest move in the essay is the worked example from the author's own career. `Deferred` — ancestor of jQuery Deferred, Promises, `async`/`await` — was an aesthetic reaction to callback tedium, not a functional necessity. And he honestly locates the tradition as "folk epic poetry": a chain of artisans each adding something until the original dissolves, with credit also owed to E's Promises. That modesty makes the claim credible where grandiose "code as poetry" essayists fail.

> If we use AI to erase all the copy-writing, all the graphic design, all the boring mundane art... then we will be removing all the practical opportunities for the vast amounts of *practice* and *contemplation* required for people to elevate their craft.

This is the actual mechanism, and it's an economic-pipeline argument dressed as an aesthetic one: skill develops on the job, mundane jobs are the training ground, and automating them all converts "definitely sometimes" into "never" for career-defining greatness. Each project's 0.1% doesn't matter; the aggregate does.

> Slop is slop, no matter the medium.

A four-word bridge from the creative-industry discourse into software.

## Key Themes

#concept — mystification (Berger) applied to the art/function dichotomy in software
#concept — mundane creative work as the apprenticeship pipeline for craft
#person — Glyph Lefkowitz, arguing from his own `Deferred` lineage
#pattern — "programming is Art" as the basis for resisting AI, paralleling writers' and artists' organised resistance

## Opinionated Take

The essay's weakness is that it proves too little about *policy*. Establishing that programming is art doesn't tell you which AI uses to resist — Glyph admits as much, punting the "precisely appropriate ways" beyond the margins of the post. And the 0.1% greatness-lottery argument can be turned around: most LLM-assisted work also has some small chance of enabling something great, and human-only projects mostly produce mediocre code that never ships.

But the practice-pipeline point is genuinely the strongest anti-AI argument in the genre, because it's not about the quality of today's output — it's about compounding. It converges with what the industry is discovering empirically: comprehension debt, senior-drift, the shrinking on-ramp. Where [[Understand to Participate]] says understanding is what keeps you a collaborator, and [[Human-in-the-Loop is Tired]] documents the psychological cost of validation-only work, Glyph supplies the missing foundation: *why* the understanding and the practice matter — because they are the only mechanism by which anyone ever became good. The essay also complicates the [[The Joy and Power of Understanding]] line of thought by giving it an aesthetic rather than pragmatic grounding: struggle and craft aren't just how you stay useful, they're the art itself.

What the essay never addresses is the economics that push the other way — when the mundane work is priced at zero, individual resistance is a luxury good. That tension is unresolved here, and honestly in the whole discourse.

## Related

- [[Understand to Participate]] — strengthens: Litt's "understanding as prerequisite for collaboration" gains an aesthetic justification here; both argue that offloading understanding is the real loss, not the code.
- [[Human-in-the-Loop is Tired]] — nuances: Summers diagnoses the misery of validation-only work; Glyph explains why that misery is a symptom of losing the craft itself rather than a workflow bug.
- [[The Joy and Power of Understanding]] — complicates: Roztropiński grounds understanding in pragmatic force-multiplication; Glyph grounds it in art, which is both more and less defensible.
- [[Building When It Feels Like There's Nothing Left to Build]] — shares the question of what human creative work is for when generation is cheap, and lands on a similar "the doing is the point" answer.

---
*Sources: [[raw/programming-isnt-special-html]], [[summary/programming-isnt-special-html]]*
*Last updated: 2026-10-10*
