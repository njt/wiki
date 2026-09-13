# Intelligence as Infrastructure — Igor Sevo (Craft 2025)

Igor Sevo — computer scientist, PhD in machine learning, Head of AI at HTEC Group — delivers a talk that starts at chatbots and ends in a theory of intelligence, held together by one provocation delivered almost in passing: your company is already an agentic runtime, and most of its code currently executes on the slow interpreter — humans. Between thesis and provocation sits a dense practitioner layer: routing-agent RAG without vector databases, process automation as reductionist decomposition, prompts deployed on their own CI/CD pipeline, and a dev loop where you refactor every 3–5 steps because AI-generated code rots faster than human code.

---

## What the talk covers

- **Chatbots are commodity; proactive agents are the direction.** Agents that initiate contact, act unprompted, and render UI as ad-hoc HTML while you converse. Agents exchange "negotiation messages" to agree on interface format before working — "pretty much replacing a front end engineer in some sense."
- **RAG without ML vernacular.** A "B-tree inspired" routing-agent tree: routing agent → category agents → leaf agents, and only the leaves touch a database. Fast to prototype, proven at POC stage, no embeddings above the leaves.
- **Process automation as reductionist decomposition.** Interview the people doing the work, replicate each step with an individual agent, show a UI only where replication fails; over time agents absorb the UI. Partial automation pays: filter 70% of files, save 70% of the work — with a human auditor acting as parser for the rest.
- **Personal tooling.** An extended-reasoning agent: fan out a main prompt plus a revision prompt across models, majority-vote, metaprompts auto-generate the revision prompts ("Because developers are lazy, and I was lazy here"). Four hours to build with Copilot and ChatGPT.
- **Prompts as code, deployed separately.** One repo, two CI/CD flows: code changes redeploy the app; prompt-folder changes push to a prompt store (Azure Blob) with no redeployment. Templating (Mustache, Jinja) with composition and inheritance enables later ethics/bias review. Same codebase pulls different prompt repositories and models per region — "practical ethics."
- **The AI-era dev loop.** Generate → inspect & verify → clean & refactor → write & revise → test & document. Refactor every 3–5 steps, commit much more often, vibe coding only for prototypes: "you cannot be a poor engineer" to use AI well.
- **Automated onboarding.** If trends hold, "junior developers will never enter the workforce" — so HTEC deploys chatbots that test learners, automated tech leads, and a "hand-holding tech lead" for juniors.
- **Protocols and the frontier.** MCP and A2A as the substrate that lets exposed tools "merge into more and more complex agents." HTEC R&D fine-tunes for behavior, not conversation — XAML operations to shared object storage — and prototyped an "agentic runtime": a simulated OS running text on two virtual machines (a deterministic interpreter and an LLM), interrupt handlers written in English, humans treated as function endpoints. Not viable: ~10x the cost of a person.
- **Four claims about intelligence.** Intelligent systems are non-modular, information-compressing, necessarily agentic, and general — "intelligence is a measure of generality." Corollary: benchmarks measure only "the subset of its abilities that overlaps with ours."

## Key quotes

> "Your current company is already built in the agentic runtime. Just most of the code is run by humans on that part of the interpreter."

The thesis, dropped in passing during a slide about A/B testing agents against humans. It reframes AI adoption as already complete in structure and merely shifting in ratio: every process is instructions in a runtime, the only question is which interpreter executes them — the deterministic one, the LLM, or the human. This is the talk's one big idea, and it lands harder for being almost throwaway.

> "You don't need to do RAG with vector databases."

Sold on team topology, not accuracy: traditional engineers can build retrieval without entering ML vernacular, and it is "very, very fast to prototype." Success is claimed at the POC stage — exactly where a routing tree shines and exactly where it is least tested. No production war story is offered for the tree at scale.

> "Vibe coding is good for really rapid prototyping. I wouldn't have it in production."

The one concession to the hype, immediately bounded. The surrounding discipline is the talk's most useful engineering content: AI-assisted code rots without enforced refactoring, so commit far more often and refactor every 3–5 steps, because duplication and "poor code written over time by these systems" accumulate faster than human code does.

> "No, you can't… to detect that you would require higher intelligence."

Q&A, asked whether a human should always verify agent outputs between steps. The most honest and least useful moment in the talk. His fallback — "resort to the traditional procedures" used for verifying people — is the same apparatus he elsewhere calls "the biggest issue with most companies." The wall every practitioner hits, stated with no way around it.

> "…we're really benchmarking only for the subset of its abilities that overlaps with ours."

His explanation for why benchmark stars fail in practice. We can only measure generality inside our own evolutionary scope; a model excelling at something we don't value isn't credited with intelligence. Hence "a proper benchmark, to me, isn't one that's a table" — the environment the model is embedded in is the actual result. How to fix this is explicitly deferred to "a separate presentation."

