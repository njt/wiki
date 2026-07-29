# Notes on Structured Programming

Dijkstra's 1970 monograph that launched structured programming — the idea that programs should be composed from a restricted set of control structures (concatenation, selection, repetition) and built through step-wise refinement. More than a methodology: it's a philosophical treatise on the cognitive limits of programmers, the impossibility of testing as proof, and the case that programming is fundamentally the art of organizing complexity. Every argument about agentic code quality, verification versus testing, and the limits of human attention traces back to this document.

---

## Key Quotes

> "Program testing can be used to show the presence of bugs, but never to show their absence!"

The single most quoted line in software engineering — and one that's been swallowed, digested, and forgotten by every generation that follows. The current wave of agentic coding has revived it with a vengeance: when code is generated faster than it can be reviewed, testing alone is demonstrably insufficient. Dijkstra's corollary is that you need the program's *structure* to reason about its correctness — which is exactly what structured programming provides, and exactly what LLM-generated code most obviously lacks.

> "Any two things that differ in some respect by a factor of already a hundred or more, are utterly incomparable."

Dijkstra opens by naming the central problem he can't solve: he wants to teach techniques for "life-size programs" but can only use small examples. The scale gap *is* the problem. This is the same gap that bedevils coding agent discourse today — demos work beautifully on toy problems, and everyone extrapolates to production codebases. Dijkstra's one-year-old-crawling-child versus supersonic jet analogy is the original warning against the fallacy that still powers every AI coding demo.

> "The art of programming is the art of organizing complexity, of mastering multitude and avoiding its bastard chaos as effectively as possible."

Dijkstra explicitly subordinates efficiency to manageability. This was controversial in 1970 and it's still the central tension in agentic development: the machine can produce infinite code, but human understanding remains the bottleneck. His argument that efficiency optimizations should only happen *after* logical correctness is established — and only if the program remains "sufficiently manageable" — is the ur-text of "make it correct, then make it fast."

> "I have a very small head and I had better learn to live with it and to respect my limitations and give them full credit, rather than to try to ignore them, for the latter vain effort will be punished by failure."

The entire enterprise of structured programming follows from this admission. Dijkstra treats cognitive limits not as a weakness to overcome but as the *design constraint* for programming methodology. This is [[Engineering for Bounded Cognition]] forty years before the term existed. The same admission explains why code review doesn't scale with AI-generated volume, why context windows aren't a substitute for structure, and why [[Agent Memory and Context]] is now the hardest problem in the field.

> "Our intellectual powers are rather geared to master static relations and our powers to visualize processes evolving in time are relatively poorly developed."

The justification for making program text *reflect* computation structure. When program progress through text maps cleanly to computation progress through time, the static text acts as a prosthetic for the dynamic process we can't hold in our heads. This is the theoretical basis for structured programming's single-entry/single-exit discipline — and it's what every coding agent framework is stumbling toward with checkpointing, state machines, and event-sourced architectures.

> "I want to view the main program as executed by its own, dedicated machine, equipped with the adequate instruction repertoire operating on the adequate variables and sequenced under control of its own instruction counter."

The layered virtual machine model — each level of abstraction is a complete machine implemented by the level below. This is the intellectual ancestor of: API design, microservices, Docker containers, MCP servers, and the entire agent abstraction stack. Dijkstra got there first, from first principles, in 1969.

## Key Themes

