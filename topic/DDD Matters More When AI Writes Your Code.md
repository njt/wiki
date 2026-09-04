# DDD Matters More When AI Writes Your Code

Miłosz Smółka's case that Domain-Driven Design is *more* relevant in the age of AI coding, not less — because DDD was never about the code. AI automates implementation, which was always the easy part; the hard part remains understanding the problem domain, modeling it well, and doing so as a team. The essay is a DDD-grounded counterweight to the "just generate a spec" enthusiasm: the value of the model is in what the team understands, not in what artifact an agent produces. #concept #pattern #person

---

## The Unchanged Hard Part

Smółka's starting point is Eric Evans's 2003 *Domain-Driven Design*, and his claim is that the book "is still surprisingly fresh."

> "The hard part of software engineering is understanding the problem domain and modeling it well in code."

The implementation details have become cheaper than ever — you can be productive without deep knowledge of a language or framework, and top coding agents generate decent code quickly. But that only reallocates attention, it doesn't change where the difficulty lives. Smółka's update of Evans is pointed: instead of obsessing over frameworks, teams now "follow the model benchmarks and optimize our agentic setup to generate *better code* with less effort" — and "funnily enough, this time the CEOs also believe it." We are still trying to solve domain problems with technology, just with newer technology.

## The Domain Model Isn't an Artifact

This is the essay's sharpest move, and the one that cuts against the current of [[Specifications as the Product]]. DDD's knowledge crunching — domain experts and engineers working out together what the software should do — is tempting to outsource, because "AI agents are brilliant at research and analysis" and could produce the model *and* implement it. Smółka calls this "a naive approach":

> "The value of working on the model is that you (and your team) understand how the domain works. An AI-generated wall of text doesn't help you figure out the business problem. The agent will just create an impressive artifact no one reads."

This is the precise inverse of the spec-as-durable-artifact thesis: the artifact is not the point, the shared understanding is. Projects fail from engineers working on the wrong thing, from no one knowing what's needed, and from scope creep — none of which a longer document fixes. It's a direct challenge to the [[AI Agents Need Clear Specs|spec-driven]] consensus that the spec *is* the product: a spec nobody read is still nobody understanding anything.

## Design Before Generating Code

Smółka connects the DDD argument to the code-review bottleneck that agents have worsened. A single-shot agent produces PRs "like working with a lone-wolf developer who prepares complete features and drops a massive PR on you." The fix is upstream of code:

> "Code review should be a double-check that the implementation is correct, not the start of a discussion about whether the approach makes sense at all."

The prescription is team design before implementation — domain parts first, then technical — with no need for a complete spec: "Some sticky notes and a whiteboard are good enough." This echoes [[Domain Storytelling]]'s premise that shared visualization *is* the requirements work, and lands the same point [[Understand to Participate]] makes: you can only guide an agent if you've already thought the problem through.

## Ubiquitous Language Is Prompt Engineering

DDD's language discipline turns out to be prompting discipline. Smółka's anecdote is the best evidence in the piece: he asked an agent to cut memory usage, it saved "many megabytes of RAM" for $1/month, because "I didn't explain to the agent that I was trying to cut costs. I just gave it the memory usage as the target." Soft skills, he deadpans, turn out to help you talk to computers.

> "You'll make the goal more obvious by sticking to the precise names."

The concrete contrast — vague "Add user to CRM and support after it's created" versus precise "Once the user signs up on the website, asynchronously create: 1) a customer entry in the CRM, 2) a profile in the support system" — is Ubiquitous Language restated as prompt craft. [[Bounded Context|Bounded contexts]] get their own AI-specific warning: agents see your whole repository and "may naively try to unify similar entities," so you must make clear they are separate for a reason. The naming discipline [[Prefix Effects]] documents — names as alignment surfaces that shape generated code — is here argued from the DDD side.

## Don't Delegate Thinking

The closing section is the most personal, and the most important. Smółka recalls being paid to think at his first job and feeling "wild," then notes the reversal: today "we focus more on the output. Why not have the AI do the thinking instead?"

> "The only reason I can judge whether the agent's output makes sense is that I've spent long hours thinking about such problems in the past."

His book-versus-summary analogy is the cleanest statement of the fluency argument that runs through [[Understand to Participate]] and [[The Joy and Power of Understanding]]: reading forces you to build a complex model in your mind; a summary is "a few sentences that you forget ten seconds later when you close the browser tab." You can't build new mental models without thinking, and you can't guide an agent you can't out-think.

## Critical Analysis

**What lands.** The essay's strength is its economy: it takes one idea — DDD was never about code — and traces it through every layer of the AI-coding stack, from prompting to code review to career advice. The cloud-cost anecdote is the kind of lived, mildly embarrassing example that no survey or benchmark can produce, and it does more work than the theory sections.

**The gap: bounded context depth.** Smółka gestures at bounded contexts and says agents will "naively try to unify similar entities," but doesn't develop how a team actually *encodes* context boundaries so an agent respects them. That's the operational question — do you put the bounded-context map in markdown near the code? in the agent's system prompt? in directory structure? — and the essay stops right before answering it.

**Where it's genuinely in tension with the spec consensus.** The essay is a one-source counterweight to [[Specifications as the Product]], not a refutation. The two positions are reconcilable — a spec is only durable if a team actually internalized the design decisions it encodes — but Smółka's emphasis on *understanding* over *artifact* is the missing human term that the spec-as-product literature tends to bracket. The reconciliation is the interesting synthesis nobody has written.

**The career advice is the most vulnerable claim.** "Learning to work with an unknown domain pays off more than focusing on technical skills alone" is plausible but unproven, and it's the same value-differential bet [[Code-First Developer]] makes — with the same caveat that nobody knows how to *train* domain fluency at speed when AI is doing the implementation.

## Related Pages

- [[Domain Storytelling]] — The collaborative modeling method whose "AI angle" Smółka's essay directly develops: domain stories feeding agents, and why understanding beats artifacts
- [[Specifications as the Product]] — The thesis this essay complicates: the domain model isn't the artifact, it's the understanding
- [[Understand to Participate]] — Litt's fluency argument, restated from the DDD side: you must understand the domain to guide the agent, not rubber-stamp it
- [[Code-First Developer]] — Stemmler's value differential and design-over-code, grounded in DDD's modeling discipline
- [[Vertical Slice Architecture]] — The code-organization cousin: DDD's bounded contexts become feature slices

---
*Sources: [[raw/ddd-and-ai-coding]], [[summary/ddd-and-ai-coding]]*
*Last updated: 2026-09-04*