> "If it can preempt you in your action, it's kind of already you."

The closing philosophical claim: coupled systems develop shared representation with their users, so agency blurs across the boundary — and the repercussion of scaling intelligence "will be philosophical or even phenomenal, like psychological," not technological.

## Themes

- #concept — **The company as agentic runtime, humans as function endpoints.** A UI is presented, the user acts, the action returns as a result; the runtime switches between LLM, deterministic, and human contexts.
- #concept — **Four claims about intelligence:** non-modular, information-compressing, necessarily agentic, and identical with generality.
- #pattern — **Routing-agent RAG ("B-tree inspired"):** retrieval through a tree of routing agents, embeddings only at the leaves.
- #pattern — **Two-pipeline prompts-as-code:** one repo, separate CI/CD for the prompt store; template prompts for later ethics review.
- #pattern — **Blind human-vs-agent A/B:** run the same task sequence with a human in one branch and an agent in the other; raters vote without knowing who did what; the winner is reinforced next run.
- #person — **Igor Sevo**, Head of AI at HTEC Group; author of a claimed hundred-page treatise on intelligence behind a link.

## Opinionated take

The strongest material is the practitioner middle: the two-pipeline prompt deployment, the refactor-every-3–5-steps cadence, the human-as-parser audit pipeline, and the refusal to price partial automation as failure ("filter 70%… 70% saved"). These are concrete, falsifiable, and experience-backed — the kind of content that survives contact with a real backlog.

The theoretical layer is asserted, not argued. The four claims arrive "from a lot of research" located in a hundred-page treatise the audience cannot check, and the non-modularity claim is the weakest of them: architecture may be a "linguistic artifact" for managers, but modularity also serves debuggability, auditability, and compliance — a system too coupled to take apart is not merely inconvenient for executives, it is unauditable, which is a liability question, not a linguistic one. The digest catches this and Sevo never engages it.

Two contradictions stand unpatched. First, the verification wall: asked whether humans should always check agent output, he says no — you can't detect failure without higher intelligence — then falls back on the same "traditional procedures" he calls the biggest issue with most companies. Second, the junior pipeline: if juniors never enter the workforce, the seniors who must "not be a poor engineer" to supervise AI have no source; onboarding bots transfer skills to existing staff but don't rebuild the missing rung of the ladder.

The absences are telling for a talk about deploying these systems globally: a runtime that ingests "all the research about the customer," writes agent state to shared storage, and treats humans as function endpoints raises access-control, PII, and prompt-injection questions — none mentioned. "Practical ethics" as regional plumbing is a genuinely novel framing and ethically thin: someone decides where the red lines are, and the talk never says who. And the economics are hand-waved in both directions — the 10x-cost number comes with no cost curve, and the token bill for fanning every query across multiple frontier models goes unmentioned.

And yet the headline thesis earns its place. Most AI-transformation talk treats adoption as a future event to be managed; Sevo's frame — the company already is an agent system in which humans execute the slow path — is a more accurate description of a 2026 enterprise, and it quietly relocates the 10x-cost objection: the question is not whether the runtime exists but which interpreter is expensive.

## Related pages

- [[Multi-Agent AI Systems Are Organizations]] — Sevo's company-as-runtime is the field-reported version of that paper's "organizations by construction" claim. Where the paper demands greater ex ante specification because artificial agents execute literally, Sevo's runtime simply assigns the slow path to humans — which makes the paper's robustness point concrete: the friction buffer exists only while humans are still an interpreter.
- [[The New Software Lifecycle]] — the refactor-every-3–5-steps cadence and commit-more-frequently discipline independently confirms Osmani's verification-heavy lifecycle, from a consulting-delivery direction rather than an engineering-leadership one. Sevo prices it in commit frequency and refactor intervals where Osmani prices it in eval suites.
- [[Goodhart's Law and AI Benchmarks]] — complicates rather than repeats the gaming story: before a benchmark is gamed it can fail at validity. Sevo's overlap argument says even an uncontaminated benchmark misses the model's practical usefulness, because generality beyond our own scope is unmeasurable.
- [[Just Brute Force Your Embeddings]] — the same anti-infrastructure instinct from opposite ends: Turnbull says skip the vector database because brute force is enough; Sevo says skip it because a routing-agent tree lets traditional engineers build retrieval without ML vernacular. Both locate the failure in the default stack, not the technique.

---
*Sources: [[raw/2d88786d2f22b98db9e9b316484fb4ca]], [[summary/2d88786d2f22b98db9e9b316484fb4ca]]*
*Last updated: 2026-09-13*