- #concept **Scale incomparability**: A factor of 100× makes things "utterly incomparable." Small programs and large programs are different *kinds* of things, not different sizes of the same thing. The central insight that drives structured programming, and the insight most consistently violated by extrapolation from AI demos.
- #concept **Correctness via structure**: Testing can only show bugs, never prove absence. Proof requires reasoning about internal structure, and structure must be *designed* to enable that reasoning. The three control structures (concatenation, selection, repetition) each map to a specific pattern of reasoning (enumeration, case analysis, mathematical induction).
- #pattern **Step-wise refinement**: Build programs by successive decomposition, deciding as little as possible at each step. Each level introduces exactly one design decision. The "pearl" (one independent decision) strung into a "necklace" (the complete program) is the metaphor — and it anticipates everything from microservice decomposition to skill-based agent architectures.
- #concept **Program families**: Don't design a program; design a family of related programs that share a common ancestor at some level of abstraction. Modification should be "pearl replacement," not text editing. This is the intellectual ancestor of: version control, refactoring, interface contracts, dependency injection, and the entire "design for change" tradition.
- #pattern **Layered virtual machines**: Each abstraction level is a complete machine with its own instruction set, state space, and program — implemented by the level below. The manual for each machine is the interface contract between levels. This is the cleanest formulation of abstraction in computing ever written.
- #tool **Mathematical induction for loops**: The only pattern of reasoning that can cope with repetition. Without it, you're limited to what you can enumerate — which Dijkstra correctly identifies as far too little for real programs.
- #concept **Abstraction as the primary mental tool**: "Our main mental technique to reduce the demands made upon enumerative reasoning." Naming an operation and using it for *what* it does while ignoring *how* it works is the programmer's essential cognitive strategy — and exactly what functions, APIs, and MCP tools provide.
- #person **Edsger W. Dijkstra** (1930–2002): Dutch computer scientist, Turing Award winner (1972), and perhaps the field's most penetrating philosophical mind. His EWD manuscripts are a genre unto themselves — handwritten, numbered, distributed to colleagues by mail, and unfailingly sharp. EWD249 is the one that changed how the world thinks about programs.

## Critical Analysis

**What holds up — and what's more relevant than ever.** The core arguments are unassailable and, if anything, more urgent now than in 1970. The agentic coding explosion has recreated exactly the conditions Dijkstra diagnosed: programs generated faster than anyone can understand them, correctness established by "it seems to work," and cognitive limits treated as an inconvenience rather than a design constraint. Every framework that adds structure to agent output — skills, harnesses, verification gates, checkpointing — is rediscovering structured programming at a higher level of abstraction.

**The missing piece: concurrency.** Dijkstra explicitly restricts himself to sequential programs. His later work (EWD310, "Cooperating Sequential Processes") addressed concurrency, but the structured programming model as presented here is purely sequential. The agents we build today are inherently concurrent — multiple models, multiple tools, multiple sessions — and the structured programming model doesn't directly apply. Yet the *spirit* of it — find the right primitives, restrict yourself to them, reason about the whole by reasoning about the parts — is exactly what agent architecture is groping toward.

**The pearl model predicts microservices.** Reading "On what we have achieved" in 2026 is uncanny. The pearl as an independent design decision, the necklace as a composition of pearls, program modification as pearl replacement — this is a precise description of what microservice architecture aspires to be and often fails to achieve. The observation that ordering pearls differently produces "messier" programs (more tightly interconnected logical threads) is a formal argument for what we now call "dependency inversion."

**The weakest section is the strongest warning.** The line-printer plotter example (the "second example of step-wise program composition") is tedious to read and the notation is half-baked. Dijkstra admits as much. But that's the point: he was inventing the notation as he went, and the messiness of the notation *is* the evidence that programming language design matters. The fact that we're still struggling with the same problem — how to express structure in a way that survives modification — suggests we haven't come as far as we think.

**What Dijkstra couldn't see: the compiler as the proof.** His correctness proofs are manual, laborious, and he admits they're impractical at scale. He couldn't have foreseen type systems, model checkers, or property-based testing. But his intuition was right: the structure that makes programs understandable to humans is also the structure that makes them verifiable by machines. The type checker is the automated version of Dijkstra's enumerative reasoning; the QuickCheck property is his mathematical induction, automated. For the formal logic that underlies these reasoning patterns, see [[An Introduction to Formal Logic (Peter Smith)]], which covers the same logical territory Dijkstra assumed his readers knew.

**The irony nobody talks about.** Dijkstra's most famous aphorism — "Go To Statement Considered Harmful" — isn't in this document. It's in EWD215 (1968). But this document is where he makes the *positive* case for what should replace GOTO: the three control structures and step-wise refinement. The negative case got the fame; the positive case is the actual contribution. Structured programming was never really about *removing* GOTO — it was about building programs from primitives that correspond to patterns of reasoning.

---

*Sources: [[raw/ewd249-notes-on-structured-programming]]*
*Last updated: 2026-07-18*
